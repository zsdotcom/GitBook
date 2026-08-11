# 005-adr-fhir-r5-over-r4.md
## ADR-005: Enforcement of FHIR R5 as the Canonical FHIR Version
### FHIR R5 only — no R4 compatibility layer, no mixed-version support

**Document type:** ADR
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Accepted

---

## Table of Contents

1. [Decision](#1-decision)
2. [Context](#2-context)
3. [Alternatives Considered](#3-alternatives-considered)
4. [Reason for Decision](#4-reason-for-decision)
5. [Consequences](#5-consequences)
6. [Status](#6-status)
7. [Cross-references](#7-cross-references)

---

## 1. Decision

Enforce FHIR Release 5 (R5) as the sole canonical FHIR version across the entire ZarishSphere ecosystem. No R4 or R4B compatibility layer will be maintained. All FHIR resources, operations, search parameters, and interactions follow the R5 specification. The Go-native FHIR server (ADR-004) implements R5 natively with no backward compatibility code paths for earlier versions.

---

## 2. Context

FHIR is the interoperability standard for the ZarishSphere Platform (Constitution Law 3 — every standard is executable). Every data entity stored or exchanged by the platform — patient records, observations, conditions, medications, immunizations, care plans, service requests — uses FHIR resource models.

At the time of this decision (June 2026), the FHIR landscape includes:

- **FHIR R4** (v4.0.1): Current normative release, widely adopted, stable, largest ecosystem of tools and servers
- **FHIR R4B** (v4.3.0): Bridge release between R4 and R5, backports some R5 features to R4
- **FHIR R5** (v5.0.0): Most recent release, introduces significant simplifications and new resources, still being adopted

The ZarishSphere Platform starts from zero — there is no existing data in any FHIR version, no legacy R4 systems to integrate with, and no migration burden. This is a greenfield FHIR decision.

---

## 3. Alternatives Considered

| Alternative | Pros | Cons |
|---|---|---|
| **FHIR R5 exclusively** | Most current standard with simplified resources (Medication, Immunization, BiologicallyDerivedProduct); new resources aligned with ZarishSphere 40 domains (e.g., DeviceAssociation, EvidenceReport, NutritionOrder); improved RESTful search (composite search parameters, `_filter`); no legacy debt; naturally aligned with ADR-004 (building custom server from scratch) | Smaller ecosystem of R5 tools; fewer third-party R5 validators; some implementers (national health systems, EHR vendors) still on R4; must keep up with R5 errata and future releases |
| **FHIR R4 exclusively** | Most widely adopted FHIR release; largest ecosystem of tools (HAPI FHIR, Firely, Smile CDR); extensive documentation and community support; stable normative resources | Lacks R5 simplifications; would need to carry R5-like extensions for ZarishSphere domains; building custom R4 server is more work than building R5 server (R4 has more resource types); will eventually need to migrate to R5 — causing painful transition later |
| **R4 with R5 backport extensions** | Access to R5 features while maintaining R4 compatibility; allows integration with existing R4 systems | Complexity of maintaining R4 base with R5 extension mapping; no existing tooling supports mixed R4+R5; increases Go server implementation complexity significantly; violates KISS principle for a solo developer |
| **Dual version support (R4 + R5)** | Maximum interoperability with external systems | 2x implementation and testing burden; version negotiation logic in every API call; most complex to implement and hardest to maintain; ADR-004 custom Go server would take 2x longer |

---

## 4. Reason for Decision

1. **Greenfield advantage:** ZarishSphere has zero existing FHIR data. There is no R4 database to migrate, no R4 API clients consuming endpoints, and no R4-dependent workflows. Choosing R5 today avoids an inevitable R4-to-R5 migration later — a migration that some national health systems are currently spending millions of dollars and years of effort to execute.

2. **Reduced implementation scope for custom server:** ADR-004 rejects HAPI FHIR in favor of a custom Go FHIR server. FHIR R5 simplified several complex resource structures:
   - MedicationRequest, MedicationAdministration, MedicationDispense → consolidated Medication resource
   - More consistent event resource pattern (e.g., Immunization simplified)
   - New `canonical` and `uri` patterns reduce validator complexity
   - Fewer resource types with overlapping semantics mean less code to write

   Building a custom server for R5 requires significantly less implementation effort than R4.

3. **Alignment with 40-domain taxonomy:** FHIR R5 introduced resources that align directly with ZarishSphere's 40-domain master taxonomy (→ **004-zs-index/002-domain-taxonomy-40.md** — 40-domain master taxonomy):
   - DeviceAssociation → biomedical engineering domain
   - EvidenceReport → research and clinical trials domain
   - NutritionOrder → nutrition domain
   - Transport → logistics domain
   These resources would require extensions in R4, adding complexity.

4. **No interoperability debt to carry:** Organizations still on R4 can use ZarishSphere's FHIR API at the R5 version. If a ZarishSphere deployment needs to exchange data with an R4 system, a gateway adapter can be built as a separate component — but the core server remains R5-only. This preserves architectural integrity (Constitution Law 7 — module sovereignty, each adapter is an independent module).

---

## 5. Consequences

**Positive:**

- Go FHIR server implements a single, internally consistent FHIR version
- No R4-specific code paths reduce codebase size by an estimated 30-40%
- R5 resources map directly to ZarishSphere 40 domains without extension hacks
- Improved R5 search (`_filter`, composite parameters) enables richer queries with less code
- Forward-looking — the industry is moving toward R5 (HL7 normative ballot for R5 ongoing)
- Can participate in R5 interoperability pilots with early-adopter health systems

**Negative:**

- Cannot directly integrate with EHR systems still on FHIR R4 without a gateway adapter
- Smaller ecosystem: fewer R5 validators, fewer reference implementations, less community knowledge
- Some R5 resources are not yet normative (trial-use) — may change in R5.1 or R6
- Training materials and documentation for R5 are less comprehensive than R4
- Contributors familiar with FHIR R4 must learn R5 differences
- Risk if HL7 releases R5.1 with breaking changes before ZarishSphere V1 launches

---

## 6. Status

Accepted. This decision is closely coupled with ADR-004 (no HAPI FHIR) — R5's resource simplification makes the custom Go server viable. If HL7 releases a breaking R5.1 before V1 launch, the ADR will be updated with a migration plan, but the principle of "newest normative release, not R4 compatibility" remains.

---

## 7. Cross-references

→ **008-zs-adrs/004-adr-no-hapi-fhir.md** — ADR-004: custom Go FHIR server (R5 viability)
→ **004-zs-index/002-domain-taxonomy-40.md** — 40-domain master taxonomy aligned with R5 resources
→ **005-zs-standards/005-fhir-r5-conventions.md** — FHIR R5 implementation conventions
→ **005-zs-standards/009-fhir-r4-bridge-policy.md** — R4 bridge/gateway policy for external systems
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 3 (every standard is executable), Law 7 (module sovereignty)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
