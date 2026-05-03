# API Reference

Programmatic access to register and query all four RCAN entity types.
The v2 API is the only supported API — v1 was removed when earlier RCAN protocol versions were retired.

**Base URL:** `robotregistryfoundation.org`  
**Authentication:** Bearer token (write ops) — GET endpoints are open  
**Rate limit:** 60 req/min  
**Response format:** `application/json`

!!! warning "API v1 removed (sunset: 2026-03-27)"
    The `/v1/` API is gone. Requests to `/v1/*` return **HTTP 410 Gone** with a migration hint.
    All integrations must use `/v2/` endpoints documented below.
    If you use OpenCastor, ensure your install calls `/v2/` endpoints — check the [OpenCastor docs](https://opencastor.com/docs/).

For the current RCAN protocol version accepted by the registry, see the [live compatibility matrix](https://rcan.dev/compatibility).

---

## Entity Types — [RCAN protocol §21.2.2](https://rcan.dev/spec/section-21/)

| Prefix | Label | Description | Example ID |
|---|---|---|---|
| **RRN** | Robot Registration Number | Whole robots — the primary unit of identity | `RRN-000000000001` |
| **RCN** | Robot Component Number | Hardware components (CPU, NPU, sensors, actuators) | `RCN-000000000001` |
| **RMN** | Robot Model Number | AI models used by robots (VLA, LLM, world model) | `RMN-000000000001` |
| **RHN** | Robot Harness Number | AI harness/orchestration layers (e.g. dual-brain) | `RHN-000000000001` |

All IDs are sequential, zero-padded 12-digit integers assigned by the registry. IDs are globally unique within their prefix and permanent.

---

## Robots (RRN)

### `POST /v2/robots/register`

Register a robot — receive an RRN.

Assigns the next sequential RRN (Robot Registration Number). Required fields: `name`, `manufacturer`, `model`, `firmware_version`, `rcan_version`. Optional: `pq_signing_pub` (ML-DSA-65 public key, base64), `pq_kid`, `ruri`, `owner_uid`.

**Request body fields:**

| Name | Type | Required | Description |
|---|---|---|---|
| `name` | string | required | Human-readable robot name |
| `manufacturer` | string | required | Manufacturer or team name |
| `model` | string | required | Model/product identifier |
| `firmware_version` | string | required | Installed firmware version string |
| `rcan_version` | string | required | RCAN protocol version string from the registering payload — see [rcan.dev/compatibility](https://rcan.dev/compatibility) for the current accepted value |
| `pq_signing_pub` | string | optional | ML-DSA-65 public key (base64-encoded, NIST FIPS 204) |
| `pq_kid` | string | optional | Key ID — 8-char hex (sha256 of pub key prefix) |
| `ruri` | string | optional | RCAN Robot URI: `rcan://{domain}/{org}/{model}/{unit}` |
| `loa_enforcement` | boolean | optional | LoA enforcement enabled (default: true) |
| `owner_uid` | string | optional | Firebase UID of registrant |

**Example response:**

```json
{
  "rrn": "RRN-000000000001",
  "registered_at": "2026-03-27T00:00:00.000Z",
  "record_url": "https://robotregistryfoundation.org/v2/robots/RRN-000000000001"
}
```

---

### `GET /v2/robots/{rrn}`

Get full robot record by RRN.

Returns the complete registry record for a robot. The RRN must follow the 12-digit zero-padded format (`RRN-000000000001`).

| Name | Type | Required | Description |
|---|---|---|---|
| `rrn` | path | required | Robot Registration Number (RRN-XXXXXXXXXXXX) |

**Example response:**

```json
{
  "rrn": "RRN-000000000001",
  "name": "OpenCastor Bob",
  "manufacturer": "ContinuonAI",
  "model": "opencastor-rpi5",
  "firmware_version": "v2026.3.27.1",
  "pq_kid": "7c21bf58",
  "loa_enforcement": true,
  "components": ["RCN-000000000001"],
  "registered_at": "2026-03-27T00:00:00.000Z"
}
```

---

### `GET /v2/robots/{rrn}/sbom`

Get or publish robot SBOM.

GET: retrieve the published SBOM for a robot. POST: publish a new SBOM (requires RRF bearer token). SBOM is countersigned by the RRF root key (ML-DSA-65).

**Example response:**

```json
{
  "rrn": "RRN-000000000001",
  "sbom_format": "spdx-json",
  "rrf_countersig": "Aj7IavBH...",
  "published_at": "2026-03-27T00:00:00.000Z"
}
```

---

## Components (RCN)

### `POST /v2/components/register`

Register a hardware component — receive an RCN.

Assigns the next sequential RCN (Robot Component Number). Links to a parent robot via `parent_rrn`. The parent robot must already be registered.

| Name | Type | Required | Description |
|---|---|---|---|
| `parent_rrn` | string | required | RRN of the owning robot (must exist in registry) |
| `type` | string | required | `cpu \| npu \| gpu \| camera \| lidar \| imu \| actuator \| sensor \| battery \| communication \| other` |
| `model` | string | required | Component model identifier |
| `manufacturer` | string | required | Component manufacturer |
| `firmware_version` | string | optional | Component firmware version |
| `serial_number` | string | optional | Physical serial number (if available) |
| `capabilities` | array | optional | Capability tags (e.g. `["rgb", "depth", "npu:hailo8"]`) |
| `specs` | object | optional | Free-form hardware specs key-value map |

**Example response:**

```json
{
  "rcn": "RCN-000000000001",
  "parent_rrn": "RRN-000000000001",
  "registered_at": "2026-03-27T00:00:00.000Z",
  "record_url": "https://robotregistryfoundation.org/v2/components/RCN-000000000001"
}
```

---

### `GET /v2/components/{rcn}`

Get component record by RCN. Returns the full registry record for a hardware component.

| Name | Type | Required | Description |
|---|---|---|---|
| `rcn` | path | required | Robot Component Number (RCN-XXXXXXXXXXXX) |

**Example response:**

```json
{
  "rcn": "RCN-000000000001",
  "parent_rrn": "RRN-000000000001",
  "type": "npu",
  "model": "Hailo-8",
  "manufacturer": "Hailo",
  "capabilities": ["tops:26", "power:2.5W"],
  "registered_at": "2026-03-27T00:00:00.000Z"
}
```

---

## AI Models (RMN)

### `POST /v2/models/register`

Register an AI model — receive an RMN.

Assigns the next sequential RMN (Robot Model Number). Tracks the AI model provenance chain required by EU AI Act Art. 11.

| Name | Type | Required | Description |
|---|---|---|---|
| `name` | string | required | Model name (e.g. `"LeWorldModel"`) |
| `version` | string | required | Semantic version string |
| `model_family` | string | required | `vision \| language \| multimodal \| vla \| embedding \| reward \| world_model \| diffusion \| other` |
| `architecture` | string | optional | Model architecture (e.g. `"jepa"`, `"transformer"`) |
| `parameter_count_b` | number | optional | Parameter count in billions (e.g. `0.015` = 15M params) |
| `quantization` | string | optional | `fp32 \| fp16 \| bf16 \| int8 \| int4 \| gguf \| bnb4 \| other` |
| `license` | string | optional | SPDX license identifier (e.g. `"apache-2.0"`) |
| `provider` | string | optional | `huggingface \| ollama \| openai \| anthropic \| local` |
| `provider_model_id` | string | optional | Provider-native model ID (e.g. `"meta-llama/Llama-3-8B-Instruct"`) |
| `repo_url` | string | optional | Source repository URL |
| `rcan_compatible` | boolean | optional | Model supports RCAN message signing (default: true) |

**Example response:**

```json
{
  "rmn": "RMN-000000000001",
  "registered_at": "2026-03-27T00:00:00.000Z",
  "record_url": "https://robotregistryfoundation.org/v2/models/RMN-000000000001"
}
```

---

### `GET /v2/models/{rmn}`

Get AI model record by RMN. Returns the full registry record for an AI model.

| Name | Type | Required | Description |
|---|---|---|---|
| `rmn` | path | required | Robot Model Number (RMN-XXXXXXXXXXXX) |

**Example response:**

```json
{
  "rmn": "RMN-000000000001",
  "name": "LeWorldModel",
  "version": "v0.1.0",
  "model_family": "world_model",
  "architecture": "jepa",
  "parameter_count_b": 0.015,
  "rcan_compatible": true,
  "registered_at": "2026-03-27T00:00:00.000Z"
}
```

---

## AI Harnesses (RHN)

### `POST /v2/harnesses/register`

Register an AI harness — receive an RHN.

Assigns the next sequential RHN (Robot Harness Number). A harness is an AI orchestration layer that runs on a robot (e.g. OpenCastor dual-brain). Links to the RMN(s) of models it uses.

| Name | Type | Required | Description |
|---|---|---|---|
| `name` | string | required | Harness name |
| `version` | string | required | Semantic version string |
| `harness_type` | string | required | `vla \| llm_planner \| multimodal \| hybrid \| specialist \| safety_monitor \| orchestrator \| other` |
| `rcan_version` | string | required | Minimum RCAN protocol version required — see [rcan.dev/compatibility](https://rcan.dev/compatibility) for the current accepted value |
| `description` | string | optional | Human-readable description |
| `model_ids` | array | optional | RMN[] — models used by this harness |
| `compatible_robots` | array | optional | RRN[] — target robots (empty = universal) |
| `open_source` | boolean | optional | Whether source code is publicly available |
| `repo_url` | string | optional | Source repository URL |
| `license` | string | optional | SPDX license identifier |

**Example response:**

```json
{
  "rhn": "RHN-000000000001",
  "registered_at": "2026-03-27T00:00:00.000Z",
  "record_url": "https://robotregistryfoundation.org/v2/harnesses/RHN-000000000001"
}
```

---

### `GET /v2/harnesses/{rhn}`

Get AI harness record by RHN. Returns the full registry record for an AI harness.

| Name | Type | Required | Description |
|---|---|---|---|
| `rhn` | path | required | Robot Harness Number (RHN-XXXXXXXXXXXX) |

**Example response:**

```json
{
  "rhn": "RHN-000000000001",
  "name": "OpenCastor Dual-Brain",
  "version": "v2026.3.27.1",
  "harness_type": "hybrid",
  "model_ids": ["RMN-000000000001"],
  "open_source": true,
  "registered_at": "2026-03-27T00:00:00.000Z"
}
```

---

## Unified Registry

### `GET /v2/registry`

List all entities across all types.

Returns a unified listing of all registered entities (robots, components, models, harnesses), sorted by registration date (newest first). Filter by entity type using `?type=`.

| Name | Type | Required | Description |
|---|---|---|---|
| `type` | query | optional | `robot \| component \| model \| harness` — filter to one entity type |
| `limit` | query | optional | Max results (default: 50, max: 200) |

**Example response:**

```json
{
  "entries": [
    {
      "id": "RHN-000000000001",
      "entity_type": "harness",
      "name": "OpenCastor Dual-Brain v2026.3.27.1",
      "registered_at": "2026-03-27T00:00:00.000Z",
      "summary": { "harness_type": "hybrid", "open_source": true }
    }
  ],
  "total": 4,
  "entity_types_count": { "robot": 1, "component": 1, "model": 1, "harness": 1 }
}
```

---

## Compliance Intake (RCAN §22-26)

Robots registered under `/v2/robots/register` can submit EU AI Act compliance artifacts. POST requires a signed body (ML-DSA-65 + Ed25519) against the robot's registered `pq_signing_pub` — the signature IS the auth; no Bearer token needed. GET is public for transparency types (safety-benchmark, ifu, eu-register); Bearer-gated for FRIA and incident-report. Both current and history keys are stored with a 10-year TTL, matching Art. 72 record-keeping obligations.

§26 EU Register (Art. 49) is scoped per AI system — the URL carries the `rmn` (Robot Model Number) rather than an individual robot's `rrn`. The submitting robot identifies itself via the `X-Submitter-RRN` header so RRF can look up its signing key; the stored doc carries `_submitted_by_rrn` for provenance.

| Endpoint | RCAN § | GET access |
|---|---|---|
| `POST /v2/robots/:rrn/fria` | §22 FRIA | Bearer-gated |
| `POST /v2/robots/:rrn/safety-benchmark` | §23 Safety Benchmark | public |
| `POST /v2/robots/:rrn/ifu` | §24 Instructions For Use (Art. 13(3)) | public |
| `POST /v2/robots/:rrn/incident-report` | §25 Post-Market Incident Report (Art. 72) | Bearer-gated |
| `POST /v2/models/:rmn/eu-register` | §26 EU Register (Art. 49) | public |

All five have a matching `GET` at the same path. §26 POST additionally requires an `X-Submitter-RRN` header.

**Example — §23 Safety Benchmark submission:**

```json
{
  "schema": "rcan-safety-benchmark-v1",
  "generated_at": "2026-04-23T00:00:00Z",
  "mode": "synthetic",
  "iterations": 100,
  "thresholds": { "discover_p95_ms": 500 },
  "results": {
    "discover": { "min_ms": 10, "mean_ms": 50, "p95_ms": 120, "p99_ms": 180, "max_ms": 220, "pass": true }
  },
  "overall_pass": true,
  "pq_signing_pub": "<base64>",
  "pq_kid": "abcd1234",
  "sig": {
    "ml_dsa": "<base64>",
    "ed25519": "<base64>",
    "ed25519_pub": "<base64>"
  }
}
```

**Example response:**

```json
{
  "ok": true,
  "rrn": "RRN-000000000001",
  "submitted_at": "2026-04-23T12:34:56.789Z",
  "safety_benchmark_url": "https://api.rrf.rcan.dev/v2/robots/RRN-000000000001/safety-benchmark"
}
```

**Per-type binding checks:**

- `§22 FRIA` — `doc.system.rrn` must equal URL `rrn`
- `§23 SafetyBenchmark` — no doc-level check (URL + signature provide binding)
- `§24 IFU` — no doc-level check (URL + signature provide binding)
- `§25 IncidentReport` — `doc.rrn` must equal URL `rrn`
- `§26 EuRegister` — `doc.rmn` must equal URL `rmn`; signer identified by `X-Submitter-RRN` header

Producer tooling: build and sign compliance docs with the [rcan-ts](https://www.npmjs.com/package/rcan-ts) builders (`buildFria`, `buildSafetyBenchmark`, `buildIfu`, `buildIncidentReport`, `buildEuRegisterEntry`) and `signBody`.

---

## Authentication

All **GET** endpoints are publicly accessible — no credentials required. **POST** (registration) and **PATCH** endpoints require a bearer token issued by RRF.

```
Authorization: Bearer <your-rrf-token>
```

To obtain a registration token, use `castor rrf register` (OpenCastor) or contact the RRF directly. Tokens are tied to an RRN and scoped to that robot's records. Tokens have a maximum TTL of 1 year and must be rotated before expiry.

---

## Quick start — curl examples

**List all registry entries:**

```bash
curl https://robotregistryfoundation.org/v2/registry
```

**Get a robot by RRN:**

```bash
curl https://robotregistryfoundation.org/v2/robots/RRN-000000000001
```

**Register a robot:**

```bash
curl -X POST https://robotregistryfoundation.org/v2/robots/register \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Robot",
    "manufacturer": "Acme Corp",
    "model": "model-001",
    "firmware_version": "v1.0.0",
    "rcan_version": "<see rcan.dev/compatibility>",
    "pq_signing_pub": "<ML-DSA-65 base64 public key>"
  }'
```

**Register a hardware component:**

```bash
curl -X POST https://robotregistryfoundation.org/v2/components/register \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "parent_rrn": "RRN-000000000001",
    "type": "npu",
    "model": "Hailo-8",
    "manufacturer": "Hailo",
    "capabilities": ["tops:26", "power:2.5W"]
  }'
```

---

## Using OpenCastor? The CLI handles registration automatically.

```bash
castor rrf register --config bob.rcan.yaml   # register robot (RRN)
castor components register                    # register components (RCN)
castor sbom generate && castor sbom publish   # publish SBOM + get countersig
```

See the [OpenCastor docs](https://opencastor.com/docs/) for full reference.

> Looking for the live registry search? See [robotregistryfoundation.org/registry](https://robotregistryfoundation.org/registry/).
