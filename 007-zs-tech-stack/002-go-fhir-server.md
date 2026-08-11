# 002-go-fhir-server.md
## Go-native FHIR R5 server specification
### Zero-JVM, Plane 0-compatible FHIR server with custom Go implementation

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Architecture overview](#1-architecture-overview)
2. [Tech stack and dependencies](#2-tech-stack-and-dependencies)
3. [FHIR R5 resource support](#3-fhir-r5-resource-support)
4. [Storage layer](#4-storage-layer)
5. [Search and query implementation](#5-search-and-query-implementation)
6. [Transaction processing](#6-transaction-processing)
7. [Performance targets by plane](#7-performance-targets-by-plane)
8. [Security](#8-security)
9. [Cross-references](#9-cross-references)

---

## 1. Architecture overview

The ZarishSphere FHIR server is a custom Go implementation (`zs-fhir-server`) that provides a FHIR R5 RESTful API. It runs as a single Go binary whose only external dependency is its storage backend: PostgreSQL 18.4 (Planes 2-4) or SQLite (Planes 0-1). The server is built on the fhir-toolbox-go FHIR R5 engine, the primary Go FHIR R5 toolkit in the ecosystem.

### 1.1 Design principles

| Principle | Description |
|---|---|
| Zero JVM | No Java runtime, no HAPI FHIR, no JVM anywhere in the stack |
| Single binary | One compiled Go binary with embedded SQLite (Planes 0-1) or PostgreSQL client (Planes 2-4) |
| Plane 0 first | Full functionality on air-gapped systems with 8 GB RAM |
| R5 native | FHIR 5.0.0 native resource model (ADR-005) |
| Offline capable | All CRUD operations work without network |
| Minimal dependencies | Three external Go modules max for FHIR processing, built on fhir-toolbox-go |

### 1.2 High-level architecture

```
HTTP Request → Chi Router → FHIR Handler Layer → Resource Validation → Storage Layer → PostgreSQL / SQLite
                 │                  │
                 │            SMART on FHIR
                 │            OAuth 2.0 Proxy
                 │
            Middleware Chain:
            - Logger
            - Rate limiter
            - Auth (JWT / OAuth)
            - Request ID
            - CORS
            - Compression
```

> **Constraint:** The FHIR server must start and respond to requests within 500 ms on a Raspberry Pi 5 (Plane 1) and within 100 ms on a laptop (development).

### 1.3 Repository structure

```
zs-fhir-server/
├── cmd/
│   └── zs-fhir-server/    — Main entry point
├── internal/
│   ├── api/               — HTTP handlers, routes
│   ├── fhir/              — FHIR type definitions, validation
│   ├── storage/           — PostgreSQL / SQLite storage layer
│   ├── search/            — Search parameter parsing and execution
│   ├── transaction/       — Batch/transaction bundle processing
│   ├── auth/              — Auth middleware, SMART on FHIR
│   └── config/            — Configuration, CLI flags
├── go.mod
└── go.sum
```

---

## 2. Tech stack and dependencies

### 2.1 Go modules

| Module | Version | Purpose |
|---|---|---|
| Go | 1.26.5 | Compiler and standard library |
| fhir-toolbox-go | v0.1.2 | Primary FHIR R5 engine — FHIRPath, REST, search |
| go-chi/chi/v5 | v5.2.1 | HTTP router and middleware |
| jackc/pgx/v5 | v5.7.2 | PostgreSQL driver and connection pool (Planes 2-4) |
| mattn/go-sqlite3 | v1.14 | SQLite database driver (Planes 0-1) |
| fastenhealth/gofhir-models | v0.0.7 | Generated FHIR R5 Go structs |
| golang-jwt/jwt/v5 | v5.2.1 | JWT authentication |
| rs/cors | v1.11 | CORS middleware |
| uber-go/zap | v1.27 | Structured logging |
| google/uuid | v1.6 | UUID generation |
| stretchr/testify | v1.10 | Testing assertions |

### 2.2 Explicit non-dependencies

| Library | Reason avoided |
|---|---|
| HAPI FHIR (Java) | JVM dependency, violates Law 11 |
| gin-gonic/gin | Non-standard context; Chi preferred for net/http compatibility |
| GORM | Heavy ORM; raw SQL/sqlx preferred for performance |
| MongoDB | Document store unnecessary for FHIR; PostgreSQL JSONB chosen |

---

## 3. FHIR R5 resource support

### 3.1 V1 resource support

The V1 implementation supports the following FHIR R5 resources, selected for the initial non-communicable disease (NCD) clinic use case:

| Resource | FHIR R5 path | Profile | Purpose |
|---|---|---|---|
| Patient | `Patient` | `zs-patient` | Patient demographics with ZS-UID |
| Encounter | `Encounter` | `zs-encounter` | Clinical encounters |
| Observation | `Observation` | `zs-observation` | Vital signs, lab results |
| Condition | `Condition` | `zs-condition` | Diagnoses and problem list |
| MedicationRequest | `MedicationRequest` | `zs-medication` | Prescriptions |
| Questionnaire | `Questionnaire` | `zs-questionnaire` | Form definitions |
| QuestionnaireResponse | `QuestionnaireResponse` | `zs-questionnaire-response` | Submitted form data |
| Bundle | `Bundle` | — | Batch, transaction, search results |
| OperationOutcome | `OperationOutcome` | — | Error and warning responses |

### 3.2 Planned V1.1 resources

| Resource | Path | Use case |
|---|---|---|
| CarePlan | `CarePlan` | Care plans and treatment pathways |
| CareTeam | `CareTeam` | Multi-provider care teams |
| ServiceRequest | `ServiceRequest` | Diagnostic and referral requests |
| Task | `Task` | Workflow tasks and follow-ups |
| DiagnosticReport | `DiagnosticReport` | Lab and imaging reports |

### 3.3 Resource validation

Each resource is validated against:
- FHIR R5 base schema constraints (cardinality, types, code systems)
- ZarishSphere profile extensions (ZS-UID, deployment context)
- Business rule validation (required fields for NCD domain)

> **Constraint:** A resource that fails validation must return a 422 Unprocessable Entity response with an OperationOutcome describing each validation failure.

### 3.4 Profile URLs

All ZarishSphere profiles are published at:
```
https://zarishsphere.com/fhir/StructureDefinition/zs-[resource]
```

Example: `https://zarishsphere.com/fhir/StructureDefinition/zs-patient`

---

## 4. Storage layer

### 4.1 PostgreSQL / SQLite schema design

FHIR resources are stored in a hybrid schema that combines relational columns for searchable fields with JSON columns for the full FHIR resource payload. PostgreSQL 18.4 uses JSONB with GIN indexes (Planes 2-4); SQLite uses JSON columns with FTS5 full-text search (Planes 0-1).

PostgreSQL schema (primary, Planes 2-4):

```sql
CREATE TABLE fhir_resources (
    id            TEXT PRIMARY KEY,          -- FHIR logical ID
    resource_type TEXT NOT NULL,              -- e.g., 'Patient', 'Observation'
    version_id    TEXT NOT NULL,              -- FHIR version ID (UUID v4)
    last_updated  TIMESTAMPTZ NOT NULL,       -- ISO 8601 timestamp
    resource_json JSONB NOT NULL,             -- Full FHIR resource as JSONB
    is_deleted    BOOLEAN DEFAULT FALSE,      -- Soft delete flag
    search_fields JSONB,                      -- Materialized search params (JSONB)
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_resource_type ON fhir_resources(resource_type);
CREATE INDEX idx_last_updated ON fhir_resources(last_updated);
CREATE INDEX idx_resource_type_id ON fhir_resources(resource_type, id);
CREATE INDEX idx_resource_json_gin ON fhir_resources USING GIN (resource_json jsonb_path_ops);

-- Row-level security for multi-tenant isolation (Plane 2+)
ALTER TABLE fhir_resources ENABLE ROW LEVEL SECURITY;
```

SQLite schema (fallback, Planes 0-1):

```sql
CREATE TABLE fhir_resources (
    id            TEXT PRIMARY KEY,          -- FHIR logical ID
    resource_type TEXT NOT NULL,              -- e.g., 'Patient', 'Observation'
    version_id    TEXT NOT NULL,              -- FHIR version ID (UUID v4)
    last_updated  TEXT NOT NULL,              -- ISO 8601 timestamp
    resource_json TEXT NOT NULL,              -- Full FHIR resource as JSON
    is_deleted    INTEGER DEFAULT 0,          -- Soft delete flag
    search_fields TEXT,                       -- Materialized search params (JSON)
    created_at    TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_resource_type ON fhir_resources(resource_type);
CREATE INDEX idx_last_updated ON fhir_resources(last_updated);
CREATE INDEX idx_resource_type_id ON fhir_resources(resource_type, id);

-- Full-text search index for text fields
CREATE VIRTUAL TABLE fhir_fts USING fts5(
    resource_type, id, content, tokenize='porter'
);
```

On Planes 2+, the TimescaleDB 2.29.0 extension provides hypertables for time-series vitals and event data.

### 4.2 CRUD operations

| Operation | FHIR method | Storage implementation |
|---|---|---|
| Create | POST `/[type]` | INSERT with generated UUID and version_id |
| Read | GET `/[type]/[id]` | SELECT by resource_type + id |
| Update | PUT `/[type]/[id]` | UPDATE with new version_id, If-Match support |
| Delete | DELETE `/[type]/[id]` | Soft delete (is_deleted = 1) |
| VRead | GET `/[type]/[id]/_history/[vid]` | History table lookup |
| History | GET `/[type]/[id]/_history` | SELECT from history table |

### 4.3 History table

All resource versions are preserved in a separate history table for audit and rollback:

```sql
CREATE TABLE fhir_resource_history (
    id            TEXT NOT NULL,
    resource_type TEXT NOT NULL,
    version_id    TEXT NOT NULL,
    last_updated  TEXT NOT NULL,
    resource_json TEXT NOT NULL,
    operation     TEXT NOT NULL,  -- 'create', 'update', 'delete'
    PRIMARY KEY (resource_type, id, version_id)
);
```

### 4.4 Concurrency

- PostgreSQL: MVCC concurrency, row-level security (RLS), connection pool via pgx
- SQLite (Planes 0-1): WAL mode enables concurrent reads during writes; write locks are held for the minimum duration per transaction
- Optimistic locking via `If-Match` header and version_id comparison
- SQLite: maximum 5 concurrent write operations before queuing

---

## 5. Search and query implementation

### 5.1 Supported search parameters

For V1, the following search parameter types are supported:

| Parameter type | Description | Example |
|---|---|---|
| Token | CodeableConcept, Identifier, status | `Patient.identifier=ZS-NCD-2026-000001` |
| String | Text search on name, address | `Patient.name=Smith` |
| Reference | Resource references | `Observation.subject=Patient/123` |
| Date | Date and datetime ranges | `Observation.date=ge2026-01-01` |
| Number | Numeric comparisons | `Observation.value-quantity=ge140` |
| Composite | Combined parameter search | N/A in V1 |

### 5.2 Query compilation

Search parameters are parsed from URL query strings and compiled into SQL WHERE clauses:

```go
type SearchQuery struct {
    ResourceType string
    Filters     []FilterClause
    Sort        []SortClause
    Count       int    // _count
    Page        int    // offset based on _count
    Include     []string  // _include
    RevInclude  []string  // _revinclude
    Summary     string // _summary
    Elements    []string // _elements
}
```

### 5.3 Pagination

- Default page size: 20 resources
- Maximum page size: 200 resources (enforced server-side)
- Pagination via `_count` and offset-based `_getpagesoffset` parameter
- Self/next/prev links in Bundle.entry

### 5.4 Search result format

All search responses are returned as FHIR Bundle resources of type `searchset`:

```json
{
    "resourceType": "Bundle",
    "type": "searchset",
    "total": 42,
    "entry": [
        {
            "fullUrl": "https://fhir.zarishsphere.com/fhir/R5/Patient/abc-123",
            "resource": { ... },
            "search": { "mode": "match" }
        }
    ]
}
```

---

## 6. Transaction processing

### 6.1 Bundle types supported

| Bundle type | Description | Status |
|---|---|---|
| batch | Independent operations, no rollback | V1 |
| transaction | Atomic operations with full rollback | V1 |
| batch-response | Response for batch bundles | V1 |
| transaction-response | Response for transaction bundles | V1 |
| searchset | Search results | V1 |
| collection | Resource collection | V1 |

### 6.2 Transaction processing pipeline

```
1. Parse Bundle JSON
2. Validate Bundle structure
3. Process bundle.entry in order:
   a. Resolve fullUrl references
   b. Execute operation (POST/PUT/DELETE)
   c. Create conditional references (urn:uuid: → id)
4. BEGIN TRANSACTION (for transaction bundles)
5. Execute all operations
6. If any fails → ROLLBACK, return OperationOutcome
7. If all succeed → COMMIT, return Bundle with responses
```

### 6.3 Conditional operations

- `If-None-Exist` header for conditional create
- `If-Match` header for version-aware update
- `If-Modified-Since` and `If-None-Match` for conditional read
- Conditional references in bundles using `urn:uuid:` and `urn:oid:` identifiers

---

## 7. Performance targets by plane

### 7.1 Performance table

| Metric | Plane 0 (air-gapped) | Plane 1 (RPi 5) | Plane 2 (district) | Plane 3 (national) | Plane 4 (global) |
|---|---|---|---|---|---|
| Hardware | Lenovo i3/8GB | RPi 5/8GB | Xeon/32GB | Cloud VM/64GB | Cloud fleet |
| Storage | SQLite file | SQLite file | PostgreSQL 18.4 | PostgreSQL 18.4 + replicas | PostgreSQL fleet + R2 replicas |
| Requests/sec (read) | 2,000 | 500 | 5,000 | 10,000 | 50,000+ |
| Requests/sec (write) | 500 | 150 | 1,200 | 2,500 | 10,000+ |
| P99 latency (read) | <50 ms | <200 ms | <30 ms | <20 ms | <10 ms |
| P99 latency (write) | <100 ms | <500 ms | <80 ms | <50 ms | <30 ms |
| Concurrent users | 1-5 | 1-10 | 50-200 | 500-2,000 | 10,000+ |
| Database size limit | 10 GB | 10 GB | 100 GB | 500 GB | Sharded |
| Cold start time | <100 ms | <500 ms | <200 ms | <1 s (with cache) | N/A (always hot) |

### 7.2 Target formula

```
Plane scaling factor:
  Plane 0 = baseline
  Plane 1 = 0.25x (RPi ARM64, lower clock)
  Plane 2 = 2.5x (better hardware, dedicated)
  Plane 3 = 5x (cloud VM, tuned)
  Plane 4 = 25x+ (horizontal scaling, read replicas)
```

---

## 8. Security

### 8.1 Authentication

| Plane | Method | Details |
|---|---|---|
| Plane 0 | None / shared secret | No network; optional local auth token |
| Plane 1 | JWT (pre-shared key) | Local JWT with symmetric key |
| Plane 2 | JWT + OAuth 2.0 | Keycloak 26.7.0 |
| Plane 3 | OAuth 2.0 / OIDC | Integration with national IdP |
| Plane 4 | SMART on FHIR 2.1 | Full OAuth 2.1 + SMART scopes |

### 8.2 Authorization

- **SMART on FHIR scopes** for granular resource-level permissions
- Scopes follow the pattern: `patient/*.read`, `user/Observation.write`
- Scope enforcement at the middleware layer before handler execution
- Keycloak integration for Plane 2+ deployments

### 8.3 Transport security

| Plane | TLS | Notes |
|---|---|---|
| Plane 0 | Not required | Loopback or no network |
| Plane 1 | Optional self-signed | Local network only |
| Plane 2 | Required (Let's Encrypt) | Public-facing deployment |
| Plane 3 | Required (managed CA) | National deployment |
| Plane 4 | Required (managed CA) | Global SaaS |

### 8.4 Audit

All write operations are logged to the FHIR audit event model:

- Server-level: `AuditEvent` resource for every create, update, delete
- User-level: Who performed the operation, from which IP
- Data-level: What changed (before/after resource snapshot in history table)

---

## 9. Cross-references

→ **007-zs-tech-stack/001-tech-stack-master.md** — Master production tech stack mapping
→ **007-zs-tech-stack/006-oss-tool-catalog.md** — Authoritative, version-pinned OSS tool catalog
→ **003-zs-platform/005-fhir-architecture.md** — FHIR R5 server architecture, resource profiles, and compliance roadmap
→ **005-zs-standards/005-fhir-r5-conventions.md** — FHIR R5 implementation conventions
→ **005-zs-standards/006-fhir-profiling-policy.md** — FHIR profile authoring policy (zs-* profiles)
→ **008-zs-adrs/001-adr-go-as-primary-language.md** — ADR-001: Go as the primary language
→ **008-zs-adrs/004-adr-no-hapi-fhir.md** — ADR-004: no HAPI FHIR / no JVM
→ **008-zs-adrs/005-adr-fhir-r5-over-r4.md** — ADR-005: FHIR R5 over R4
→ **008-zs-adrs/013-adr-postgresql-primary-database.md** — ADR-013: PostgreSQL 18.4 primary database
→ **004-zs-index/003-metadata-schema.md** — ZarishIndex metadata schema (ZS-UID and identifiers)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
