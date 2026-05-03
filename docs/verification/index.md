# How Verification Works

Not all registry entries are equal. The RRF's 4-tier verification system lets you understand exactly how much confidence to place in a robot's identity record.

| Tier | Icon | Title |
|---|---|---|
| Tier 1 | ⬜ | Community |
| Tier 2 | 🟡 | Verified |
| Tier 3 | 🔵 | Certified |
| Tier 4 | ✅ | Accredited |

---

## ⬜ Tier 1 — Community

**What it means:** Self-reported. Robot exists and was submitted by a real person. No independent vetting or cross-checking.

**Who qualifies:** Anyone. Submit a robot via the registry form and it immediately receives Community tier.

**Trust level:** Lowest. Do not rely on Community-tier records for safety-critical decisions.

**How to get it:** Automatic on submission. No additional steps required.

**Requirements:**

- Valid submission form
- Contact email
- Plausible manufacturer and model name

**Use cases:**

- Open-source hobby robots
- Research prototypes
- First-generation commercial robots establishing presence in the registry

---

## 🟡 Tier 2 — Verified

**What it means:** Basic verification: the manufacturer exists, the model is real, and the serial number has been cross-checked where possible.

**Who qualifies:** Any robot whose manufacturer can be independently confirmed as a real entity.

**Trust level:** Moderate. Manufacturer and model have been confirmed, but the specific robot unit has not been inspected.

**How to get it:** The RRF reviews the submission against public records and available manufacturer documentation.

**Requirements:**

- Manufacturer is a registered legal entity
- Model appears in manufacturer public records or product pages
- Serial number format is consistent with manufacturer's documented scheme

**Use cases:**

- Commercial robots from known manufacturers
- Robots used in regulated environments requiring documentation
- Insurance and procurement workflows

---

## 🔵 Tier 3 — Certified

**What it means:** The manufacturer organization itself has been verified by the RRF. All robots from this manufacturer automatically carry Certified verification.

**Who qualifies:** Robots from manufacturers that have gone through the RRF manufacturer verification program.

**Trust level:** High. The manufacturer has signed agreements with the RRF and takes responsibility for the accuracy of all registrations under their namespace.

**How to get it:** Manufacturer applies for RRF membership at Supporting Member tier. RRF verifies legal entity, reviews governance documents, and signs a data accuracy agreement.

**Requirements:**

- Manufacturer is an RRF Supporting Member
- Signed data accuracy agreement
- Designated registry contact at the manufacturer
- Manufacturer-issued serial numbers follow documented scheme
- RCAN §21 registry handshake recommended (RRN↔RURI canonical mapping)

**Use cases:**

- Robots deployed in enterprise environments
- Robots subject to EU AI Act registration requirements
- Robots used in safety-critical infrastructure

---

## ✅ Tier 4 — Accredited

**What it means:** Full conformance audit. The robot has passed the RCAN conformance test suite (L1/L2/L3/L4) and carries a signed conformance certificate.

**Who qualifies:** Robots that implement the RCAN protocol and have passed all four conformance levels.

**Trust level:** Highest. The robot's identity, communication protocol, reporting, and registry integration have all been independently audited.

**How to get it:** Robot owner runs the RCAN conformance test suite and submits results. RRF issues a signed certificate with a cryptographic root of trust.

**Requirements:**

- Must hold Certified tier
- Must pass RCAN L1 (Identity), L2 (Communication), L3 (Reporting), and L4 (Registry Integration) conformance tests
- RURI must be present and resolvable per RCAN §21 (Robot Registry Integration)
- RRN↔RURI ownership proof submitted per §21.3
- Certificate must be renewed annually or after major firmware updates

**Use cases:**

- Robots deployed in regulated environments requiring verifiable identity
- Swarm safety applications
- Robots subject to ISO/TC 299 compliance
- Insurance underwriting for autonomous systems

---

## RCAN §21 — Robot Registry Integration

Certified and Accredited robots should implement the RCAN §21 registry handshake for canonical RRN↔RURI mapping and ownership proof. Required for Accredited tier.

[Read §21](https://rcan.dev/spec/section-21/)

---

## Ready to register?

All new robots start at Community tier. Verification upgrades are requested after registration.

[Register Your Robot](https://robotregistryfoundation.org/registry/submit/)
