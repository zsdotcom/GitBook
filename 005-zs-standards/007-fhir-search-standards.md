# 007-fhir-search-standards.md
## FHIR search standards
### Required search parameters, pagination, chaining, performance — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all FHIR search implementation in the ZarishSphere Platform

---

## Table of contents

1. [Universal search parameters](#1-universal-search-parameters)
2. [Required search parameters by resource type](#2-required-search-parameters-by-resource-type)
3. [Search modifiers and chaining](#3-search-modifiers-and-chaining)
4. [Multi-tenant search enforcement](#4-multi-tenant-search-enforcement)
5. [Pagination rules](#5-pagination-rules)
6. [Performance requirements](#6-performance-requirements)

---

## 1. Universal search parameters

### 1.1 Parameters for all resources

Every FHIR resource type in the ZarishSphere Platform MUST support these universal search parameters:

| Parameter | Type | Description |
|---|---|---|
| `_id` | token | Search by FHIR resource ID (UUID v7) |
| `_lastUpdated` | date | Filter by last modification timestamp |
| `_tag` | token | Filter by resource tag (tenant, program) |
| `_profile` | uri | Filter by declared profile URL |
| `_count` | number | Limit results per page (max: 200, default: 20) |
| `_offset` | number | Pagination offset from start |
| `_sort` | string | Sort results (prefix `-` for descending) |
| `_summary` | token | Return summary form: `true`, `false`, `count` |
| `_include` | string | Include referenced resources in search results |
| `_revinclude` | string | Include resources that reference the matched resource |

### 1.2 _sort behaviour

| Sort parameter | Order |
|---|---|
| `_sort=_lastUpdated` | Ascending (oldest first) |
| `_sort=-_lastUpdated` | Descending (newest first) — default for most searches |
| `_sort=family` | Alphabetical by family name (Patient only) |

---

## 2. Required search parameters by resource type

### 2.1 Patient

| Parameter | Type | Example |
|---|---|---|
| `family` | string | `?family=Ahmed` |
| `given` | string | `?given=Mohammed` |
| `birthdate` | date | `?birthdate=1990-01-15` |
| `gender` | token | `?gender=male` |
| `identifier` | token | `?identifier=NID\|ABC123` |
| `name` | string | `?name=Ahmed` (full-text search across all name components) |
| `address` | string | `?address=Dhaka` |
| `address-country` | token | `?address-country=BD` |
| `telecom` | token | `?telecom=phone\|+8801712345678` |

### 2.2 Observation

| Parameter | Type | Example |
|---|---|---|
| `subject` | reference | `?subject=Patient/0192fbad-...` |
| `code` | token | `?code=http://loinc.org\|8310-5` |
| `date` | date | `?date=ge2026-01-01` |
| `category` | token | `?category=vital-signs` |
| `status` | token | `?status=final` |
| `value-quantity` | quantity | `?value-quantity=gt37\|Cel` |

### 2.3 Encounter

| Parameter | Type | Example |
|---|---|---|
| `subject` | reference | `?subject=Patient/0192fbad-...` |
| `status` | token | `?status=in-progress` |
| `date` | date | `?date=ge2026-01-01` |
| `class` | token | `?class=AMB` |
| `location` | reference | `?location=Location/org-clinic-01` |

### 2.4 Condition

| Parameter | Type | Example |
|---|---|---|
| `subject` | reference | `?subject=Patient/0192fbad-...` |
| `code` | token | `?code=http://snomed.info/sct\|73211009` |
| `clinical-status` | token | `?clinical-status=active` |
| `recorded-date` | date | `?recorded-date=ge2026-01-01` |

### 2.5 MedicationRequest

| Parameter | Type | Example |
|---|---|---|
| `subject` | reference | `?subject=Patient/0192fbad-...` |
| `status` | token | `?status=active` |
| `medication` | token | `?medication=rxcui\|1049502` |
| `authored` | date | `?authored=ge2026-01-01` |

### 2.6 Practitioner

| Parameter | Type | Example |
|---|---|---|
| `name` | string | `?name=Fatima` |
| `identifier` | token | `?identifier=NID\|ABC123` |
| `specialty` | token | `?specialty=http://snomed.info/sct\|394802001` |

### 2.7 Organization

| Parameter | Type | Example |
|---|---|---|
| `name` | string | `?name=Clinic` |
| `type` | token | `?type=http://hl7.org/fhir/organization-type\|prov` |
| `address` | string | `?address=Dhaka` |

---

## 3. Search modifiers and chaining

### 3.1 Supported search modifiers

| Modifier | Applies to | Example |
|---|---|---|
| `:exact` | string | `?family:exact=Ahmed` |
| `:contains` | string | `?name:contains=med` |
| `:missing` | token | `?birthdate:missing=true` |

### 3.2 Chained search parameters

ZarishSphere supports chained search parameters for reference fields:

```
# Observations for patients with a given family name
GET /fhir/R5/Observation?subject.family=Ahmed

# Encounters at locations in a specific city
GET /fhir/R5/Encounter?location.address-city=Teknaf

# Conditions diagnosed by a specific practitioner
GET /fhir/R5/Condition?asserter.name=Fatima
```

### 3.3 Reverse chaining

```
# Patients who have an active condition of diabetes
GET /fhir/R5/Patient?_has:Condition:subject:code=SNOMED\|44054006

# Patients with an observation exceeding a threshold
GET /fhir/R5/Patient?_has:Observation:subject:value-quantity=gt200\|mg/dL
```

> **Constraint:** Chained searches are supported to a maximum depth of 3 resource hops. Deeper chains must use the `_has` reverse chaining pattern instead of nested chaining.

---

## 4. Multi-tenant search enforcement

### 4.1 Tenant context

Every FHIR search request is scoped to a tenant. The tenant context is extracted from the JWT claim at the API gateway layer.

### 4.2 _tenant parameter

The `_tenant` parameter is a ZarishSphere custom search parameter:

```
GET /fhir/R5/Patient?_tenant=tenant-org-01&family=Ahmed
```

If `_tenant` is not provided in the request, the server defaults to the tenant encoded in the JWT. Explicitly providing a different tenant value returns HTTP 403 Forbidden.

> **Constraint:** Cross-tenant search is not permitted. No search parameter or modifier can override the tenant boundary. The tenant filter is enforced at the query level by appending a `meta.tag` filter to every search.

---

## 5. Pagination rules

### 5.1 Pagination behaviour

| Rule | Value |
|---|---|
| Default page size | 20 |
| Maximum page size | 200 |
| Pagination model | Offset-based (`_offset` + `_count`) |
| Response format | `Bundle.type = searchset` |
| Navigation links | `Bundle.link` with `self`, `next`, `prev` relations |

### 5.2 Pagination example

```json
{
  "resourceType": "Bundle",
  "type": "searchset",
  "total": 1247,
  "link": [
    { "relation": "self", "url": "/fhir/R5/Patient?family=Ahmed&_count=20&_offset=0" },
    { "relation": "next", "url": "/fhir/R5/Patient?family=Ahmed&_count=20&_offset=20" }
  ],
  "entry": [...]
}
```

### 5.3 Large result sets

For searches returning more than 10,000 matching resources, the server SHOULD return a `warning` in the search response and suggest a more specific query. The `total` field MUST still report the accurate count.

---

## 6. Performance requirements

### 6.1 Response time targets

| Search type | Max response time | Index strategy |
|---|---|---|
| Single resource by ID | 10ms | UUID primary key lookup |
| Simple search (single indexed field) | 100ms | PostgreSQL GIN index |
| Complex chained search (2+ hops) | 500ms | Query plan optimisation |
| Full-text search (name, address) | 50ms | Typesense or PostgreSQL full-text index |
| Aggregation / count query | 200ms | Materialised count cache |

### 6.2 Indexing requirements

| Search pattern | Index type |
|---|---|
| `identifier` searches | Unique B-tree on `identifier.system` + `identifier.value` |
| `_lastUpdated` ordering | B-tree on `meta.lastUpdated` descending |
| Tag-based filtering | GIN on `meta.tag` |
| Full-text name search | GIN on `name` (PostgreSQL tsvector) or Typesense |
| Code-based searches | GIN on `code.coding` |
| Reference searches | B-tree on reference fields (`subject`, `patient`) |

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/005-fhir-r5-conventions.md** — Searchset bundle format and pagination rules (parallel definition)
→ **005-zs-standards/010-api-rest-design-guide.md** — REST API design and pagination conventions (parallel definition)
→ **005-zs-standards/006-fhir-profiling-policy.md** — Profile definition for searched resources
→ **003-zs-platform/005-fhir-architecture.md** — FHIR engine architecture implementing search

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
