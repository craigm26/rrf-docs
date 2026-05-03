# How the RRF and RCAN Work Together

[RCAN protocol §21](https://rcan.dev/spec/section-21/) · Stable

Two independent standards designed to work together. Use either alone or both together for globally identifiable, cryptographically verifiable robot identity.

---

## Two Different Things, Designed to Compose

**RRF — The Registry**

- Assigns permanent RRNs
- Any robot can register
- No software required
- Neutral, independent governance

**RCAN — The Protocol**

- Defines how robots identify, sign, and prove behavior
- Any language or runtime
- ML-DSA-65 message signing (NIST FIPS 204)
- Hierarchical URIs (RURIs)
- See the [live compatibility matrix](https://rcan.dev/compatibility) · 4 entity types (RRN/RCN/RMN/RHN)
- Registry-relevant features: federated consent protocol (FEDERATION_SYNC, cross-registry JWT trust); human identity LoA 1/2/3 (FIDO2, `min_loa_for_control`); robot identity revocation API (`revocation_status`); bandwidth-constrained transports (LoRa, BLE, CBOR)

**Used together:** a robot becomes globally identifiable via its RRN *and* cryptographically verifiable via its RCAN keys. Anyone can look up the robot, resolve its RURI, and verify that its signed messages are authentic.

---

## The Identifier Relationship

**RRN** — `RRN-000000000001`

Opaque, permanent, registry-assigned. Stable across renames, re-deployments, ownership transfers.

**RURI** — `rcan://rcan.dev/myorg/my-robot-model/unit-001`

Hierarchical, protocol-level address. Human-readable. Scoped to an organization and model.

**Link** — A registered robot can associate its RURI with its RRN. The registry stores the mapping.

**Lookup:**

```
GET /api/v1/resolve?ruri=rcan://...
```

Returns the RRN if the RURI is registered.

---

## Ownership Proof Flow

The challenge/sign/verify handshake from [RCAN §21](https://rcan.dev/spec/section-21/). A robot proves it holds the private key for its RURI without transmitting that key.

**Step 1 — Request a challenge:**

```
POST /api/v1/challenge
{ "ruri": "rcan://rcan.dev/myorg/mybot/unit1" }
```

**Step 2 — Registry returns a 32-byte nonce:**

```json
{ "nonce": "a3f8...", "expires_at": "2026-03-12T00:05:00Z" }
```

**Step 3 — Robot signs nonce with ML-DSA-65 key:**

```bash
# Same ML-DSA-65 key used for RCAN message signing
signature = ml_dsa_sign(private_key, nonce)
```

**Step 4 — Submit proof:**

```
POST /api/v1/verify
{ "ruri": "...", "nonce": "...", "signature": "...", "public_key": "..." }
```

**Step 5 — Registry verifies → robot promoted to "verified" tier:**

```json
{ "status": "verified", "rrn": "RRN-000000000005", "verified_at": "2026-03-12T00:00:42Z" }
```

### Using rcan-py

```python
from rcan import RCANClient, MessageSigner

signer = MessageSigner(private_key_path="~/.rcan/key.pem")
client = RCANClient(ruri="rcan://rcan.dev/myorg/mybot/unit1")

# Register with the Robot Registry Foundation
result = client.registry_register(
    registry_url="https://robotregistryfoundation.org/api/v1",
    metadata={"name": "My Robot", "manufacturer": "MyOrg"}
)
print(result.rrn)  # "RRN-000000000001"
```

rcan-py handles the full challenge/verify handshake automatically.

---

## Choose Your Path

**Without RCAN**

Submit the form, get an RRN. No software to install, no keys to manage. Works for any robot — commercial, research, homebrew.

Verification tier: `community` — suitable for most use cases. The robot is registered and globally identifiable.

[Register now](https://robotregistryfoundation.org/registry/submit/)

**With RCAN**

If your robot runs RCAN, prove ownership cryptographically and automatically upgrade to `verified` or `manufacturer` tier.

The registry stores your public key. Anyone can verify your robot's signed messages without contacting you directly.

[Get RCAN](https://rcan.dev)

---

## §19 INVOKE — Remote Skill Execution

RCAN §19 defines how a principal triggers a named skill on a robot and receives a structured result. Robots registered with the RRF can expose INVOKE endpoints, allowing authorized callers to execute skills by RURI + RRN.

**INVOKE request:**

```json
{
  "type": "INVOKE",
  "skill": "pick_and_place",
  "params": { "target": "red_cube" },
  "timeout_ms": 5000,
  "msg_id": "invoke_abc123"
}
```

**INVOKE_RESULT (success):**

```json
{
  "type": "INVOKE_RESULT",
  "skill": "pick_and_place",
  "status": "success",
  "reply_to": "invoke_abc123",
  "duration_ms": 1230
}
```

**INVOKE_RESULT (failure):**

```json
{
  "type": "INVOKE_RESULT",
  "skill": "pick_and_place",
  "status": "failure",
  "reply_to": "invoke_abc123",
  "duration_ms": 312,
  "error": { "code": 7006, "name": "SkillFailed", "message": "Gripper could not secure the target object" }
}
```

**RBAC enforcement:** INVOKE messages are subject to RCAN role-based access control. A principal must hold the required capability lease for the target skill. RRF-registered robots can publish their skill manifests publicly, enabling third-party orchestration.

[Full §19 INVOKE specification](https://rcan.dev/spec/section-19/)

---

## §20 Telemetry Field Registry

RCAN §20 defines standard telemetry objects — joint state, robot state, and Prometheus exposition — that robots emit on a predictable cadence. RRF-registered robots may publish a telemetry endpoint URL in their registry record so observers can subscribe without prior coordination.

**Joint telemetry object:**

```json
{
  "joint": "shoulder_pan",
  "position_rad": 0.785,
  "velocity_rad_s": 0.02,
  "effort_nm": 0.41,
  "temperature_c": 38.5,
  "error_flags": 0,
  "timestamp_us": 1741737600000000
}
```

**Prometheus exposition (excerpt):**

```
rcan_acb_position_rad{joint="shoulder_pan",ruri="rcan://robot.local"} 0.785
rcan_acb_velocity_rad_s{joint="shoulder_pan",ruri="rcan://robot.local"} 0.02
rcan_robot_battery_v{ruri="rcan://robot.local"} 14.2
rcan_robot_uptime_ms{ruri="rcan://robot.local"} 3600000
```

**RRF integration:** A robot's registry record can include a `telemetry_url` field. This lets dashboards and monitoring systems discover the Prometheus scrape endpoint directly from the RRN lookup — no separate service discovery required.

[Full §20 Telemetry Field Registry](https://rcan.dev/spec/section-20/)

---

## OpenCastor — RCAN-Conformant Implementation

OpenCastor is an open-source Python framework for building RCAN-conformant robots. It implements the full RCAN stack including RBAC, ML-DSA-65 signing (NIST FIPS 204), safety invariants, and §21 registry integration.

Full [RCAN protocol](https://rcan.dev/compatibility) conformance · ML-DSA-65 PQ signing · RBAC + capability leases · Virtual FS with safety kernel · 4-type registry (RRN/RCN/RMN/RHN) · RRF [§21](https://rcan.dev/spec/section-21/) registry integration · Runs on Raspberry Pi, x86, and more.

- [opencastor.com](https://opencastor.com)
- [GitHub](https://github.com/craigm26/OpenCastor)

---

## Go deeper

- [RCAN §21 — Registry Integration](https://rcan.dev/spec/section-21/)
- [OpenCastor](https://opencastor.com)
- [OpenCastor SDK — Getting Started](https://opencastor.com/docs/getting-started/)
- [RCAN protocol spec](https://rcan.dev)
- [ROBOT.md — one-file robot manifest](https://robotmd.dev)

> Looking for the live registry search? See [robotregistryfoundation.org/registry](https://robotregistryfoundation.org/registry/).
