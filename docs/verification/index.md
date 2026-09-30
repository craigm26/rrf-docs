# How Verification Works

A registry record says who registered a robot and what they declared. Verification tiers say what, if anything, has been checked beyond that. They are recorded as `verification_status` on the robot record.

!!! warning "What verification is not"
    No tier certifies a robot, tests it, or says it is safe. Conformance is not certification. Compliance evidence is not regulatory sufficiency. The registry currently has one maintainer and no third-party auditors.

| Tier | `verification_status` | What was checked |
|---|---|---|
| 0 | `unverified` | The registration was signed with the key it declares (ML-DSA-65 + Ed25519). Nothing else. |
| 1 | `community` | A maintainer looked at the record. No independent check. |
| 2 | `manufacturer_claimed` | A DNS TXT record on the manufacturer's domain matches the robot. Proves control of that domain. |
| 3 | `manufacturer_verified` | The DNS TXT check, plus a signed manufacturer attestation and the robot's RURI manifest. |

Tiers move one step at a time and never skip. Promotion to tiers 2 and 3 is a signed request to `POST /v2/robots/{rrn}/verify-tier`; tier 1 is set by a maintainer.

---

## `unverified`

Every new registration starts here. Registration must be signed, so the record is bound to a key, but nobody has checked the name, manufacturer or model.

## `community`

A maintainer has looked at the record. This is a human sanity check, not an audit. Do not rely on it for safety-critical decisions.

## `manufacturer_claimed`

The manufacturer publishes a DNS TXT record binding its domain to the robot. The registry resolves it and records the result. This proves that whoever controls the domain agrees; it says nothing about the robot's behaviour.

## `manufacturer_verified`

In addition to the DNS TXT record, the manufacturer signs an attestation and the robot's RURI manifest matches. This is the strongest identity statement the registry makes today. It is still an identity statement, not a safety or conformance statement.

---

## Earlier tier names

Earlier versions of this page described "Verified", "Certified" and "Accredited" tiers, including RRF-issued conformance certificates renewed annually. Those were never implemented and have been removed. The rcan.dev registry (a separate node) still uses the values `verified`, `certified` and `accredited`; there they mean an owner-requested upgrade with an evidence URL, not a certification.

## Physical assurance

A robot record may also carry a self-declared physical assurance level (A1–A3, RCAN Appendix C). That is a separate field from verification, is always labelled self-declared, and A3 is shown only with a third-party evidence link. See [robotregistryfoundation.org/physical-assurance](https://robotregistryfoundation.org/physical-assurance/).

## RCAN §21 — Robot Registry Integration

Robots that implement RCAN should use the §21 registry handshake for canonical RRN↔RURI mapping and ownership proof. [Read §21](https://docs.rcan.dev/spec/section-21/)

---

## Ready to register?

All new robots start at `unverified`. Verification upgrades are requested after registration.

[Register Your Robot](https://robotregistryfoundation.org/registry/submit/)
