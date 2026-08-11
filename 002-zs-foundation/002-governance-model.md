# 002-governance-model.md
## ZarishSphere Foundation governance model
### Decision-making, GitHub as government, and future governance bodies

**Document type:** Normative standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable
**GitHub:** https://github.com/zsdotcom
**Depends on:** `002-zs-foundation/001-foundation-charter.md`

---

## Table of contents

1. [Governance principles](#1-governance-principles)
2. [Current structure: single-founder governance](#2-current-structure-single-founder-governance)
3. [Decision types and processes](#3-decision-types-and-processes)
4. [Future governance bodies](#4-future-governance-bodies)
5. [Activation triggers for governance expansion](#5-activation-triggers-for-governance-expansion)
6. [Dispute resolution](#6-dispute-resolution)
7. [Cross-references](#7-cross-references)

---

## 1. Governance principles

All governance under the ZarishSphere Foundation follows these principles:

| Principle | Meaning |
|---|---|
| GitHub is the government | Every governance action is a public commit, PR, or issue. No off-chain governance. |
| Documentation before decision | Every significant decision is preceded by an ADR or RFC. |
| Transparency by default | All repositories are public. All discussions happen in public. |
| Minimum viable governance | Only as much process as needed. No bureaucracy without demonstrated need. |
| Future-proof structure | Governance is designed for transition from single-founder to community. |

## 2. Current structure: single-founder governance

### 2.1 Current state

The Foundation is currently a single-founder institution. All decisions — technical, editorial, administrative, and strategic — are made by the founder.

| Role | Person |
|---|---|
| Founder and sole decision-maker | Mohammad Ariful Islam |
| GitHub org owner | Mohammad Ariful Islam (arwazarish) |
| Repository admin | Mohammad Ariful Islam |

### 2.2 How decisions are made

1. **Minor decisions** (typos, formatting, non-substantive edits): committed directly to `main`
2. **Moderate decisions** (documentation updates, ZUSS compliance fixes): committed directly or via PR with self-approval
3. **Major decisions** (architecture changes, new specifications, licensing questions): preceded by an ADR or RFC, committed via PR

### 2.3 Limits of single-founder authority

Even a single founder cannot:
- Change the 12 laws of the constitution without following the amendment process
- Violate the zero-cost obligation (Law 5, Law 12)
- Grant proprietary licenses to any ecosystem component
- Transfer Foundation assets to a for-profit entity

## 3. Decision types and processes

| Decision type | Required | Process | Recorded in |
|---|---|---|---|
| Non-substantive edit | None | Direct commit | Git log |
| Documentation update | None | Direct commit or PR | Git log |
| ZUSS compliance fix | None | Direct commit or PR | Git log |
| New specification | ADR | Write ADR, wait 72h, commit | `008-zs-adrs/` |
| Architecture change | ADR | Write ADR, wait 7 days, commit | `008-zs-adrs/` |
| Constitutional amendment | RFC + ADR | Write RFC, 30-day comment period, ADR, commit | `001-zs-meta/001-zarishsphere-constitution.md`, `008-zs-adrs/` |
| Licensing change | RFC + ADR | Write RFC, 30-day comment period, ADR, commit | `001-zs-meta/001-zarishsphere-constitution.md`, `008-zs-adrs/` |
| Governance body creation | Constitutional amendment | Full amendment process | `002-zs-foundation/` |

## 4. Future governance bodies

These bodies may be created as the ecosystem grows. Each requires a constitutional amendment.

### 4.1 Technical Advisory Board

| Attribute | Value |
|---|---|
| Role | Technical architecture guidance and ADR review |
| Composition | 3-7 technical contributors with demonstrated expertise |
| Authority | Advisory — recommends, does not decide |
| Activation trigger | 10+ active external contributors to platform repositories |

### 4.2 Standards Advisory Council

| Attribute | Value |
|---|---|
| Role | Standards adoption oversight and ZI-UID validation |
| Composition | Domain experts from relevant fields |
| Authority | Editorial — validates standard entries against quality criteria |
| Activation trigger | 1,000+ indexed standards in ZARISH-INDEX |

### 4.3 Community Stewardship Council

| Attribute | Value |
|---|---|
| Role | Marketplace quality, component lifecycle, distribution approval |
| Composition | Mix of technical and domain contributors |
| Authority | Operational — approves distributions and marketplace listings |
| Activation trigger | 50+ modules or apps in the Marketplace |

## 5. Activation triggers for governance expansion

| Trigger | Action | Body created |
|---|---|---|
| 10+ external contributors | Proposed TAB formation | Technical Advisory Board |
| 1,000+ indexed standards | Proposed SAC formation | Standards Advisory Council |
| 50+ Marketplace items | Proposed CSC formation | Community Stewardship Council |
| Founder unable to continue | Full governance transfer | New non-profit entity |

Each activation requires a constitutional amendment. The amendment must define the body's composition, selection process, decision-making rules, and relationship to the founder.

## 6. Dispute resolution

While the Foundation is single-founder, the founder is the final arbiter of all disputes. Once governance bodies are created, dispute resolution follows these principles:

1. Technical disputes are escalated to the Technical Advisory Board
2. Editorial disputes are escalated to the Standards Advisory Council
3. Operational disputes are escalated to the Community Stewardship Council
4. Constitutional disputes require a formal RFC and amendment process

No dispute is resolved outside of public GitHub repositories. Private resolution carries no standing.

## 7. Cross-references

→ **001-zs-meta/001-zarishsphere-constitution.md** — Supreme law; the 12 laws and the amendment process bound every decision this model makes
→ **002-zs-foundation/001-foundation-charter.md** — Foundation mission, scope of authority, and binding obligations
→ **002-zs-foundation/003-licensing-policy.md** — Dual-licensing rules that no governance decision may override
→ **002-zs-foundation/004-contributor-guidelines.md** — Contribution rules and review standards enforced through GitHub
→ **008-zs-adrs/003-adr-github-as-government.md** — ADR-003: GitHub as the government (Law 1 implementation)
→ **008-zs-adrs/012-adr-no-single-person-dependency.md** — ADR-012: design rule behind the single-founder to community transition
→ **009-zs-operations/002-sop-github-workflow.md** — Repository branching, protecting, merging, and pull request models that operationalise Law 1

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
