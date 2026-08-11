# 016-terminology-governance.md
## Terminology governance policy
### Code systems, coding requirements, update policy — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all terminology management in the ZarishSphere Platform

---

## Table of contents

1. [Overview](#1-overview)
2. [Approved terminology systems](#2-approved-terminology-systems)
3. [Coding requirements by domain](#3-coding-requirements-by-domain)
4. [Terminology update policy](#4-terminology-update-policy)
5. [Local code extensions](#5-local-code-extensions)
6. [Caching and availability strategy](#6-caching-and-availability-strategy)
7. [Deprecation policy](#7-deprecation-policy)

---

## 1. Overview

Coded data is only interoperable when everyone uses the same codes with the same meaning. This policy defines which terminology systems the ZarishSphere Platform uses for each domain and how they are maintained.

### 1.1 Governing principles

| Principle | Meaning |
|---|---|
| **Standard first** | Always prefer international standard terminology over local codes |
| **Dual coding** | When interoperability demands, store both standard and local codes |
| **Versioned references** | Every code system reference includes the specific version |
| **Machine-readable** | All terminology systems are consumable via FHIR `CodeSystem` resources |
| **Offline capable** | Core terminology is cached locally for Plane 0 and Plane 1 operation |

---

## 2. Approved terminology systems

### 2.1 Primary and secondary systems by domain

| Domain | Primary system | Secondary system | Source repository |
|---|---|---|---|
| Diagnoses | ICD-11 | SNOMED CT | `zs-data-icd11` |
| Laboratory observations | LOINC 2.82 | — | `zs-data-loinc` |
| Clinical findings / symptoms | SNOMED CT | — | `zs-data-snomed` |
| Vaccines | CVX (CDC) | — | `zs-data-cvx` |
| Medications | RxNorm | Local formulary | `zs-data-rxnorm` |
| OpenMRS concepts | CIEL (OpenMRS) | — | `zs-data-ciel` |
| Units of measure | UCUM 2.2 | — | Built-in FHIR |
| Administrative | HL7 FHIR value sets | — | Built-in FHIR |
| Geography | ISO 3166-1 | UN/LOCODE | Built-in |
| Occupations | ISCO-08 | — | `zs-data-isco` |
| Education | ISCED 2011 | — | `zs-data-isced` |
| Finance | ISO 4217 | ISO 20022 | Built-in |

### 2.2 Terminology selection criteria

A terminology system is approved when it meets all these criteria:

| Criterion | Requirement |
|---|---|
| Openly accessible | Freely available for use, at minimum for low-income and humanitarian deployments |
| Machine-readable | Available in a format that can be parsed and loaded into the platform |
| Maintained | Active maintenance by a recognised issuing body with regular updates |
| FHIR compatible | Either available as a FHIR `CodeSystem` or mappable to one |
| Versioned | Clear version identifiers for each release |

---

## 3. Coding requirements by domain

### 3.1 Observations (vitals and lab)

All `Observation` resources MUST have `Observation.code` with a LOINC code:

- LOINC panel codes used for ordered lab panels
- LOINC component codes for individual measurements
- Example: Blood pressure panel LOINC `55284-4`; systolic component `8480-6`

### 3.2 Diagnoses and conditions

All `Condition` resources MUST have a code from one of:

| System | Preference | Use case |
|---|---|---|
| ICD-11 | Preferred | Administrative reporting, aggregate statistics, mortality |
| SNOMED CT | Preferred | Clinical decision support, problem lists, EHR integration |
| Both | Best practice | Include both as `coding` array entries when both are known |

```json
{
  "code": {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "73211009",
        "display": "Diabetes mellitus (disorder)"
      },
      {
        "system": "http://id.who.int/icd/release/11/2026-01/mms",
        "code": "5A10",
        "display": "Diabetes mellitus"
      }
    ],
    "text": "Diabetes mellitus"
  }
}
```

### 3.3 Medications

All `MedicationRequest` resources MUST reference an RxNorm code:

- Country-specific drug codes may be included as secondary coding
- Generic names always included alongside brand names

### 3.4 Vaccines

All `Immunization` resources MUST have a CVX code:

| Requirement | Detail |
|---|---|
| Primary system | CVX (CDC vaccine administered code set) |
| Secondary | Country EPI schedule codes may be included |
| CVX code required | Every immunization record |

### 3.5 Units of measure

All measurable values MUST use UCUM units:

```json
{
  "valueQuantity": {
    "value": 120,
    "unit": "mmHg",
    "system": "http://unitsofmeasure.org",
    "code": "mm[Hg]"
  }
}
```

---

## 4. Terminology update policy

### 4.1 Update schedule

| System | Update frequency | Update process |
|---|---|---|
| ICD-11 | Annual (January) | Automated PR via `zs-agent-dependency-updater` |
| LOINC | Bi-annual (February / August) | Automated PR with release notes |
| SNOMED CT | Monthly (first of each month) | Automated PR via SNOMED International release mirror |
| CVX | As needed (CDC publishes dated releases) | Manual PR when CDC publishes update |
| RxNorm | Continuous | NLM API called at runtime; local cache refreshed weekly |
| CIEL | Irregular | Manual PR when OpenMRS publishes update |
| ISO 3166-1 | As needed | Manual PR on country code changes |

### 4.2 Update process

Each terminology update follows this process:

1. **Release detected** — Automated agent or manual notification identifies a new terminology release
2. **Data download** — Updated terminology files are downloaded to the `zs-data-*` repository
3. **Schema validation** — Validate that the data format matches expected schema
4. **Diff review** — Compare against current version; identify additions, changes, and deprecations
5. **PR created** — Automated PR with release notes and change summary
6. **CI validation** — Validate that all existing forms and workflows still resolve against the updated terminology
7. **Merge and deploy** — After review and approval

### 4.3 Emergency updates

For critical terminology updates (e.g., new ICD-11 emergency codes during a disease outbreak):

1. Bypass the scheduled cycle
2. Fast-track review within 24 hours
3. Deploy as a patch release

---

## 5. Local code extensions

### 5.1 When to create local codes

Countries and deployments may add local code systems for concepts not covered by standard terminologies:

```json
{
  "system": "https://zarishsphere.com/CodeSystem/bgd-camp-codes",
  "code": "camp-1w",
  "display": "Camp 1 West (Kutupalong)"
}
```

### 5.2 Local code system requirements

| Requirement | Detail |
|---|---|
| URI pattern | `https://zarishsphere.com/CodeSystem/{cc}-{name}` |
| FHIR CodeSystem | MUST be defined as a FHIR `CodeSystem` resource |
| Concept list | Complete concept list with displays in at least EN and the local language |
| Versioning | Each CodeSystem MUST have a version identifier |
| Review | Local code systems must be reviewed annually for inclusion into standard terminology |

> **Constraint:** Local codes are a temporary solution. The platform SHOULD migrate to standard international codes when they become available for the concept. All local codes must be reviewed annually.

---

## 6. Caching and availability strategy

### 6.1 Multi-layer caching

| Layer | Technology | Update frequency | Purpose |
|---|---|---|---|
| Hot cache | Valkey (in-memory) | TTL 24 hours | Frequently accessed codes |
| Local mirror | PostgreSQL | Updated on terminology release | Offline support |
| External API | Terminology source API | Runtime (with circuit breaker) | Fallback for cache miss |

### 6.2 Cache fallback order

```
1. Valkey cache (hot) → 5ms
2. PostgreSQL local mirror → 20ms
3. Source API (circuit breaker protected) → 200-2000ms
4. Graceful degradation (cached subset) → immediate
```

### 6.3 Circuit breaker configuration

| Parameter | Value |
|---|---|
| Failure threshold | 5 consecutive failures |
| Cooldown period | 30 seconds |
| Half-open retry interval | 10 seconds |
| Fallback behaviour | Return results from PostgreSQL mirror only |

---

## 7. Deprecation policy

### 7.1 Terminology deprecation process

When a terminology system version is superseded:

1. **Announcement** — Deprecation notice published 6 months before removal
2. **Dual support** — Both old and new versions supported for 3 months
3. **Migration window** — 3 months to migrate to the new version
4. **Removal** — Old version removed from active cache
5. **Archive** — Old version preserved in the data repository for historical queries

### 7.2 Code deprecation within a system

Individual codes within a terminology system MAY become deprecated by the issuing body:

| Deprecation state | Platform behaviour |
|---|---|
| Active | Normal use |
| Deprecated by issuer | Warning on use; still valid for existing records |
| Retired by issuer | Blocked for new records; existing data remains readable |

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/017-terminology-code-systems.md** — Per-system code system details: source, license, access, coding patterns
→ **003-zs-platform/007-data-model.md** — Platform data model and identifier patterns
→ **007-zs-tech-stack/004-data-pipeline.md** — Terminology data pipeline

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
