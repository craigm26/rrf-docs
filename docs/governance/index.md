# Robot Registry Foundation

Draft Founding Charter · Seeking co-founders, endorsing organizations, and stakeholder input.

!!! warning "DRAFT"
    Last revised 2026-03-04 · [Discuss on GitHub issue #13](https://github.com/continuonai/rcan-spec/issues/13)

---

## Mission

The **Robot Registry Foundation (RRF)** shall operate the root RCAN robot registry as **neutral, independent, public infrastructure** — analogous to IANA/ICANN for the internet's domain name system, or the Linux Foundation for open-source governance.

- Maintain the authoritative root registry of Robot Registration Numbers (RRNs) and registered robot identities under the RCAN protocol.
- Operate that registry in a vendor-neutral, financially transparent, and publicly accountable manner.
- Serve as the **registry of last resort**, ensuring continuity of robot identity infrastructure regardless of any single organization's commercial fate.
- Steward the [RCAN specification](https://rcan.dev/spec/) in alignment with the broader RCAN community.

---

## The Problem

Robot identity infrastructure today has no independent authority. This creates systemic risks:

**1. Manufacturer-controlled registries**

Each manufacturer operates its own robot registry under its own terms. There is no shared namespace, no interoperability guarantee, and no recourse if a manufacturer changes its policies or exits the market. When a manufacturer's registry disappears, the identity records of deployed robots disappear with it.

**2. No dispute resolution**

If two organizations claim the same Robot Registry Number prefix, or if a manufacturer's identity is impersonated in a third-party registry, there is currently no neutral body to adjudicate the conflict. Legal action in national courts — slow, expensive, and jurisdictionally fragmented — is the only recourse.

**3. No registry of last resort**

There is no entity whose explicit mandate is to preserve the root registry and ensure that deployed robots can always resolve their identity. Infrastructure that depends on any single commercial entity's survival is fragile by design.

**4. Regulatory vacuum**

Emerging frameworks (EU AI Act, ISO/TC 299 standards) increasingly require traceable robot identity for compliance. Without a neutral registration body, regulators cannot point implementors to a trustworthy, independent authority.

---

## Governance Principles

1. **Multi-stakeholder representation** — No single sector — industry, government, academia, or civil society — shall dominate governance. The board and committees shall structurally represent all affected communities.
2. **No single-entity control** — No corporation, government body, or individual shall hold a majority of board seats, veto power over technical decisions, or unilateral control over registry data or software.
3. **Open membership** — Membership shall be open to any organization or individual committed to the RRF's mission. Membership tiers shall reflect contribution levels, not gatekeeping.
4. **Transparent decision-making** — Board deliberations, votes, financial statements, and policy changes shall be published publicly. Major decisions shall include a public comment period of not less than 30 days.
5. **Open source software requirement** — All software used to operate the root registry shall be released under an OSI-approved open source license. This ensures that the community can fork and continue operations if the RRF itself fails.

---

## Proposed Board Composition

The founding board shall consist of **10 voting seats** plus non-voting observers:

| Seats | Constituency | Selection |
|---|---|---|
| 2 | Robot manufacturers | Elected by manufacturer member organizations; rotating 2-year terms; no single manufacturer may hold both seats simultaneously |
| 2 | Safety standards bodies | Designated by ISO/TC 299 (1 seat) and A3 — Association for Advancing Automation (1 seat) |
| 2 | Academic / research institutions | Elected by academic member organizations |
| 1 | Civil society | Elected by contributor-tier members |
| 3 | Foundation general | Elected at large by all member classes combined |
| — | Government observers | Non-voting; open to national standards bodies and regulatory agencies (e.g., NIST, EU Commission, BSI) |

Board members serve 2-year staggered terms. No individual may serve more than three consecutive terms. A simple majority (6/10) is required for operational decisions. A supermajority (8/10) is required for charter amendments, registry policy changes, or dissolution.

---

## Membership Tiers

**Founding Members**

Organizations that actively co-create this charter prior to the Foundation's formal incorporation. Receive a permanent seat in the founding board election. Recognized by name in the Foundation's public record. Commit to a minimum 3-year financial contribution at the Supporting Member level.

**Supporting Members**

Organizations that operate RCAN-compatible registries, deploy RCAN-identified robots, or otherwise depend on the root registry infrastructure. Pay annual dues (tiered by organization size; sliding scale available for non-profits and academic institutions). Elect the manufacturer and academic board seats.

**Contributors**

Individuals who contribute to the RCAN specification, registry software, documentation, or tooling. Recognized in the project's public contributor list. Elect the civil society board seat. No financial obligation.

---

## Registry of Last Resort

The RRF's core operational guarantee is **continuity of the root registry**, independent of any single organization's survival, including the RRF itself.

1. **Data escrow** — A full, cryptographically signed snapshot of the root registry database shall be deposited with at least two independent escrow agents on a weekly basis. The escrow format shall be an open, documented schema with no proprietary dependencies.
2. **Software escrow** — The complete registry software stack shall be escrowed alongside the data. Any organization shall be able to reconstruct and operate a functionally equivalent root registry from the escrowed materials.
3. **Succession plan** — The RRF charter shall name a successor organization (e.g., ISOC, Linux Foundation, or a designated academic consortium) that shall assume operations if the RRF dissolves without an alternative arrangement.
4. **No data hostage** — Registry data shall never be used as collateral, transferred to a for-profit entity without member consent, or made subject to any license that would restrict the community's ability to migrate to a successor registry.

---

## EU AI Act Relevance

### Article 49 — Registration of High-Risk AI Systems

Article 49 requires providers of high-risk AI systems to register those systems in the EU database before placing them on the market. Autonomous robots that fall under Annex III, Category 3 (management and operation of critical infrastructure) or other categories involving physical interaction with persons are likely to qualify as high-risk AI systems.

The current EU database is EU-operated and EU-jurisdictional. This creates compliance complexity for manufacturers outside the EU and for multi-jurisdiction deployments. An independent, internationally governed body such as the RRF could serve as a **candidate supplementary registration body**, accepted by regulators across jurisdictions, reducing the compliance burden for global robot deployments.

RCAN's Robot Registration Numbers (RRNs) and RURIs provide the technical identity layer that registration frameworks require: a unique, persistent, globally resolvable identifier per robot; a structured namespace enabling rapid lookup by regulatory authorities; and an audit chain linking physical robot to registered identity.

---

## Legal Structure Options

The RRF is exploring the following incorporation options:

| Option | Pros | Cons |
|---|---|---|
| US 501(c)(3) | Established framework, donor tax deductions, strong legal precedent for foundations | US-centric, potential regulatory tension for international governance body |
| US 501(c)(6) | Better fit for membership-based industry organizations, more flexibility | Less donor-friendly, still US-centric |
| Swiss Foundation | Internationally neutral jurisdiction, proven model for global tech governance (W3C, CERN) | Higher setup complexity, different member contribution rules |
| EU Association | Directly relevant to EU AI Act compliance efforts | Ties governance to EU jurisdiction, complicates global representation |

---

## How to Get Involved

| Role | What to do |
|---|---|
| Co-founder | Comment on [GitHub issue #13](https://github.com/continuonai/rcan-spec/issues/13) expressing intent to co-found; include your organization name and primary interest |
| Endorsing organization | Post a short statement of endorsement on issue #13; no financial commitment required at this stage |
| Technical contributor | Open a pull request against `docs/governance/` with proposed amendments — [rcan-spec repo](https://github.com/continuonai/rcan-spec) |
| Standards body representative | Contact the RCAN maintainers directly via the repository to discuss formal liaison |
| Interested observer | Star the rcan-spec repository and subscribe to [issue #13](https://github.com/continuonai/rcan-spec/issues/13) for updates |

> This charter was drafted by the RCAN specification maintainers as a starting point for community discussion. It does not represent a final legal document. Nothing herein creates any binding obligation on any party.
