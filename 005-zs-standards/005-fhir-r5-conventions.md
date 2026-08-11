# 005-fhir-r5-conventions.md
## FHIR R5 resource conventions
### Resource IDs, tenant isolation, metadata, audit, response format — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all FHIR R5 resource handling in the ZarishSphere Platform

---

## Table of contents

1. [Resource ID convention](#1-resource-id-convention)
2. [Tenant isolation pattern](#2-tenant-isolation-pattern)
3. [Required metadata on all resources](#3-required-metadata-on-all-resources)
4. [AuditEvent requirements](#4-auditevent-requirements)
5. [Search response bundle format](#5-search-response-bundle-format)
6. [Error response format](#6-error-response-format)

---

## 1. Resource ID convention

### 1.1 UUID v7 requirement

All FHIR resource IDs in the ZarishSphere Platform MUST be UUID v7 format. UUID v7 is time-ordered, enabling chronological sorting without a separate timestamp index.

```go
// UUID v7 — time-ordered, cluster-safe
// Format: 0192fbad-xxxx-7xxx-xxxx-xxxxxxxxxxxx
id := db.UUIDv7()
```

PostgreSQL 18.4 provides `uuidv7()` natively. Where the database layer does not support UUID v7, the application layer MUST generate it.

### 1.2 Resource ID rules

| Rule | Description |
|---|---|
| Format | Must be UUID v7 (`xxxxxxxx-xxxx-7xxx-xxxx-xxxxxxxxxxxx`) |
| Uniqueness | Must be unique within the tenant scope |
| Reuse | Never reuse a resource ID after the resource is deleted |
| Assignment | Always server-assigned; client-supplied IDs are rejected on create |

> **Constraint:** A FHIR resource ID, once assigned, is immutable for the lifetime of that resource. If a resource is deleted, its ID enters a tombstone state and must never be reassigned to a different resource.

---

## 2. Tenant isolation pattern

### 2.1 Meta tag structure

Every FHIR resource stored in the ZarishSphere Platform MUST include a tenant identifier in `meta.tag`. This enables multi-tenant data isolation at the database layer without separate databases per tenant.

```json
{
  "resourceType": "Patient",
  "id": "0192fbad-1234-7abc-def0-123456789012",
  "meta": {
    "lastUpdated": "2026-03-24T10:30:00Z",
    "versionId": "1",
    "tag": [
      {
        "system": "https://zarishsphere.com/tags/tenant",
        "code": "tenant-org-01",
        "display": "Tenant Organization 01"
      },
      {
        "system": "https://zarishsphere.com/tags/program",
        "code": "program-refugee-response",
        "display": "Refugee Response Program"
      }
    ]
  }
}
```

### 2.2 Tag system URIs

| Tag | System URI | Required |
|---|---|---|
| Tenant | `https://zarishsphere.com/tags/tenant` | Yes |
| Program | `https://zarishsphere.com/tags/program` | Conditional — required for multi-program deployments |
| Translation | `https://zarishsphere.com/tags/translation` | Conditional — present on R4 bridge translated resources |

### 2.3 Tenant resolution

The tenant context is resolved at the API gateway layer from the JWT claim. The `meta.tag` value MUST match the authenticated tenant. Cross-tenant access returns HTTP 403.

> **Constraint:** No FHIR query or write operation may span multiple tenants. The tenant filter is applied automatically from the authentication context and cannot be overridden by the client.

---

## 3. Required metadata on all resources

### 3.1 Mandatory meta fields

Every FHIR resource stored in the ZarishSphere Platform MUST include these `meta` fields:

| Field | Requirement | Set by |
|---|---|---|
| `meta.lastUpdated` | Required — instant of last write | Server (always) |
| `meta.versionId` | Required — positive integer, incremented on every update | Server (always) |
| `meta.source` | Required — URI identifying the system that created or last updated the resource | Server or client |
| `meta.tag` | Required — at minimum the tenant tag | Server (from auth context) |

```json
{
  "meta": {
    "lastUpdated": "2026-03-24T10:30:00Z",
    "versionId": "1",
    "source": "https://fhir.zarishsphere.com/zs-svc-patient",
    "tag": [
      {
        "system": "https://zarishsphere.com/tags/tenant",
        "code": "tenant-org-01",
        "display": "Tenant Organization 01"
      }
    ]
  }
}
```

### 3.2 Metadata validation

The FHIR server MUST reject any write operation (create or update) that omits required meta fields. Rejection returns an `OperationOutcome` with severity `error` and code `required`.

---

## 4. AuditEvent requirements

### 4.1 Operations requiring audit

Every read, write, search, or delete operation on a FHIR resource containing protected information MUST generate a FHIR `AuditEvent`.

| Operation | Trigger | Audit action code |
|---|---|---|
| Read | `GET /fhir/R5/{ResourceType}/{id}` | `R` |
| Search | `GET /fhir/R5/{ResourceType}?...` | `R` |
| Create | `POST /fhir/R5/{ResourceType}` | `C` |
| Update | `PUT /fhir/R5/{ResourceType}/{id}` | `U` |
| Delete | `DELETE /fhir/R5/{ResourceType}/{id}` | `D` |
| Export | `POST /fhir/R5/$export` | `R` |
| Authentication failure | Any authenticated endpoint → 401 | `E` |

Protected resources include: `Patient`, `Encounter`, `Observation`, `Condition`, `MedicationRequest`, `Procedure`, `AllergyIntolerance`, `DiagnosticReport`, `DocumentReference`, `CarePlan`, `Consent`, `NutritionOrder`, and any resource containing personally identifiable data as defined by the deployment jurisdiction.

### 4.2 AuditEvent structure

```json
{
  "resourceType": "AuditEvent",
  "type": {
    "system": "http://terminology.hl7.org/CodeSystem/audit-event-type",
    "code": "rest"
  },
  "action": "C",
  "recorded": "2026-03-24T10:30:00Z",
  "outcome": "0",
  "agent": [{
    "who": {
      "identifier": { "value": "keycloak-user-uuid" }
    },
    "requestor": true
  }],
  "entity": [{
    "what": {
      "reference": "Patient/0192fbad-1234-7abc-def0-123456789012"
    },
    "type": { "code": "2" }
  }]
}
```

### 4.3 AuditEvent field requirements

| Field | Requirement |
|---|---|
| `type` | Must be `rest` for REST API operations |
| `action` | `C` (create), `R` (read), `U` (update), `D` (delete), `E` (execute) |
| `recorded` | ISO 8601 UTC timestamp — must be current server time |
| `outcome` | `0` (success), `4` (minor failure), `8` (serious failure), `12` (major failure) |
| `agent[0].who` | Must reference the authenticated user |
| `entity[0].what` | Must reference the affected resource |

> **Constraint:** AuditEvent resources are write-once, never-modified records. No update or delete operation is permitted on an AuditEvent resource. Retention is governed by the platform audit policy (see → **[005-zs-standards/008-fhir-audit-policy.md]**).

---

## 5. Search response bundle format

### 5.1 Searchset bundle structure

All FHIR search responses return a `Bundle` of type `searchset`:

```json
{
  "resourceType": "Bundle",
  "type": "searchset",
  "total": 47,
  "link": [
    {
      "relation": "self",
      "url": "/fhir/R5/Patient?_count=20&_offset=0"
    },
    {
      "relation": "next",
      "url": "/fhir/R5/Patient?_count=20&_offset=20"
    }
  ],
  "entry": [...]
}
```

### 5.2 Pagination rules

| Rule | Value |
|---|---|
| Default page size | 20 |
| Maximum page size | 200 |
| Pagination mechanism | Offset-based via `_offset` parameter |
| Link relations | `self`, `next`, `prev` (if applicable) |

> **Constraint:** The server must never return all matching resources without pagination. An explicit `_count` or `_offset` parameter is required for all search requests.

---

## 6. Error response format

### 6.1 OperationOutcome structure

All errors return a FHIR `OperationOutcome` resource:

```json
{
  "resourceType": "OperationOutcome",
  "issue": [{
    "severity": "error",
    "code": "invalid",
    "diagnostics": "Patient.name is required",
    "expression": ["Patient.name"]
  }]
}
```

### 6.2 HTTP status code mapping

| HTTP status | When used | OperationOutcome severity |
|---|---|---|
| `200` | Successful read or search | (none — success) |
| `201` | Successful create (with `Location` header) | (none — success) |
| `400` | Bad request / validation failure | `error` |
| `401` | Missing or invalid authentication | `error` |
| `403` | Insufficient SMART scope or cross-tenant access | `error` |
| `404` | Resource not found | `error` |
| `409` | Version conflict (optimistic locking) | `error` |
| `422` | FHIR validation failure (unprocessable entity) | `error` |
| `500` | Internal server error | `fatal` |

### 6.3 Issue code values

Use FHIR R5 `issue-type` codes from `http://hl7.org/fhir/ValueSet/issue-type`. Common values:

| Code | Use |
|---|---|
| `invalid` | Field validation failure |
| `required` | Missing required field |
| `not-found` | Resource not found |
| `duplicate` | Duplicate resource ID |
| `forbidden` | Authorization failure |
| `login` | Authentication required |
| `conflict` | Version ID conflict |
| `exception` | Unexpected server error |

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/006-fhir-profiling-policy.md** — FHIR profile derivation rules for ZS-* resources
→ **005-zs-standards/007-fhir-search-standards.md** — Search parameter and pagination rules (parallel definition)
→ **005-zs-standards/008-fhir-audit-policy.md** — AuditEvent retention and platform audit policy
→ **005-zs-standards/010-api-rest-design-guide.md** — REST API design and pagination conventions (parallel definition)
→ **005-zs-standards/004-standards-to-platform-pipeline.md** — Enforcement pipeline that consumes these conventions

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
