# 001-foundation-charter.md
## ZarishSphere Foundation charter
### Mission, scope, obligations, and project relationships

**Document type:** Charter
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable
**GitHub:** https://github.com/zsdotcom
**Depends on:** `001-zs-meta/001-zarishsphere-constitution.md`

---

## Table of contents

1. [Purpose of this charter](#1-purpose-of-this-charter)
2. [Foundation mission](#2-foundation-mission)
3. [Scope of authority](#3-scope-of-authority)
4. [Obligations](#4-obligations)
5. [Relationship to the four projects](#5-relationship-to-the-four-projects)
6. [Relationship to other ecosystem components](#6-relationship-to-other-ecosystem-components)
7. [Perpetuity and dissolution](#7-perpetuity-and-dissolution)
8. [Charter amendments](#8-charter-amendments)
9. [Cross-references](#9-cross-references)

---

## 1. Purpose of this charter

This charter establishes the ZarishSphere Foundation as the governing institution for the ZarishSphere ecosystem. It defines:

- What the Foundation is and what it exists to do
- The scope and limits of its authority over each project
- Its binding obligations to deployers, contributors, and the public
- The constitutional relationship between the Foundation and the four core projects

This charter is subordinate to the constitution (→ **001-zs-meta/001-zarishsphere-constitution.md**). Where this charter conflicts with the constitution, the constitution governs.

## 2. Foundation mission

The ZarishSphere Foundation exists to build, maintain, and govern a single integrated ecosystem that:

1. Indexes every global standard across every domain of human civilization
2. Transforms indexed standards into executable digital assets
3. Makes those assets deployable and usable at zero cost by anyone with a browser
4. Provides the complete toolchain — console, marketplace, builder, apps, forms, SDK, CLI, API, services, modules, distributions, engine, system — to enable universal access

The Foundation is not a software vendor, consulting firm, hosted-service provider, or data broker. It is a public infrastructure institution.

## 3. Scope of authority

### 3.1 What the Foundation governs

| Domain | Authority level |
|---|---|
| Constitution and ZUSS standard | Supreme — binding on all projects |
| ZARISH-INDEX schema and quality | Editorial — sets rules, does not curate |
| ZARISH-STANDARDS transformation rules | Editorial — defines the pipeline, not the outputs |
| ZarishSphere Platform architecture | Technical — sets constraints, not implementation |
| Ecosystem component specifications | Technical — defines interfaces, not internals |
| Licensing and IP | Absolute — all projects comply |
| Repository organisation | Administrative — owns the GitHub org |

### 3.2 What the Foundation does not govern

- The content of any indexed standard (that belongs to the standards body)
- How any deployer configures their deployment
- What data any deployer collects through the platform
- How any contributor structures their contributions (within ZUSS bounds)

## 4. Obligations

The Foundation has the following binding obligations:

| Obligation | Source | Description |
|---|---|---|
| Permanent zero cost | Law 5 | No ecosystem component may ever carry a price tag |
| No-code accessibility | Law 6 | Every capability must be operable via GUI |
| Plane 0 compliance | Law 4 | Every component must function on a 4 GB air-gapped device |
| Data portability | Law 8 | Every deployer can export all data at any time |
| Documentation first | Law 2 | Every component is documented before its first commit |
| GitHub transparency | Law 1 | Every governance decision is a public commit |
| No vendor lock-in | Law 9 | No dependency that cannot be replaced in 90 days |
| Emergency key destruction | Law 4 | Ability to render records unrecoverable in 60 seconds |

## 5. Relationship to the four projects

The Foundation maintains four distinct but interdependent projects:

### 5.1 ZarishSphere Platform

The deployable infrastructure. The Foundation sets the architectural constraints (Go-native, Plane 0-compatible, no-code interface), publishes the specifications, and maintains the canonical documentation. Implementation is the Foundation's responsibility until the community grows large enough to share it.

### 5.2 ZARISH-INDEX

An autonomous open research data product. The Foundation sets the metadata schema (`ZI-[DOMAIN]-[NNNNN]`), the domain taxonomy, and the quality standards. The actual indexing work is an ongoing research activity led by the founder.

### 5.3 ZARISH-STANDARDS

A standards classification and transformation system. The Foundation defines the transformation model, the G2A pipeline interfaces, and the output schema. The Foundation does not create or modify the content of any standard — it only transforms what standards bodies publish.

### 5.4 Ecosystem components

The complete suite of user-facing and developer-facing tools (Console, Marketplace, Builder, Apps, Forms, SDK, CLI, API, Services, Modules, Distributions, Engine, System). The Foundation specifies, builds, and maintains all components. Each component has its own specification document in `010-zs-ecosystem/`.

## 6. Relationship to other ecosystem components

The Foundation maintains the specifications and canonical implementations for all 13 ecosystem components, published as 19 product specifications in `010-zs-ecosystem/`. Each component specification is a binding contract between the Foundation and any implementer — whether the Foundation itself or a future community contributor.

Component specifications are written in ZUSS format with standard front matter, numbered sections, and cross-references to related components.

## 7. Perpetuity and dissolution

### 7.1 Perpetuity intent

The ZarishSphere Foundation is designed to outlast its founder. All assets — repositories, documentation, domains, configurations — are structured so they can be transferred to a community governance body if the founder can no longer maintain them.

### 7.2 Dissipation

The only acceptable form of dissolution is the transfer of all Foundation assets to a non-profit entity with a mission identical to the Foundation's. No Foundation asset may ever be transferred to a for-profit entity, sold, or monetized in any form. This provision is irrevocable.

## 8. Charter amendments

Amendments to this charter follow the constitutional amendment process defined in → **001-zs-meta/001-zarishsphere-constitution.md** §10. All amendments are recorded as ADRs in `008-zs-adrs/` and committed to the repository.

## 9. Cross-references

→ **001-zs-meta/001-zarishsphere-constitution.md** — Supreme law of the ecosystem; defines the 12 laws and the amendment process this charter is subordinate to
→ **001-zs-meta/002-zarishsphere-profile.md** — Foundation identity, mandate, and the full ecosystem vision
→ **002-zs-foundation/002-governance-model.md** — Decision-making model: single-founder governance, ADR and RFC processes, future governance bodies
→ **002-zs-foundation/003-licensing-policy.md** — Apache 2.0 (code) and CC BY 4.0 (documentation) dual-licensing rules
→ **002-zs-foundation/004-contributor-guidelines.md** — How anyone can contribute to any ZarishSphere project
→ **002-zs-foundation/005-country-adoption-model.md** — Country Adoption Maturity Model pathway from first contact to digital health sovereignty
→ **003-zs-platform/001-platform-overview.md** — The deployable platform project: architecture, deployment planes, and technology stack
→ **008-zs-adrs/009-adr-no-vendor-lock-in.md** — ADR-009: no-vendor-lock-in obligation (Law 9) implementation
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: zero-cost toolchain obligation (Law 5) implementation
→ **010-zs-ecosystem/001-console-spec.md** — First of the 13 ecosystem component specifications (Console)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
