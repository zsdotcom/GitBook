# 001-zarish-standards-overview.md
## ZARISH-STANDARDS — Strategic Direction
### ZarishSphere 40-Domain Standards Registry & Implementation Hierarchy · V1

**Document type:** Direction
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative direction for the ZARISH-STANDARDS standards classification project

---

## Table of Contents

1. [Project Identity](#1-project-identity)
2. [How to Read This Document](#2-how-to-read-this-document)
3. [Three-Type Standards Framework](#3-three-type-standards-framework)
4. [Governance Scope Labels](#4-governance-scope-labels)
5. [ZARISH-STANDARDS vs ZARISH-INDEX](#5-zarish-standards-vs-zarish-index)
6. [Complete Standards Hierarchy](#6-complete-standards-hierarchy)
7. [Key Standards by Domain](#7-key-standards-by-domain)
8. [Three-Architecture-Law Mapping](#8-three-architecture-law-mapping)
9. [Standards Display Rules](#9-standards-display-rules)
10. [Maintenance & Governance](#10-maintenance--governance)

---

## 1. Project Identity

| Field | Value |
|---|---|
| Codename | ZARISH-STANDARDS |
| Full name | ZarishSphere Standards Registry & Implementation Hierarchy |
| GitHub repository | `https://github.com/zsdotcom/zs-module-zarish-standards` |
| License | CC BY 4.0 |
| Classification | Standards classification system — ZarishSphere implementation reference |
| Relationship to ZARISH-INDEX | ZARISH-STANDARDS is the curated *implementation subset* of ZARISH-INDEX. It contains only the standards ZarishSphere implements, enforces, or references. |
| Relationship to ZarishSphere | Normative. Every standard implemented in ZarishSphere must appear in this registry. |
| Domains covered | 40 human domains |
| Contact | platform@zarishsphere.com |

### What ZARISH-STANDARDS is

ZARISH-STANDARDS is ZarishSphere's authoritative standards registry — the curated, implementation-focused classification of every international, regional, national, and humanitarian standard that ZarishSphere enforces, implements, or references.

Where ZARISH-INDEX is the world's complete research index of all global standards (88,000+ entries, 40 domains, all governance levels), ZARISH-STANDARDS is ZarishSphere's implementation layer: the standards ZarishSphere actually uses, stored in a hierarchical taxonomy with three mandatory type labels per entry.

ZARISH-STANDARDS answers the question: "Which standard governs this function in ZarishSphere, and what kind of standard is it?"

---

## 2. How to Read This Document

Every node in the hierarchy carries **three mandatory labels** that must always be stated together. Confusing these three types is the most common design error in health information systems.

### The three-label rule

When documenting any standard in ZarishSphere, always write:

```
Standard Name [TYPE-A|B|C] [GOVERNANCE] [STATUS: ACTIVE|BETA|DEPRECATED]
```

Example:
```
ICD-11 (2026-01) [TYPE-A] [GLOBAL] [STATUS: ACTIVE]
HL7 FHIR R5 (v5.0.0) [TYPE-B] [GLOBAL] [STATUS: ACTIVE]
WHO PEN Protocol [TYPE-C] [GLOBAL] [STATUS: ACTIVE]
```

This three-part label is not optional. It is how ZarishSphere engineers know whether they need a classification system, an exchange protocol, or a governance framework.

---

## 3. Three-Type Standards Framework

The central architectural rule of ZARISH-STANDARDS: every standard in the system carries exactly one primary type.

| Label | Category | Defines | Example |
|---|---|---|---|
| **TYPE-A** | Classification Standard | What things are called — taxonomy, ontology, code system, terminology | ICD-11 classifies diseases. SNOMED CT classifies clinical concepts. ISCO-08 classifies occupations. |
| **TYPE-B** | Exchange Standard | How data moves between systems — protocol, API format, messaging standard, data schema | HL7 FHIR R5 defines how to transmit a patient record. ISO 20022 defines how to transmit a payment. IETF RFC 9110 defines HTTP/1.1. |
| **TYPE-C** | Management Standard | How organisations operate — framework, policy, governance rule, operational guideline | ISO 9001 defines how to manage quality. WHO PEN Protocol defines how to run an NCD clinic. Sphere defines how to deliver humanitarian response. |

### Why this typing matters

A universal architecture needs all three — and must know which is which. Consider an NCD consultation in ZarishSphere:

- **ICD-11** [TYPE-A] tells the system what disease this is (classification)
- **HL7 FHIR R5** [TYPE-B] tells the system how to send that record to a partner system (exchange)
- **WHO PEN Protocol** [TYPE-C] tells the system how to manage the clinical process (governance)

All three are present simultaneously in every clinical encounter. Implementing only one type produces an incomplete system. Confusing the types produces a broken architecture.

---

## 4. Governance Scope Labels

All entries also carry a governance scope label:

| Label | Meaning |
|---|---|
| `[GLOBAL]` | Governed by an international SDO (ISO, WHO, UN, HL7, IEC, ITU) |
| `[REGIONAL]` | Governed by a regional body (EU, WHO SEARO, OCHA, CEN/CENELEC) |
| `[NATIONAL]` | Governed by a national body — referenced as adaptation layer for a specific country |
| `[HUMANITARIAN]` | Governed by the humanitarian standards community (Sphere, IASC, CALP, CPWG) |

And a status label:

| Label | Meaning |
|---|---|
| `[STATUS: ACTIVE]` | Currently in force and mandated for use in ZarishSphere |
| `[STATUS: BETA]` | Released for testing, not yet final — use with caution |
| `[STATUS: DEPRECATED]` | Being phased out — retained for legacy compatibility only |

---

## 5. ZARISH-STANDARDS vs ZARISH-INDEX

These are two distinct, complementary projects. Understanding the difference is essential.

| Dimension | ZARISH-INDEX | ZARISH-STANDARDS |
|---|---|---|
| Scope | All global standards — 40 domains, 88,000+ entries | Standards ZarishSphere implements — curated implementation subset |
| Purpose | Research index — find any standard in the world | Implementation registry — govern what ZarishSphere uses |
| Audience | Researchers, NGOs, policymakers, developers globally | ZarishSphere engineers, clinical leads, compliance reviewers |
| Governance | Autonomous — independent project, CC BY 4.0 | Part of ZarishSphere — any change requires an ADR |
| Change authority | Curator (Ariful) — no ZarishSphere approval needed | Architecture changes require ZarishSphere ADR review |
| Update trigger | New global standard published, status change, new relationship | ZarishSphere adds a new domain, implements a new standard, or deprecates an old one |
| License | CC BY 4.0 | CC BY 4.0 (documentation) · Apache 2.0 (implementation code) |

---

## 6. Complete Standards Hierarchy

The full ZARISH-STANDARDS hierarchy covers all 40 domains in the ZarishSphere standards registry. Below is the authoritative tree, with expansion status noted where relevant. `← NEW` marks entries added in the current V1 expansion beyond the original architecture document.

```
./ZARISH_SPHERE/
│
├── /CORE/
│   ├── /UNIVERSAL-IDENTITY/
│   ├── /UNIVERSAL-GEOGRAPHY/
│   ├── /UNIVERSAL-TIME/
│   ├── /UNIVERSAL-ENTITY/
│   ├── /UNIVERSAL-CONSENT/              ← NEW
│   ├── /UNIVERSAL-CLASSIFICATION/       ← NEW
│   ├── /AI-GOVERNANCE/                  ← NEW
│   ├── /CONFORMITY-ASSESSMENT/          ← NEW
│   └── /GOVERNANCE-MODEL/
│
├── /STANDARDS-REGISTRY/
│   ├── /HEALTH/
│   │   ├── /WHO-FIC-FAMILY/
│   │   │   ├── ICD-11 [TYPE-A][GLOBAL][STATUS: ACTIVE]
│   │   │   ├── ICF [TYPE-A][GLOBAL][STATUS: ACTIVE]
│   │   │   ├── ICHI Beta-3 [TYPE-A][GLOBAL][STATUS: BETA]    ← NEW
│   │   │   ├── ICTM [TYPE-A][GLOBAL][STATUS: ACTIVE]         ← NEW
│   │   │   └── WHO-FIC-DERIVED                               ← NEW
│   │   ├── /CLINICAL-TERMINOLOGY/
│   │   │   ├── SNOMED-CT (monthly latest) [TYPE-A][GLOBAL][STATUS: ACTIVE]
│   │   │   ├── LOINC 2.82 [TYPE-A][GLOBAL][STATUS: ACTIVE]
│   │   │   ├── MedDRA [TYPE-A][GLOBAL][STATUS: ACTIVE]       ← NEW
│   │   │   ├── Orphanet [TYPE-A][GLOBAL][STATUS: ACTIVE]     ← NEW
│   │   │   ├── WHODAS-2.0 [TYPE-A][GLOBAL][STATUS: ACTIVE]   ← NEW
│   │   │   └── ATC Classification [TYPE-A][GLOBAL][STATUS: ACTIVE]  ← NEW
│   │   ├── /MEDICINE-DRUG/
│   │   │   ├── RxNorm [TYPE-A][NATIONAL][STATUS: ACTIVE]
│   │   │   ├── WHO-Drug [TYPE-A][GLOBAL][STATUS: ACTIVE]
│   │   │   ├── WHO-EML [TYPE-C][GLOBAL][STATUS: ACTIVE]
│   │   │   └── WHO-Prequalification [TYPE-C][GLOBAL][STATUS: ACTIVE]  ← NEW
│   │   ├── /DATA-EXCHANGE/
│   │   │   ├── HL7-FHIR-R5 (v5.0.0) [TYPE-B][GLOBAL][STATUS: ACTIVE]
│   │   │   ├── HL7-FHIR-R4 (bridge) [TYPE-B][GLOBAL][STATUS: ACTIVE]
│   │   │   ├── HL7-v2.9.1 [TYPE-B][GLOBAL][STATUS: ACTIVE (LEGACY)]     ← NEW
│   │   │   ├── HL7-CDA-R2 [TYPE-B][GLOBAL][STATUS: ACTIVE]            ← NEW
│   │   │   ├── openEHR [TYPE-B][GLOBAL][STATUS: ACTIVE]               ← NEW
│   │   │   ├── DICOM [TYPE-B][GLOBAL][STATUS: ACTIVE]                 ← NEW
│   │   │   ├── SMART-App-Launch-2.2.0 [TYPE-B][GLOBAL][STATUS: ACTIVE]         ← NEW
│   │   │   └── WHO-SMART-Guidelines [TYPE-B][GLOBAL][STATUS: ACTIVE]  ← NEW
│   │   ├── /SURVEILLANCE-AMR/
│   │   │   ├── EWARS [TYPE-C][GLOBAL][STATUS: ACTIVE]        ← NEW
│   │   │   ├── GLASS [TYPE-C][GLOBAL][STATUS: ACTIVE]        ← NEW
│   │   │   └── DHIS2-Data-Model [TYPE-B][GLOBAL][STATUS: ACTIVE]  ← NEW
│   │   └── /CLINICAL-PROTOCOLS/
│   │       ├── WHO-PEN [TYPE-C][GLOBAL][STATUS: ACTIVE]      ← NEW
│   │       ├── WHO-mhGAP [TYPE-C][GLOBAL][STATUS: ACTIVE]    ← NEW
│   │       ├── SPHERE-Health [TYPE-C][HUMANITARIAN][STATUS: ACTIVE]  ← NEW
│   │       └── MPEHS [TYPE-C][NATIONAL][STATUS: ACTIVE]      ← NEW (Bangladesh camp)
│   │
│   ├── /GEOGRAPHY/
│   │   ├── /COORDINATE-SYSTEMS/ [TYPE-A]                     ← NEW
│   │   │   ├── WGS-84 (EPSG:4326) — MANDATORY for all coordinates
│   │   │   ├── ITRF-2020
│   │   │   └── EPSG-Dataset
│   │   ├── /ADMINISTRATIVE-CODES/ [TYPE-A]
│   │   │   ├── ISO-3166-1 (country codes)
│   │   │   ├── ISO-3166-2 (subdivision codes)
│   │   │   ├── ISO-3166-3 (historical codes)
│   │   │   ├── UN-LOCODE
│   │   │   ├── OCHA-P-Codes (camp admin)   ← NEW
│   │   │   ├── OCHA-CODs                   ← NEW
│   │   │   └── GADM                        ← NEW
│   │   ├── /GEOSPATIAL-EXCHANGE/ [TYPE-B]                    ← NEW
│   │   │   ├── OGC-API-Features (ISO 19168)
│   │   │   ├── GeoJSON (de facto web mapping)
│   │   │   ├── KML (OGC)
│   │   │   └── ISO-19100-Series
│   │   └── /LOW-RESOURCE-ADDRESSING/ [TYPE-A]                ← NEW
│   │       ├── Plus-Codes (Open Location Code)
│   │       └── What3Words (3m² grid addressing)
│   │
│   ├── /FINANCE/
│   │   ├── /ACCOUNTING-FRAMEWORKS/ [TYPE-C]
│   │   │   ├── IFRS
│   │   │   ├── IPSAS (public sector)
│   │   │   └── UN-SNA                      ← NEW
│   │   ├── /MESSAGING-EXCHANGE/ [TYPE-B]                     ← NEW
│   │   │   ├── ISO-20022 [STATUS: ACTIVE — mandatory globally Nov 2025]
│   │   │   └── ISO-4217 (currency codes)
│   │   ├── /AID-TRANSPARENCY/ [TYPE-B/C]
│   │   │   ├── IATI-Standard
│   │   │   ├── OECD-DAC-Codes              ← NEW
│   │   │   └── UN-OCHA-FTS                 ← NEW
│   │   ├── /SUSTAINABILITY-ESG/ [TYPE-C]                     ← NEW
│   │   │   ├── GRI-Standards
│   │   │   ├── IFRS-S1-S2 (ISSB)
│   │   │   ├── TCFD
│   │   │   └── OECD-BEPS
│   │   ├── /AUDIT/ [TYPE-C]
│   │   │   ├── INTOSAI-Standards
│   │   │   └── ISA
│   │   └── /HUMANITARIAN-FINANCE/ [TYPE-C]                   ← NEW
│   │       ├── CVA-Standards (CALP Network)
│   │       └── GSMA-Mobile-Money
│   │
│   ├── /LOGISTICS-SUPPLY/
│   │   ├── /IDENTIFICATION/ [TYPE-A]
│   │   │   ├── GS1-GTIN
│   │   │   ├── GS1-GLN
│   │   │   └── GS1-SSCC
│   │   ├── /TRACEABILITY/ [TYPE-B]                           ← NEW
│   │   │   ├── EPCIS
│   │   │   ├── GS1-GTS2
│   │   │   └── GS1-GDSN
│   │   ├── /HUMANITARIAN-LOGISTICS/ [TYPE-C]                 ← NEW
│   │   │   ├── SPHERE-Logistics
│   │   │   └── WHO-Cold-Chain
│   │   └── /MEDICINE-SUPPLY/ [TYPE-C]                        ← NEW
│   │       ├── WHO-Prequalification
│   │       └── WHO-EML-Supply
│   │
│   ├── /HUMAN-RESOURCES/
│   │   ├── /CLASSIFICATION/ [TYPE-A]
│   │   │   ├── ISCO-08
│   │   │   └── ISIC-Rev4                   ← NEW
│   │   ├── /REPORTING/ [TYPE-C]                              ← NEW
│   │   │   ├── ISO-30414-2025 (2nd edition)
│   │   │   └── GRI-400-Series
│   │   ├── /DATA-EXCHANGE/ [TYPE-B]                          ← NEW
│   │   │   └── HR-Open-Standards
│   │   └── /LABOUR-STANDARDS/ [TYPE-C]
│   │       ├── ILO-Core-Conventions (C029, C087, C098, C100, C105, C111, C138, C182)
│   │       ├── ILO-OSH-2001
│   │       └── ISO-45001
│   │
│   ├── /EDUCATION/
│   │   ├── ISCED-2011 [TYPE-A][GLOBAL]
│   │   ├── LTI [TYPE-B][GLOBAL]            ← NEW
│   │   ├── xAPI [TYPE-B][GLOBAL]
│   │   └── INEE-Standards [TYPE-C][HUMANITARIAN]
│   │
│   ├── /ENVIRONMENT-WASH/
│   │   ├── ISO-14001 [TYPE-C][GLOBAL]
│   │   ├── GHG-Protocol [TYPE-B/C][GLOBAL] ← NEW
│   │   ├── GRI-300-Series [TYPE-C][GLOBAL]
│   │   ├── JMP-Definitions [TYPE-C][GLOBAL] ← NEW
│   │   ├── WHO-GDWQ [TYPE-C][GLOBAL]       ← NEW
│   │   └── SPHERE-WASH [TYPE-C][HUMANITARIAN]  ← NEW
│   │
│   ├── /PROTECTION-LEGAL/
│   │   ├── /REFUGEE-DISPLACEMENT/ [TYPE-C]
│   │   │   ├── 1951-Refugee-Convention [GLOBAL]
│   │   │   ├── 1967-Protocol [GLOBAL]
│   │   │   └── Global-Compact-Refugees-2018 [GLOBAL]  ← NEW
│   │   ├── /HUMAN-RIGHTS-TREATIES/ [TYPE-C]           ← EXPANDED
│   │   │   ├── UDHR, ICCPR, ICESCR
│   │   │   ├── CRC [GLOBAL]               ← NEW
│   │   │   ├── CEDAW [GLOBAL]             ← NEW
│   │   │   └── CRPD [GLOBAL]              ← NEW
│   │   ├── /DATA-PRIVACY/ [TYPE-C]                    ← EXPANDED
│   │   │   ├── GDPR [REGIONAL]
│   │   │   ├── PDPA-Bangladesh [NATIONAL] ← NEW
│   │   │   ├── ISO-27701 [GLOBAL]         ← NEW
│   │   │   └── ISO-27001 [GLOBAL]         ← NEW
│   │   └── /CHILD-PROTECTION/ [TYPE-C]               ← NEW
│   │       ├── CRC-Optional-Protocols
│   │       └── CPMS (Child Protection Minimum Standards)
│   │
│   ├── /COMMUNICATION-ICT/                            ← NEW (entire section)
│   │   ├── /INTERNET-PROTOCOLS/ [TYPE-B]
│   │   │   ├── IETF-RFC-Standards (TCP/IP, HTTP, TLS, DNS)
│   │   │   └── W3C-Standards (HTML, CSS, XML)
│   │   ├── /IDENTITY-SECURITY/ [TYPE-B]
│   │   │   ├── OAuth-2.0
│   │   │   ├── OpenID-Connect
│   │   │   └── FIDO2-WebAuthn
│   │   ├── /API-DATA-EXCHANGE/ [TYPE-B]
│   │   │   └── OpenAPI-Specification-3.x
│   │   └── /CYBERSECURITY/ [TYPE-C]
│   │       ├── ISO-27001
│   │       ├── NIST-CSF
│   │       └── ISO-IEC-42001 (AI management)   ← NEW
│   │
│   ├── /ENERGY/                                       ← NEW (entire section)
│   │   ├── IEC-60364 [TYPE-B][GLOBAL]
│   │   ├── IEC-TS-62257 (off-grid solar) [TYPE-C][GLOBAL]
│   │   └── WHO-Solar-Guidelines [TYPE-C][GLOBAL]
│   │
│   ├── /SHELTER-SETTLEMENT/                           ← NEW (entire section)
│   │   ├── SPHERE-Shelter-Standards [TYPE-C][HUMANITARIAN]
│   │   ├── UNHCR-Shelter-Standards [TYPE-C][HUMANITARIAN]
│   │   └── CCCM-Standards [TYPE-C][HUMANITARIAN]
│   │
│   ├── /FOOD-AGRICULTURE/                             ← NEW (entire section)
│   │   ├── Codex-Alimentarius [TYPE-C][GLOBAL]
│   │   ├── IPC [TYPE-A/C][GLOBAL]         ← NEW
│   │   └── SPHERE-Food-Nutrition [TYPE-C][HUMANITARIAN]
│   │
│   ├── /CIVIL-REGISTRATION/                           ← NEW (entire section)
│   │   ├── CRVS-Framework [TYPE-B/C][GLOBAL]
│   │   ├── ICAO-9303 (machine-readable travel docs) [TYPE-B][GLOBAL]
│   │   └── MOSIP [TYPE-B][GLOBAL]
│   │
│   └── /SOCIAL-PROTECTION/                            ← NEW (entire section)
│       ├── CALP-CVA-Standards [TYPE-B][HUMANITARIAN]
│       ├── GSMA-Mobile-Money-API [TYPE-B][GLOBAL]
│       └── GovStack-Registry-BB [TYPE-B][GLOBAL]
```

---

## 7. Key Standards by Domain

### Health (quick reference)

| Standard | Type | Status | Key use in ZarishSphere |
|---|---|---|---|
| ICD-11 (2026-01) | A | Active | Diagnosis coding — WHO free API |
| ICF, ICHI Beta-3, ICTM | A | Active / Beta | WHO-FIC family: function + interventions + traditional medicine |
| SNOMED CT (monthly, latest) | A | Active | Clinical terminology — free for DPG implementers |
| LOINC 2.82 | A | Active | Lab test codes — Regenstrief free CSV |
| MedDRA, ATC Classification | A | Active | Adverse events + drug classification |
| HL7 FHIR R5 (v5.0.0) | B | Active | **Primary clinical data exchange — all records** |
| HL7 FHIR R4 (bridge) | B | Active | Partner system compatibility — 73% of partners still R4 |
| HL7 v2.9.1 (legacy adapter) | B | Active (Legacy) | ~80% of global hospitals — adapter required |
| DICOM | B | Active | Medical imaging |
| WHO SMART Guidelines | B | Active | Machine-readable protocols — primary G2A input |
| WHO PEN, mhGAP | C | Active | NCD + mental health clinical protocols |
| SPHERE Health, MPEHS | C | Active | Humanitarian + Bangladesh camp minimum health package |
| EWARS, GLASS, DHIS2 | C | Active | Disease surveillance + AMR + aggregate reporting |

### Geography (quick reference)

| Standard | Type | Mandate |
|---|---|---|
| WGS-84 (EPSG:4326) | A | **Mandatory — all ZarishSphere coordinates** |
| ISO 3166-1/2/3 | A | Country and subdivision codes |
| OCHA P-Codes, CODs | A/B | Humanitarian admin codes (Cox's Bazar camps) |
| OGC API Features (ISO 19168), GeoJSON | B | Geospatial data exchange |
| Plus Codes, What3Words | A | Low-resource addressing — camp context |

### Finance (quick reference)

| Standard | Type | Status | Notes |
|---|---|---|---|
| ISO 20022 | B | Active | **Mandatory for all inter-institutional payments** (Nov 2025 global switch complete) |
| IATI Standard | C | Active | Donor reporting transparency |
| GRI Standards | C | Active | ESG / sustainability reporting |
| IFRS-S1/S2, TCFD | C | Active | Sustainability disclosure |
| INTOSAI, ISA | C | Active | Supreme audit + external audit standards |
| CALP CVA, GSMA Mobile Money | C/B | Active | Cash assistance + mobile money interoperability |

### Protection & legal (quick reference)

| Standard | Type | Priority |
|---|---|---|
| 1951 Refugee Convention + 1967 Protocol | C | **Core FDMN legal protection — highest priority** |
| UNHCR Data Protection Policy | C | FDMN data governance |
| Bangladesh PDPA 2023 | C | National data protection compliance |
| ICRC Data Protection Handbook | C | Conflict-sensitive data handling |
| CPMS, CEDAW, CRC | C | Child protection + gender + children's rights |

### AI & data governance (quick reference)

| Standard | Type | Status |
|---|---|---|
| ISO/IEC 42001:2023 | C | Active — AI management systems |
| ITU AI Standards 2025 | C | Active — international AI governance |
| OAuth 2.0 (RFC 6749 + RFC 9700), OpenAPI 3.2 | B | Active — authentication + API specification |
| ISO 27001, ISO 27701 | C | Active — information security + privacy management |

---

## 8. Three-Architecture-Law Mapping

Every standard maps to at least one of ZarishSphere's Six Architecture Laws.

| Law | Primary Standards |
|---|---|
| Law 3 — Guideline-as-Code | WHO SMART Guidelines, WHO PEN, WHO mhGAP, SPHERE Handbook, INEE Standards, CALP CVA — all are G2A engine inputs |
| Law 4 — Identity Without Surveillance | 1951 Refugee Convention, UNHCR Data Protection Policy, Bangladesh PDPA 2023, ICRC Data Protection Handbook |
| Law 5 — Zero Lock-In | All data in PostgreSQL (open). All APIs in OpenAPI 3.2. All exchange via FHIR R5 (open). Apache 2.0 everywhere. |
| Law 6 — Freemium DPI | IATI (donor transparency), GRI, ISO 30414 — the standards that make ZarishSphere's public accountability visible |
| All laws | FHIR R5 (exchange), ICD-11 (classification), WGS-84 (geography) — foundational to every operational layer |

---

## 9. Standards Display Rules

These rules govern how standards appear in the ZarishSphere user interface and in all generated outputs.

**Rule 1 — Local first, international underneath.**
Users see local names and local terminology. The system stores and transmits ICD-11, RxNorm, and FHIR IDs automatically in the background. No field health worker is required to know an ICD-11 code. They see "High blood pressure". The system records it as ICD-11 BA00.

**Rule 2 — Display priority follows context.**
In Cox's Bazar: Bangladesh MoH coding appears as the display layer. The MPEHS package takes display priority over generic WHO PEN protocol. International standards are the exchange layer underneath.

**Rule 3 — Deprecated standards are blocked at generation.**
If a standard's `status` field in ZARISH-INDEX is `DEPRECATED`, the G2A Engine refuses to generate new content from it. If a standard is deprecated and a replacement exists, the replacement is automatically referenced.

**Rule 4 — Three-type labels appear in all technical documentation.**
Whenever a standard is referenced in a ZarishSphere technical document, its `[TYPE-A|B|C]` label must be included. No exceptions. This prevents type confusion in implementation.

---

## 10. Maintenance & Governance

### When ZARISH-STANDARDS is updated

Any change to ZARISH-STANDARDS requires an Architecture Decision Record (ADR) because standards directly govern ZarishSphere's clinical and operational behaviour.

| Trigger | Action required |
|---|---|
| New international standard published in a covered domain | Create issue, evaluate impact, create ADR if applicable, update hierarchy |
| Standard status changes to DEPRECATED | Update status, identify replacement, create migration ADR if in active use |
| ZarishSphere adds a new domain or module | Add required standards for that domain to the hierarchy |
| Conflict between two standards covering the same function | Create ADR documenting the resolution decision |

### ADR format for standards changes

```markdown
# ADR-[NNN]-[standard-name]-[action]

## Decision
[Which standard was added / changed / deprecated]

## Context
[Why this standard is relevant to ZarishSphere]

## Alternatives considered
[Other standards evaluated for the same function]

## Reason for decision
[Why this standard was chosen over alternatives]

## Consequences
[Impact on existing code, forms, or workflows]

## Status
Accepted / Superseded / Proposed
```

### Relationship to ZARISH-INDEX updates

When ZARISH-INDEX publishes a new release, the consuming module `zs-module-zarish-standards` reviews the delta for entries relevant to the 40 implementation domains. Relevant new entries or status changes are evaluated and may trigger a ZARISH-STANDARDS update with an accompanying ADR.

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
