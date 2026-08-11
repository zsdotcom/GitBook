# 007-data-model.md
## Data model
### Core entities, ZS-UID, data sovereignty

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [ZS-UID identifier system](#1-zs-uid-identifier-system)
2. [Core entities](#2-core-entities)
3. [Data sovereignty rules](#3-data-sovereignty-rules)
4. [Cross-module data isolation](#4-cross-module-data-isolation)
5. [Data export](#5-data-export)
6. [Cross-references](#6-cross-references)

---

## 1. ZS-UID identifier system

Every entity in the ZarishSphere ecosystem has a unique ZS-UID. The ZS-UID is not a patient identifier — it is a universal entity identifier across all modules and domains.

### 1.1 ZS-UID format

```
ZS-[MODULE]-[YEAR]-[NNNNNN]
```

| Component | Description | Example |
|---|---|---|
| ZS | ZarishSphere prefix | ZS |
| MODULE | Module code (3-8 uppercase chars) | NCD, HEALTH, LOG |
| YEAR | 4-digit year | 2025, 2026 |
| NNNNNN | Zero-padded sequence number (per module per year) | 000001 |

Examples:
- `ZS-NCD-2025-000001` — first NCD patient registered in 2025
- `ZS-HEALTH-2026-000042` — 42nd health module entity in 2026

### 1.2 Cross-reference table

Cross-references between identifiers (e.g., ZS-UID to UNHCR PROGRES ID) are stored in an encrypted, access-controlled mapping table. This table is the only place where identifier links exist.

```
┌──────────────────────────────────────────┐
│           CROSS-REFERENCE TABLE           │
│  ZS-UID (encrypted key)                   │
│  ├── UNHCR PROGRES ID (encrypted)         │
│  ├── Bangladesh NID (encrypted)           │
│  ├── Camp registration (encrypted)        │
│  ├── OpenMRS ID (encrypted)               │
│  └── ...                                  │
│  Access: role-based, fully audited        │
└──────────────────────────────────────────┘
```

## 2. Core entities

| Entity | ZS-UID prefix | Module | Description |
|---|---|---|---|
| Patient | PAT | All | Person receiving services |
| Provider | PRO | All | Healthcare or service provider |
| Encounter | ENC | Health | Clinical encounter |
| Condition | CON | Health | Diagnosis or condition |
| Observation | OBS | Health | Vital signs, measurements |
| Medication | MED | Health | Prescribed medication |
| Form | FRM | All | Form definition |
| Submission | SUB | All | Submitted form data |
| Workflow | WKF | All | Workflow definition |
| Module | MOD | System | Module instance |
| Deployment | DEP | System | Deployment instance |

## 3. Data sovereignty rules

### 3.1 Data ownership

Data belongs to the deployer, not to the Foundation. The Foundation never claims ownership, access rights, or usage rights to any deployer's data.

### 3.2 Data routing (FDMN protection)

| Population | Records flow to |
|---|---|
| FDMN (Rohingya) | Plane 1 → Plane 2 → UNHCR/NGO cloud only |
| Host community (Bangladeshi) | Plane 1 → Plane 2 → Plane 3 (national DHIS2) |

FDMN individual records never sync to government-accessible systems. Only k-anonymized aggregate counts cross that boundary.

### 3.3 Emergency key destruction

FDMN individual health records in Plane 1 are encrypted with a key stored separately from the data. In a security emergency, the key can be destroyed in under 60 seconds via a single GUI action. This makes the data permanently and irreversibly unreadable.

## 4. Cross-module data isolation

Each module has its own database schema. Cross-module data access must go through the API gateway:

```
Module A (schema_a) → API Gateway → Module B (schema_b)
```

No module may directly import, read, or write another module's database schema.

## 5. Data export

All data is exportable at any time in open formats without Foundation permission:

| Format | Use case |
|---|---|
| FHIR JSON | Health data interoperability |
| CSV | General analysis |
| JSON | Machine processing |
| Parquet | Large dataset analytics |

Exports are generated through the Console (GUI) or API (automated).

## 6. Cross-references

→ **003-zs-platform/002-module-architecture.md** — Module sovereignty and per-module schema isolation
→ **003-zs-platform/005-fhir-architecture.md** — FHIR resource profiles mapped onto the core entities
→ **003-zs-platform/006-api-design.md** — API contracts through which cross-module data access flows
→ **004-zs-index/003-metadata-schema.md** — ZI identifier schema for standards-index entities (related registry IDs)
→ **008-zs-adrs/011-adr-privacy-by-architecture.md** — ADR-011: privacy by architecture, the basis for the cross-reference table
→ **008-zs-adrs/005-adr-fhir-r5-over-r4.md** — ADR-005: FHIR R5 export format baseline

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
