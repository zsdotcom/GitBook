# 002-transformation-model.md
## ZARISH-STANDARDS — Transformation Model
### INDEX-to-STANDARDS Pipeline: Data Mapping Rules · Field Translation · Validation Gates · V1

**Document type:** Specification — Canonical
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative. Governs all ZARISH-INDEX to ZARISH-STANDARDS transformations.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Input Format: ZARISH-INDEX Entry](#2-input-format-zarish-index-entry)
3. [Output Format: ZS-* Standard Definition](#3-output-format-zs--standard-definition)
4. [Field-by-Field Transformation Table](#4-field-by-field-transformation-table)
5. [Transformation Rules and Logic](#5-transformation-rules-and-logic)
6. [Validation Gates](#6-validation-gates)
7. [Error Handling and Rejection Rules](#7-error-handling-and-rejection-rules)
8. [Version Tracking Through the Pipeline](#8-version-tracking-through-the-pipeline)
9. [Pipeline Automation](#9-pipeline-automation)

---

## 1. Overview

The transformation layer is the bridge between ZARISH-INDEX (the world's unified research index of global standards) and ZARISH-STANDARDS (ZarishSphere's curated implementation registry). It converts raw standard metadata — 22-field research entries with ZI-* identifiers — into executable ZS-* standard definitions that the G2A Engine can consume.

**What the transformation layer does:**

- Reads a ZARISH-INDEX entry (22 fields, research-grade metadata)
- Enriches it with ZarishSphere-specific implementation metadata
- Validates the entry against ZARISH-STANDARDS schema rules
- Assigns a ZS-* identifier and type classification (TYPE-A/B/C)
- Generates the deployable standard definition record
- Passes the record to the standards-to-platform pipeline

**What the transformation layer does not do:**

- It does not harvest or collect standards data — that is ZARISH-INDEX's role
- It does not deploy standards to the platform — that is the pipeline layer's role
- It does not modify ZARISH-INDEX source data — transformation is additive, not destructive

The transformation operates as a batch pipeline triggered by ZARISH-INDEX releases, with individual re-transform capability for ad-hoc updates.

---

## 2. Input Format: ZARISH-INDEX Entry

Every transformation begins with a validated ZARISH-INDEX entry. These entries follow the 22-field master schema defined in → **[004-zs-index/003-metadata-schema.md](../004-zs-index/003-metadata-schema.md)**.

### 2.1 ZI-* identifier pattern

Each entry carries a `zarish_id` in the format:

```
[DOMAIN_CODE]-[ISSUER_CODE]-[SHORT_ID]-[YEAR]
```

**Examples:**

| Entry Type | zarish_id |
|---|---|
| WHO ICD-11 | `HL-WHO-ICD11-2026` |
| ISO 15189:2022 | `HL-ISO-15189-2022` |
| Sphere Handbook 2018 | `HM-IASC-SPHERE-2018` |
| FATF 40 Recommendations | `FB-FATF-40REC-2012` |
| HL7 FHIR R5 | `ICT-HL7-FHIRR5-2024` |

### 2.2 Required input fields

The transformation pipeline requires these 22 fields from the ZARISH-INDEX master schema:

| # | Field | Type | Critical for transformation |
|---|---|---|---|
| 1 | `zarish_id` | String | Becomes the ZS-* source reference |
| 2 | `entry_type` | Enum | Determines output classification |
| 3 | `meta_layer` | Enum | Filters domain applicability |
| 4 | `domain` | String | Maps to ZarishSphere 40-domain taxonomy |
| 5 | `sub_domain` | String | Used for ZS-* category grouping |
| 6 | `name_full` | String | Preserved as authoritative name |
| 7 | `name_short` | String | Retained as display name |
| 8 | `standard_id` | String | Used for issuer reference resolution |
| 9 | `issuer` | String | Mapped to standards body registry |
| 10 | `issuer_type` | Enum | Determines governance metadata |
| 11 | `governance_layer` | Enum | Maps to scope label (GLOBAL/REGIONAL/NATIONAL/HUMANITARIAN) |
| 12 | `geographic_scope` | String | Retained for deployment plane filtering |
| 13 | `year_published` | Integer | Base for version tracking |
| 14 | `year_first` | Integer | Used for lifecycle context |
| 15 | `status` | Enum | Maps to ZS-* lifecycle status |
| 16 | `mandate` | Enum | Determines enforcement level in platform |
| 17 | `sector_applicability` | String | Filters domain module applicability |
| 18 | `why_it_matters` | String | Retained for documentation output |
| 19 | `key_outputs` | String | Used for G2A rule generation guidance |
| 20 | `official_url` | URL | Stored as authoritative source reference |
| 21 | `data_source` | String | Audit trail — provenance tracking |
| 22 | `notes` | String | Free-text context for transformation decisions |

---

## 3. Output Format: ZS-* Standard Definition

The transformation produces a ZS-* record — a structured standard definition that the ZarishSphere Platform can consume. Each output record contains ZARISH-STANDARDS-enriched metadata beyond the original ZARISH-INDEX fields.

### 3.1 ZS-* identifier pattern

```
ZS-[DOMAIN_CODE]-[STANDARD_NUMBER]-[YEAR]
```

| Component | Rule | Example |
|---|---|---|
| `ZS` | Fixed prefix — all ZARISH-STANDARDS identifiers | `ZS` |
| `DOMAIN_CODE` | Two-letter domain code from 40-domain taxonomy | `HL` |
| `STANDARD_NUMBER` | Sequential or issuer-based identifier | `ICD11` |
| `YEAR` | Four-digit year of the version | `2025` |

**Examples:**

| Standard | ZS-* ID |
|---|---|
| ICD-11 (2026) | `ZS-HL-ICD11-2026` |
| HL7 FHIR R5 | `ZS-ICT-FHIRR5-2024` |
| WHO PEN Protocol | `ZS-HL-WHOPEN-2023` |
| Sphere Handbook 2018 | `ZS-HM-SPHERE-2018` |
| ISO 14001:2015 | `ZS-EN-ISO14001-2015` |

### 3.2 Output record structure

Each ZS-* output record contains 28 fields:

| # | Field | Source | Description |
|---|---|---|---|
| 1 | `zs_id` | Generated | ZS-* unique identifier |
| 2 | `zarish_id` | Input | Source ZARISH-INDEX entry reference |
| 3 | `name_full` | Input preserved | Authoritative standard name |
| 4 | `name_short` | Input preserved | Short display name |
| 5 | `standard_type` | Transformed | TYPE-A, TYPE-B, or TYPE-C classification |
| 6 | `governance_scope` | Transformed | GLOBAL, REGIONAL, NATIONAL, or HUMANITARIAN |
| 7 | `domain` | Input preserved | Canonical domain from 40-domain taxonomy |
| 8 | `sub_domain` | Input preserved | Sub-domain within the taxonomy |
| 9 | `lifecycle_status` | Transformed | ACTIVE, BETA, DEPRECATED, or RETIRED |
| 10 | `issuer` | Input preserved | Issuing body name |
| 11 | `issuer_type` | Input preserved | Classification of issuing body |
| 12 | `year_published` | Input preserved | Year of current edition |
| 13 | `year_first` | Input preserved | Year first published |
| 14 | `mandate_level` | Transformed | MANDATORY, RECOMMENDED, OPTIONAL, or REFERENCE |
| 15 | `enforcement_rules` | Generated | Platform-level enforcement configuration |
| 16 | `g2a_ruleset_ref` | Generated | Reference to G2A ruleset in the platform |
| 17 | `display_priority` | Generated | Display priority for UI contexts (1-100) |
| 18 | `localisation_rules` | Generated | Field-level display and terminology rules |
| 19 | `replaces` | Generated | ZS-* ID of prior version, if any |
| 20 | `replaced_by` | Generated | ZS-* ID of successor, if any |
| 21 | `dependencies` | Generated | Array of ZS-* IDs this standard depends on |
| 22 | `domain_modules` | Generated | Array of domain modules that consume this standard |
| 23 | `deployment_planes` | Generated | Which planes this standard applies to (0-4) |
| 24 | `official_url` | Input preserved | Primary source URL |
| 25 | `why_it_matters` | Input preserved | Plain-language significance explanation |
| 26 | `data_source` | Input preserved | Provenance record |
| 27 | `transformation_log` | Generated | Audit trail of transformation events |
| 28 | `notes` | Input + Generated | Combined contextual notes |

---

## 4. Field-by-Field Transformation Table

This table defines how each ZARISH-INDEX input field maps to its ZS-* output field, including transformation logic where applicable.

| Input Field | Output Field | Transformation Rule | Notes |
|---|---|---|---|
| `zarish_id` | `zarish_id` | Direct copy | Preserved as foreign key to ZARISH-INDEX |
| (none) | `zs_id` | Generated from ZS-* pattern | See §3.1 for format |
| `entry_type` | `standard_type` | Map via type classification table (see §4.1) | Critical — determines TYPE-A/B/C |
| `meta_layer` | (inferred for domain validation) | Used to verify domain consistency | Not stored in output |
| `domain` | `domain` | Validated against 40-domain taxonomy | Must match canonical name exactly |
| `sub_domain` | `sub_domain` | Direct copy | May be enriched with ZS-standardised sub-domains |
| `name_full` | `name_full` | Direct copy | Authoritative — never modified |
| `name_short` | `name_short` | Direct copy | May be overwritten if ZS standard abbreviation differs |
| `standard_id` | — | Stored in metadata for reference | Not a direct ZS-* field |
| `issuer` | `issuer` | Direct copy | Normalised via standards body registry |
| `issuer_type` | `issuer_type` | Direct copy | Used for governance context |
| `governance_layer` | `governance_scope` | Map: International→GLOBAL, Regional→REGIONAL, National→NATIONAL; plus HUMANITARIAN override if issuer is humanitarian body | Four-value scope label |
| `geographic_scope` | — | Stored for deployment plane filtering | Not a direct ZS-* field |
| `year_published` | `year_published` | Direct copy | Used for version comparison |
| `year_first` | `year_first` | Direct copy | Used for lifecycle context |
| `status` | `lifecycle_status` | Map: Active→ACTIVE, Under Review→ACTIVE, Superseded→DEPRECATED, Withdrawn→RETIRED, Under Development→BETA | Three-value ZS status |
| `mandate` | `mandate_level` | Map: Mandatory→MANDATORY, Treaty-binding→MANDATORY, Voluntary→RECOMMENDED, Voluntary-with-regulatory-adoption→varies by deployment context | Reviewed during transformation |
| `sector_applicability` | `domain_modules` | Parsed and mapped to ZarishSphere domain module codes | Algorithmic mapping with curator override |
| `why_it_matters` | `why_it_matters` | Direct copy | Used in documentation and UI tooltips |
| `key_outputs` | (used for G2A ruleset generation) | Parsed for G2A rule generation | Not a direct ZS-* field |
| `official_url` | `official_url` | Direct copy | Verified at transformation time |
| `data_source` | `data_source` | Direct copy | Audit trail |
| `notes` | `notes` | Direct copy + appended transformation notes | Composite field |

### 4.1 Entry-type to standard-type mapping

| ZARISH-INDEX `entry_type` | ZS `standard_type` | Notes |
|---|---|---|
| `Classification` | TYPE-A | Taxonomy, ontology, code system |
| `Standard` | TYPE-A, TYPE-B, or TYPE-C | Determined by issuer context and function |
| `Framework` | TYPE-C | Governance or operational framework |
| `Treaty` | TYPE-C | Legal governance instrument |
| `Guideline` | TYPE-C | Operational or clinical guideline |
| `Regulation` | TYPE-C | Legally binding governance |
| `Code of Practice` | TYPE-C | Professional practice rules |
| `Recommendation` | TYPE-C | Non-binding guidance |
| `Protocol` | TYPE-C | Supplementary treaty or amendment |
| `Standards Body` | (metadata-only) | Organisation record, not a standard |

> **Constraint:** The `entry_type` to `standard_type` mapping must be reviewed and confirmed by a ZarishSphere standards curator. The automated mapping is a suggestion, not a final decision.

---

## 5. Transformation Rules and Logic

### 5.1 Standard type inference

Where the ZARISH-INDEX entry does not carry explicit TYPE-A/B/C classification, the transformation layer infers the type using the following logic:

1. If the standard defines a **classification** (code system, taxonomy, ontology) → TYPE-A
2. If the standard defines a **data exchange format** (API, protocol, messaging format, schema) → TYPE-B
3. If the standard defines an **operational process** (management system, guideline, framework, governance) → TYPE-C

Where inference is ambiguous, the transformation sets `standard_type` to `UNVERIFIED` and flags the record for curator review.

### 5.2 Governance scope resolution

The ZARISH-INDEX `governance_layer` field maps to ZS `governance_scope` as follows:

- International → `GLOBAL`
- Regional → `REGIONAL`
- National → `NATIONAL`

**Humanitarian scope override:** If the issuing body is a humanitarian organisation (Sphere, IASC, CALP, CPWG, INEE) regardless of the `governance_layer` value, the transformation sets `governance_scope` to `HUMANITARIAN`. This override exists because humanitarian standards are governance-layer-independent in practice.

### 5.3 Lifecycle status mapping

| ZARISH-INDEX Status | ZS Lifecycle Status | Meaning in Platform |
|---|---|---|
| Active | ACTIVE | Bindings enforced; forms generated; workflows active |
| Under Review | ACTIVE | Current edition remains active; new edition pending |
| Under Development | BETA | Released for testing; not for production use |
| Superseded | DEPRECATED | Still available for legacy data; new records discouraged |
| Withdrawn | RETIRED | No longer available; removed from active pipeline |
| UNVERIFIED | PENDING | Not released to platform until verified |

### 5.4 Mandate level assignment

| ZARISH-INDEX Mandate | ZS Mandate Level | Platform Behaviour |
|---|---|---|
| Mandatory | MANDATORY | Field values must conform; validation enforced |
| Treaty-binding | MANDATORY | Legal compliance flag; audit trail required |
| Voluntary | RECOMMENDED | Guidance displayed; non-conformance logged but not blocked |
| Voluntary-with-regulatory-adoption | RECOMMENDED or MANDATORY | Determined by deployment plane and jurisdiction |

### 5.5 Display priority assignment

Display priority (1-100) determines the order in which standards appear in the ZarishSphere UI:

| Criteria | Priority Range |
|---|---|
| Foundational standards (FHIR, ICD-11, WGS-84) | 90-100 |
| Domain-specific primary standards | 70-89 |
| Supporting standards | 50-69 |
| Legacy or transitional standards | 30-49 |
| Humanitarian or national adaptations | 10-29 |

---

## 6. Validation Gates

The transformation pipeline enforces five validation gates before a ZS-* record is released to the platform.

### 6.1 Gate 1 — Schema completeness

Verifies that all 28 output fields are populated. No null values permitted. Empty strings must use the convention `NONE` (for optional fields) or `UNKNOWN` (for fields that require curator input).

### 6.2 Gate 2 — Identifier uniqueness

Confirms that the generated `zs_id` does not collide with any existing ZS-* identifier in the registry. Collisions trigger a rejection with a suggestion to append a disambiguation suffix.

### 6.3 Gate 3 — TYPE-A/B/C validation

Verifies that the assigned `standard_type` is consistent with the entry's purpose. Uses a heuristic check:

- TYPE-A entries must reference a code system or taxonomy repository
- TYPE-B entries must reference an API specification, schema, or protocol document
- TYPE-C entries must reference a governance, operational, or framework document

Standards that fail this check are flagged for curator review but not automatically rejected.

### 6.4 Gate 4 — Cross-reference integrity

Verifies that all references in `dependencies`, `replaces`, `replaced_by`, and `domain_modules` resolve to existing ZS-* records in the registry. Broken references are a rejection.

### 6.5 Gate 5 — Deployment plane compatibility

Verifies that the standard's `deployment_planes` array contains only valid plane values (0, 1, 2, 3, 4) and that at least one plane is assigned. Standards with no planes assigned are rejected.

---

## 7. Error Handling and Rejection Rules

### 7.1 Rejection categories

| Category | Cause | Action |
|---|---|---|
| Schema validation failure | Missing required field, invalid type, null value | Record rejected; entry logged to transformation error log; curator notified |
| Identifier collision | Generated `zs_id` duplicates an existing record | Record held in staging; curator must resolve collision |
| Cross-reference failure | Dependency references a non-existent ZS-* record | Record held in staging; dependency must be created or reference corrected |
| Plane assignment error | No deployment plane specified | Record rejected; deployer must assign at least one plane |
| TYPE-A/B/C ambiguity | Standard type cannot be determined from entry data | Record flagged for curator review; automatically assigned UNVERIFIED |

### 7.2 Handling of deprecated input

When a ZARISH-INDEX entry has `status: Superseded` or `status: Withdrawn`, the transformation layer:
1. Creates or updates the ZS-* record with `lifecycle_status: DEPRECATED` or `RETIRED`
2. Populates the `replaced_by` field if the ZARISH-INDEX entry specifies a successor
3. Generates a notification to the platform deployment system that active bindings may need migration

### 7.3 Manual override protocol

Any transformation stage that fails gates 2, 3, or 4 can be overridden by a ZarishSphere standards curator through a documented override record. Override records must include:
- The ZS-* identifier
- The gate that was overridden
- The reason for override
- The curator's identifier
- A timestamp

Overrides are logged to the transformation audit log and are visible in the G2A Engine's compliance dashboard.

---

## 8. Version Tracking Through the Pipeline

### 8.1 Version identifiers

Every ZS-* record carries a semantic version: `ZS-[ID]:v[MAJOR].[MINOR].[PATCH]`

| Component | Increment trigger |
|---|---|
| MAJOR | Standard version change (new year, new edition) |
| MINOR | Standards registry metadata change (classification update, dependency change) |
| PATCH | Documentation or non-functional metadata update |

### 8.2 Transformation event log

Each transformation event is recorded in the transformation log:

| Field | Description |
|---|---|
| `zs_id` | The ZS-* record affected |
| `event_type` | CREATE, UPDATE, DEPRECATE, RETIRE, OVERRIDE |
| `zarish_version` | Source ZARISH-INDEX version |
| `timestamp` | ISO 8601 UTC timestamp |
| `trigger` | Batch pipeline run, individual re-transform, or manual override |
| `gate_results` | Pass/fail for each of the 5 validation gates |
| `operator` | System (automated) or curator ID (manual) |

### 8.3 Pipeline state machine

```
ZARISH-INDEX Entry
    |
    v
[Staging Queue]
    |
    v
Gate 1: Schema Completeness ---FAIL--> Rejection Log
    |                                       |
    PASS                                    v
    v                                (Curator Review)
Gate 2: Identifier Uniqueness -FAIL---------+
    |                                       |
    PASS                                    v
    v                                (Resolve or Override)
Gate 3: TYPE Validation --------FAIL--------+
    |                                       |
    PASS                                    v
    v                                (Curator Review)
Gate 4: Cross-Reference --------FAIL--------+
    |                                       |
    PASS                                    v
    v                                (Resolve or Override)
Gate 5: Plane Assignment -------FAIL--> Rejection Log
    |
    PASS
    v
[ZS-* Record Created/Updated]
    |
    v
[Transformation Log Updated]
    |
    v
[Standards-to-Platform Pipeline Triggered]
```

### 8.4 Version archive

All historical versions of each ZS-* record are preserved in the standards registry archive. The archive supports:
- Point-in-time reconstruction of the standards registry
- Audit of when and why a standard was modified
- Rollback to a prior version if a transformation error is discovered

---

## 9. Pipeline Automation

### 9.1 Trigger events

The transformation pipeline runs automatically when:

| Trigger | Behaviour |
|---|---|
| ZARISH-INDEX release published | Batch transform all new and modified entries |
| Individual ZARISH-INDEX entry updated | Re-transform single entry |
| ZARISH-STANDARDS curator request | Re-transform with manual overrides |
| Scheduled weekly full sync | Re-process all entries to catch upstream changes |

### 9.2 Integration with ZARISH-INDEX

The pipeline subscribes to the ZARISH-INDEX release manifest via a GitHub release webhook or equivalent event-based notification. On each release, it:
1. Downloads the current `zarish_master.csv`
2. Computes the delta from the prior release
3. Applies transformations to new and modified entries
4. Runs all five validation gates
5. Publishes updated ZS-* records to the standards registry

### 9.3 Integration with downstream consumers

Transformed ZS-* records are published to:
- The ZARISH-STANDARDS registry (Git repository)
- The G2A Engine's standards data store (for real-time compliance checking)
- The deploying module's standards cache (for offline Plane 0 operation)

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Strategic direction and three-type framework
→ **004-zs-index/003-metadata-schema.md** — ZARISH-INDEX 22-field input schema
→ **005-zs-standards/003-standards-schema.md** — ZS-* record validation schemas
→ **005-zs-standards/004-standards-to-platform-pipeline.md** — Downstream pipeline to G2A Engine

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
