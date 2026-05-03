# Revocation & Key API

Robot identity revocation and public key management endpoints. Defined by **GAP-02** (Robot Identity Revocation, §13) and **GAP-09** (Key Lifecycle, §8.6) of the RCAN protocol.

| Spec | MessageType | Cache TTL |
|---|---|---|
| RCAN protocol §13 + §8.6 | 19 — ROBOT_REVOCATION broadcast | 1h (active), 5m (revoked) |

!!! warning "Protocol Safety Invariants"
    - ESTOP from a revoked robot is **always accepted** by peers — revocation never blocks safety halt.
    - RESUME from a revoked robot is **always rejected** — the robot cannot clear its own emergency stop.
    - Revocation check occurs **after** replay check but **before** command execution.
    - Offline robots may continue operation for up to `max_revocation_staleness_s` (default 3600s) before quarantine.

---

## `GET /api/v1/robots/{rrn}/revocation-status`

**Auth:** None (public)

Get revocation status.

Returns the current revocation status of a robot. Robots MUST check this endpoint on startup and cache the result for up to 1 hour (configurable via max-age). If the registry is unreachable, robots may continue operation for up to `max_revocation_staleness_s` (default 3600s) before entering quarantine mode.

**Parameters:**

| Name | Type | Required | Description |
|---|---|---|---|
| `rrn` | path | required | Robot Registration Number (e.g. `RRN-000000000001`) |

**Example:**

```json
{
  "rrn": "RRN-000000000001",
  "status": "active",
  "revoked_at": null,
  "reason": null,
  "authority": "RRF Registry",
  "checked_at": "2026-03-16T20:00:00Z",
  "cache_max_age_s": 3600
}
```

**Status values / Notes:**

```
// Revoked robot:
{
  "rrn": "RRN-000000000099",
  "status": "revoked",
  "revoked_at": "2026-03-15T08:30:00Z",
  "reason": "Stolen — private key compromised",
  "authority": "RRF Registry (owner request)",
  "checked_at": "2026-03-16T20:00:00Z",
  "cache_max_age_s": 300
}

// Possible status values: "active" | "revoked" | "suspended"
// suspended = temporarily restricted; ESTOP still accepted, commands blocked
// revoked   = permanent; all commands blocked; ESTOP still accepted
// active    = normal operation
```

---

## `POST /api/v1/robots/{rrn}/revoke`

**Auth:** Bearer JWT — role: creator or registry admin

Revoke a robot (registry admin only).

Marks a robot as revoked or suspended. Requires registry admin or verified owner JWT (role: `creator`). On success, the registry broadcasts a `ROBOT_REVOCATION` (MessageType 19) message to all connected peers so they can invalidate cached consent and public key material. ESTOP commands from revoked robots are still accepted by peers; RESUME commands are rejected.

**Parameters:**

| Name | Type | Required | Description |
|---|---|---|---|
| `rrn` | path | required | Robot Registration Number to revoke |
| `status` | string | required | `"revoked"` or `"suspended"` |
| `reason` | string | required | Human-readable reason for revocation (max 500 chars) |
| `authority` | string | optional | Issuing authority identifier (defaults to authenticated JWT sub) |

**Example:**

```json
// Request body:
{
  "status": "revoked",
  "reason": "Stolen — private key believed compromised after device loss on 2026-03-15",
  "authority": "craigm26 (owner)"
}

// Response:
{
  "rrn": "RRN-000000000001",
  "status": "revoked",
  "revoked_at": "2026-03-16T20:05:00Z",
  "reason": "Stolen — private key believed compromised after device loss on 2026-03-15",
  "authority": "craigm26 (owner)",
  "broadcast_sent": true,
  "broadcast_message_type": 19
}
```

---

## `GET /api/v1/robots/{rrn}/keys`

**Auth:** None (public)

List public keys for a robot (JWKS).

Returns the JSON Web Key Set (JWKS, RFC 7517) for all signing keys associated with this robot — current and historical. Used for delegation chain verification (GAP-01), offline operation key caching (GAP-06), and audit trail signature verification (GAP-21). Keys include validity windows; expired keys are retained for `replay_window_s × 2` to allow in-flight validation.

**Parameters:**

| Name | Type | Required | Description |
|---|---|---|---|
| `rrn` | path | required | Robot Registration Number |
| `active_only` | boolean | optional | If true, returns only non-expired, non-revoked keys (default: false) |

**Example:**

```json
{
  "rrn": "RRN-000000000001",
  "keys": [
    {
      "key_id": "kid-2026-03-001",
      "kty": "OKP",
      "crv": "ML-DSA-65",
      "use": "sig",
      "alg": "EdDSA",
      "x": "base64url-encoded-public-key",
      "valid_from": "2026-03-01T00:00:00Z",
      "valid_until": "2027-03-01T00:00:00Z",
      "revoked_at": null,
      "is_current": true
    },
    {
      "key_id": "kid-2026-01-001",
      "kty": "OKP",
      "crv": "ML-DSA-65",
      "use": "sig",
      "alg": "EdDSA",
      "x": "base64url-encoded-old-key",
      "valid_from": "2026-01-01T00:00:00Z",
      "valid_until": "2026-03-01T00:00:00Z",
      "revoked_at": null,
      "is_current": false
    }
  ],
  "current_key_id": "kid-2026-03-001",
  "jwks_uri": "https://api.robotregistryfoundation.org/api/v1/robots/RRN-000000000001/keys"
}
```

---

## MessageType 19 — ROBOT_REVOCATION Broadcast

When a robot is revoked via `POST /api/v1/robots/{rrn}/revoke`, the registry broadcasts a `ROBOT_REVOCATION` message (MessageType 19, RCAN protocol §13.4) to all registered peers. Receiving robots **MUST**:

- Invalidate all cached consent grants from the revoked RRN
- Invalidate the revoked robot's cached public key material
- Reject all future commands originating from the revoked RRN (except ESTOP)
- Log a `ROBOT_REVOKED` audit event with the revocation timestamp

**ROBOT_REVOCATION Wire Format:**

```json
{
  "msg_type": 19,
  "msg_id": "uuid-v4",
  "timestamp": "2026-03-16T20:05:00Z",
  "sender_type": "service",
  "service_id": "rrf-registry",
  "payload": {
    "revoked_rrn": "RRN-000000000001",
    "status": "revoked",
    "revoked_at": "2026-03-16T20:05:00Z",
    "reason": "Stolen — private key believed compromised",
    "authority": "craigm26 (owner)"
  }
}
```

---

## Related API Endpoints

| Path | Description |
|---|---|
| `/api/v1/robots/{rrn}` | Full robot record (includes `revocation_status`, `key_id`) |
| `/api/v1/robots/{rrn}/keys` | JWKS for delegation chain verification (this page) |
| `/api/v1/public-keys` | Registry-wide signing keys for offline caching (GAP-06) |

[Back to API Reference](index.md)
