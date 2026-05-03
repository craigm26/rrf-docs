# Registry Tier API

Federation endpoints for the RCAN protocol registry trust hierarchy (GAP-14). Enables cross-registry command validation via signed trust anchors.

For protocol versions, see [rcan.dev/compatibility](https://rcan.dev/compatibility).

## Registry Tiers

- **root** — Single authoritative root (`registry.opencastor.com`). DNSSEC-anchored, signs authoritative registries.
- **authoritative** — Organisation registries verified by root. Carry root signature in trust anchor.
- **community** — Self-hosted, no root verification. Used for local/private deployments.

!!! note "Status: Specification"
    These endpoints are specified by the [RCAN protocol §21](https://rcan.dev/spec/section-21/). The root registry endpoint
    (`POST /api/v1/registries/{domain}/verify`) is only active on `registry.opencastor.com`.
    Community RRF instances expose GET-only trust anchor lookups.

---

## `GET /api/v1/registries`

List known registries with tier and key fingerprint.

Returns a list of registries known to this RRF instance. Each entry includes the registry domain, trust tier, ML-DSA-65 public key (NIST FIPS 204) fingerprint, and DNSSEC verification status. Root and authoritative registries include signed trust anchors.

**Parameters:**

| Name | In | Type | Required | Description |
|---|---|---|---|---|
| `tier` | query | string | optional | Filter by tier: `root \| authoritative \| community` |
| `verified_only` | query | boolean | optional | If true, return only root and authoritative registries (default: false) |

**Example response:**

```json
{
  "registries": [
    {
      "domain": "registry.opencastor.com",
      "tier": "root",
      "key_fingerprint": "sha256:AbCdEf1234567890abcdef1234567890abcdef1234567890abcdef1234567890",
      "dnssec_verified": true,
      "robots_count": 12,
      "registered_at": "2025-01-01T00:00:00Z",
      "description": "OpenCastor Root Registry — operated by ContinuonAI"
    },
    {
      "domain": "robots.example.org",
      "tier": "authoritative",
      "key_fingerprint": "sha256:1234abcd...",
      "dnssec_verified": true,
      "verified_by": "registry.opencastor.com",
      "verified_at": "2026-03-01T00:00:00Z",
      "robots_count": 3
    },
    {
      "domain": "my-lab.local",
      "tier": "community",
      "key_fingerprint": null,
      "dnssec_verified": false,
      "robots_count": 1
    }
  ],
  "total": 3
}
```

---

## `GET /api/v1/registries/{domain}/trust-anchor`

Get public key PEM and DNSSEC verification status for a registry.

Returns the ML-DSA-65 public key (NIST FIPS 204) in PEM format and DNSSEC verification status for the specified registry domain. For authoritative registries, the root registry signature is included. Robots can use this endpoint to populate their TrustAnchorCache (GAP-14).

**Parameters:**

| Name | In | Type | Required | Description |
|---|---|---|---|---|
| `domain` | path | string | required | Registry domain, e.g. `registry.opencastor.com` or `robots.example.org` |

**Example response:**

```json
{
  "domain": "registry.opencastor.com",
  "tier": "root",
  "public_key_pem": "-----BEGIN PUBLIC KEY-----\nMCowBQYDK2VwAyEA...\n-----END PUBLIC KEY-----",
  "key_algorithm": "ML-DSA-65",
  "key_id": "kid-root-2026-01-001",
  "key_valid_from": "2026-01-01T00:00:00Z",
  "key_expires_at": null,
  "dnssec_verified": true,
  "dnssec_checked_at": "2026-03-17T10:00:00Z",
  "root_signature": null
}
```

**Error responses:**

- `404` — Registry not found or not tracked by this RRF instance

---

## `POST /api/v1/registries/{domain}/verify`

Root registry: verify an authoritative registry.

Root-registry-only endpoint. Verifies a candidate authoritative registry by:

1. Fetching its `/.well-known/rcan-trust-anchor` (ML-DSA-65 public key, NIST FIPS 204)
2. Verifying DNSSEC for the domain
3. Signing the registry's public key with the root private key
4. Recording the verification in the registry database

Only callable by authenticated root registry administrators. Returns the root-signed trust anchor for the verified registry.

**Auth:** Requires root registry admin JWT (`registry_admin: true` claim)

**Parameters:**

| Name | In | Type | Required | Description |
|---|---|---|---|---|
| `domain` | path | string | required | Domain of the registry to verify, e.g. `robots.example.org` |

**Request body:**

```json
{
  "public_key_pem": "-----BEGIN PUBLIC KEY-----\nMCowBQYDK2VwAyEA...\n-----END PUBLIC KEY-----",
  "key_id": "kid-example-2026-03-001",
  "contact_email": "admin@example.org",
  "description": "Example Organisation Robot Registry"
}
```

**Example response:**

```json
{
  "domain": "robots.example.org",
  "tier": "authoritative",
  "key_id": "kid-example-2026-03-001",
  "root_signature": "base64-encoded-root-signature...",
  "verified_by": "registry.opencastor.com",
  "verified_at": "2026-03-17T18:00:00Z",
  "dnssec_verified": true,
  "message": "Registry robots.example.org verified and signed by root"
}
```

**Error responses:**

- `400` — Invalid public key PEM or key_id
- `401` — Not authenticated as root registry admin
- `403` — Caller is not the root registry
- `422` — DNSSEC verification failed for domain
- `409` — Registry already verified — use PATCH to rotate key
