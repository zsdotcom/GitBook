# 008-fhir-audit-policy.md
## FHIR audit policy
### AuditEvent generation, retention, and review — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all audit logging in the ZarishSphere Platform

---

## Table of contents

1. [Operations requiring audit](#1-operations-requiring-audit)
2. [AuditEvent implementation](#2-auditevent-implementation)
3. [Audit log retention](#3-audit-log-retention)
4. [Audit review schedule](#4-audit-review-schedule)

---

## 1. Operations requiring audit

### 1.1 Mandatory audit triggers

The following operations MUST generate a FHIR `AuditEvent`:

| Operation | Trigger | Action code |
|---|---|---|
| Read protected resource | `GET /fhir/R5/{ResourceType}/{id}` | `R` |
| Search protected resources | `GET /fhir/R5/{ResourceType}?...` | `R` |
| Create protected resource | `POST /fhir/R5/{ResourceType}` | `C` |
| Update protected resource | `PUT /fhir/R5/{ResourceType}/{id}` | `U` |
| Delete protected resource | `DELETE /fhir/R5/{ResourceType}/{id}` | `D` |
| Export data | `POST /fhir/R5/$export` | `R` |
| Failed authentication | Any authenticated endpoint → 401 | `E` |
| R4↔R5 translation | Bridge translates a resource | `E` |
| Compliance override | Override of a standards validation failure | `E` |

### 1.2 Protected resources

Protected resources are those that contain personally identifiable data or information protected under applicable privacy regulations. The following resource types are always protected:

```
Patient, Encounter, Observation, Condition, MedicationRequest,
Procedure, AllergyIntolerance, DiagnosticReport, DocumentReference,
CarePlan, Consent, NutritionOrder, Person, RelatedPerson,
ClinicalImpression, FamilyMemberHistory, QuestionnaireResponse
```

> **Constraint:** The list of protected resource types may be extended by deployment configuration based on jurisdictional requirements. It must never be reduced below the baseline set in §1.2.

### 1.3 Non-protected resources

The following resources do not require audit events for read or search operations:

```
Practitioner, Organization, Location, HealthcareService,
ValueSet, CodeSystem, ConceptMap, StructureDefinition,
Questionnaire, ActivityDefinition, PlanDefinition,
SearchParameter, NamingSystem
```

---

## 2. AuditEvent implementation

### 2.1 Go interface

```go
// zs-pkg-go-audit implements audit logging
// Used by every FHIR handler after successful operation

type AuditLogger interface {
    LogRead(ctx context.Context, resourceType, resourceID, userID, tenantID string) error
    LogCreate(ctx context.Context, resourceType, resourceID, userID, tenantID string) error
    LogUpdate(ctx context.Context, resourceType, resourceID, userID, tenantID string) error
    LogDelete(ctx context.Context, resourceType, resourceID, userID, tenantID string) error
    LogSearch(ctx context.Context, resourceType, query, userID, tenantID string) error
    LogExport(ctx context.Context, userID, tenantID string) error
    LogTranslation(ctx context.Context, resourceType, resourceID, direction string) error
}
```

### 2.2 Async audit log writing

Audit events are written asynchronously to avoid blocking the FHIR API response:

1. **Primary store:** `audit.events` PostgreSQL table (immediate write)
2. **Stream:** NATS subject `zs.audit.event` (for streaming consumers)
3. **Backup:** WAL stream to Cloudflare R2 for off-site retention

```go
// Implementation writes to both PostgreSQL and NATS
// PostgreSQL write is synchronous within the same transaction
// NATS publish is fire-and-forget with retry
```

### 2.3 AuditEvent structure

See → **[005-zs-standards/005-fhir-r5-conventions.md#4-auditevent-requirements]** for the required AuditEvent JSON structure and field requirements.

| Field | Requirement |
|---|---|
| `action` | Must be one of `C`, `R`, `U`, `D`, `E` |
| `recorded` | ISO 8601 UTC — server timestamp at write |
| `outcome` | `0` (success), `4` (minor failure), `8` (serious failure), `12` (major failure) |
| `agent[0].who` | Authenticated user identifier |
| `entity[0].what` | Reference to the affected resource |

> **Constraint:** AuditEvent resources are write-once, never-modified. No API endpoint exists to update or delete an AuditEvent. Any modification requires direct database access with documented approval.

---

## 3. Audit log retention

### 3.1 Retention periods

| Environment | Retention period | Storage |
|---|---|---|
| Plane 3-4 (cloud) | 7 years | Centralised `audit.events` PostgreSQL table with partition pruning |
| Plane 2 (district server) | 2 years | Local PostgreSQL with monthly export |
| Plane 1 (RPi edge) | 1 year | Local PostgreSQL with quarterly export |
| Plane 0 (air-gapped) | 2 years | Local storage with operator-scheduled export |

### 3.2 Backup and archival

- Audit data is included in regular PostgreSQL backups
- Backups are stored in Cloudflare R2 with cross-region replication
- Partitioned tables use monthly partitions for efficient pruning
- After the retention period, partitions are archived to cold storage (not deleted immediately)

### 3.3 Access control

| Role | Access level |
|---|---|
| System (automated) | Write only |
| Audit reviewer | Read only on audit tables |
| Application user | No access to audit tables |

---

## 4. Audit review schedule

### 4.1 Review cadence

| Review type | Frequency | Scope |
|---|---|---|
| Authentication failure review | Weekly | Review failed login attempts and unauthorised access patterns |
| Access pattern review | Monthly | Identify unusual access patterns, data volume anomalies |
| Full audit log review | Quarterly | Comprehensive review of all audit events across all operations |
| Compliance review | Annual | Verify compliance with GDPR, HIPAA, PDPA, and other applicable frameworks |

### 4.2 Review tooling

Audit review is performed through the ZarishSphere Console compliance dashboard, which provides:

- Time-range filtering
- User and resource-type aggregation
- Anomaly detection alerts
- Export to CSV for external audit

> **Constraint:** Audit reviewers must have documented delegation from the ZarishSphere Foundation governance body. No single individual may review their own audit trail. This is enforced by the platform access control system (see → **[008-zs-adrs/012-adr-no-single-person-dependency.md]**).

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/005-fhir-r5-conventions.md** — AuditEvent structure and field requirements (§4)
→ **005-zs-standards/004-standards-to-platform-pipeline.md** — Audit logging in the enforcement pipeline (§7)
→ **008-zs-adrs/012-adr-no-single-person-dependency.md** — Governance ADR enforced by audit access control

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
