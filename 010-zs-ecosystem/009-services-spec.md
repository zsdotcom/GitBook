# 009-services-spec.md
## ZarishSphere Services specification
### Backend microservices

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Service catalog](#2-service-catalog)
3. [Service principles](#3-service-principles)
4. [Plane 0 adaptation](#4-plane-0-adaptation)
5. [Service architecture](#5-service-architecture)
6. [Service registry and discovery](#6-service-registry-and-discovery)
7. [Identity service](#7-identity-service)
8. [Audit service](#8-audit-service)
9. [Sync service](#9-sync-service)
10. [Notification service](#10-notification-service)
11. [Export service](#11-export-service)
12. [Integration service](#12-integration-service)
13. [Health and readiness endpoints](#13-health-and-readiness-endpoints)
14. [Cross-references](#14-cross-references)

---

## 1. Purpose

ZarishSphere Services are the backend microservices that power the ecosystem. Each service is independently deployable, communicates via APIs, and follows the Plane 0 constraint.

## 2. Service catalog

| Service | Purpose | Technology |
|---|---|---|
| Identity service | Authentication, authorization, user management | Keycloak + Go middleware |
| Audit service | Immutable audit log for all operations | Go + PostgreSQL |
| Sync service | Offline data synchronization across planes | Go + NATS JetStream |
| Notification service | Email, SMS, in-app notifications | Go + SMTP / Twilio |
| Export service | Data export in open formats | Go + FHIR / CSV / JSON / Parquet |
| Integration service | Webhook and API gateway for external systems | Go + NATS |

## 3. Service principles

- **Independently deployable** — each service runs as a separate binary
- **API-only communication** — no shared databases between services
- **Stateless where possible** — state delegated to PostgreSQL or NATS
- **Plane 0 capable** — all services run on minimal hardware
- **Full audit trail** — every operation is logged in the audit service

## 4. Plane 0 adaptation

At Plane 0, services consolidate:

- **Identity** — local auth (no Keycloak). Uses bcrypt-hashed local accounts stored in SQLite. Admin credentials provisioned at setup time.
- **Audit** — SQLite-backed append-only log. Exported as NDJSON on sync. Local retention limit: 100,000 entries or 30 days.
- **Sync** — USB bundle creation instead of NATS. Bundle format: tar.gz with SQLite snapshot + media files.
- **Notification** — local display only (console banner, log file). No SMTP or SMS delivery.
- **Export** — file-system export only. No streaming or cloud upload.
- **Integration** — disabled. Webhooks and external API calls are queued for sync-time execution.

## 5. Service architecture

All services share a common architecture base:

```
┌─────────────────────────────────────────────┐
│                Service binary                │
│  ┌─────────┐ ┌──────────┐ ┌──────────────┐  │
│  │ HTTP/RPC │ │ Business │ │ Data access  │  │
│  │ handler  │ │ logic    │ │ layer        │  │
│  └────┬─────┘ └────┬─────┘ └──────┬───────┘  │
│       │            │              │          │
│  ┌────▼────────────▼──────────────▼───────┐  │
│  │      Common middleware stack           │  │
│  │  Auth → Audit → Rate limit → Tracing   │  │
│  └────────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

**Communication patterns:**

| Pattern | Protocol | Use case |
|---|---|---|
| Synchronous request-response | gRPC (Go) | Service-to-service calls within same plane |
| Asynchronous messaging | NATS JetStream | Event-driven workflows, sync triggers |
| HTTP REST | JSON/HTTPS | External API consumption (SDK, CLI, webhooks) |
| File-based | Local filesystem | Plane 0 fallback for all patterns |

**Technology base:** All services are written in Go 1.22+ with shared libraries from → **[010-zs-ecosystem/012-engine-spec.md](010-zs-ecosystem/012-engine-spec.md)**. PostgreSQL is the primary database at Planes 1-4. SQLite is used at Plane 0. NATS JetStream provides message queuing at Planes 2-4.

## 6. Service registry and discovery

Services discover each other through a lightweight registry:

- **Planes 2-4** — NATS JetStream key-value store serves as the service registry. Each service registers its address and health status on startup.
- **Plane 1** — static configuration file (`/etc/zarishsphere/services.yaml`) listing all service addresses.
- **Plane 0** — all services run as a single binary with in-process routing. No external registry needed.

Registration entry format:

```yaml
identity:
  address: "192.168.1.10:8443"
  health: "passing"
  version: "1.0.0"
  plane: 2
  tags: ["core", "auth"]
```

The registry supports health-based routing: unhealthy services are removed from the pool automatically. Clients poll the registry every 30 seconds.

## 7. Identity service

The Identity service manages all authentication, authorization, and user lifecycle operations.

### Capabilities

- **User management** — create, read, update, deactivate accounts. Supports email, username, and federated identity.
- **Role-based access control (RBAC)** — hierarchical roles: `system_admin`, `plane_admin`, `org_admin`, `operator`, `viewer`. Custom roles are defined through the Console.
- **Permission model** — granular permissions per resource type (module, form, app, distribution). Permissions are additive; deny rules can be set for specific resources.
- **Authentication flows** — OAuth2 authorization code flow (browser), OAuth2 client credentials (machine-to-machine), API token (headless/CLI), local password (Plane 0).
- **OIDC provider** — OpenID Connect discovery endpoint at `/.well-known/openid-configuration`. Supports standard claims: sub, name, email, groups.
- **Session management** — JWT access tokens (15-minute expiry) with refresh tokens (7-day expiry, single-use). Token revocation via blacklist.

### Plane 0 behaviour

At Plane 0, the Identity service uses a local SQLite database with bcrypt-hashed passwords. No OAuth2 or OIDC. Admin account is created during initial setup. Additional local accounts can be provisioned through the Console. Tokens use HMAC-SHA256 signing with a local secret key stored in the System layer's encrypted config store.

## 8. Audit service

The Audit service provides an immutable, tamper-evident log of all operations across the ecosystem.

### Log entry format

```json
{
  "id": "aud_9a8b7c6d5e4f",
  "timestamp": "2026-06-11T10:30:00Z",
  "actor": "user:alice@zarishsphere.com",
  "action": "module.deploy",
  "resource": "module:immunization-tracker",
  "target_plane": 2,
  "source_ip": "192.168.1.100",
  "user_agent": "zs-cli/1.0.0",
  "outcome": "success",
  "details": {
    "version": "1.2.0",
    "config_hash": "sha256-abc123"
  }
}
```

### Storage and retention

- **Primary store** — PostgreSQL (Planes 1-4), SQLite (Plane 0)
- **Append-only** — no updates or deletes. Log entries are inserted, never modified.
- **Retention policy** — 90 days hot storage, then archived to Parquet files. Plane 0 retains 30 days.
- **Integrity** — each entry includes a SHA-256 hash of the previous entry forming a hash chain. Tampering is detectable by re-computing the chain.
- **Capacity** — approximately 5 bytes per log entry, ~500 million entries per TB of storage.

### Query API

The audit service exposes a query API with the following filters:

`GET /api/v1/audit?actor=user:alice&action=module.deploy&since=2026-06-01&until=2026-06-11&outcome=failure&limit=100&offset=0`

### Export

Audit logs are exportable as NDJSON or Parquet for external SIEM integration:

```bash
zs data export --service audit --format ndjson --since 2026-05-01 --out audit-export.ndjson
```

## 9. Sync service

The Sync service manages data replication across deployment planes, handling offline periods and network interruptions.

### Sync topology

| Plane combination | Topology | Mechanism |
|---|---|---|
| Plane 0 → Plane 1 | File (USB) | Tar.gz bundle with SQLite snapshot |
| Plane 1 → Plane 2 | Star | NATS JetStream push/pull |
| Plane 2 → Plane 3 | Star | NATS JetStream with compression |
| Plane 3 → Plane 4 | Mesh | NATS JetStream super-cluster |

### Conflict resolution

| Strategy | Description | Default domain |
|---|---|---|
| Last-write-wins (LWW) | Record with most recent `updated_at` timestamp wins. Sufficient for most humanitarian data. | Health, WASH, Nutrition |
| CRDT (merge) | Conflict-free replicated data types for concurrent edits. Used where merge matters. | Education, Human rights |
| Manual review | Conflicts flagged in Console for human resolution. | Protection, Legal aid |

LWW is the default for all domains. CRDT is enabled per-module for domains where data merging is critical. Manual review is triggered when both resolution strategies produce ambiguous results.

### Sync scheduling

- **Continuous** — Planes 2-4 with persistent connectivity
- **Periodic** — Plane 1 (configurable interval, default 5 minutes)
- **On-demand** — Plane 0 (triggered by USB bundle import)
- **Delta-only** — only changed records since last sync are transmitted

### Bundle format (Plane 0 → Plane 1)

```
sync-bundle-2026-06-11.tar.gz
├── manifest.json              ← Bundle metadata, checksums
├── snapshot.sqlite            ← Full SQLite database snapshot
├── media/                     ← Attached files (images, documents)
│   ├── photo-001.jpg
│   └── consent-002.pdf
└── audit.ndjson               ← Pending audit log entries
```

## 10. Notification service

The Notification service delivers alerts, reminders, and system messages to users across multiple channels.

### Supported channels

| Channel | Transport | Plane 0 support | Delivery guarantee |
|---|---|---|---|
| In-app | Console notification drawer | Yes | At-most-once |
| Email | SMTP (SendGrid, AWS SES, or local relay) | No | At-least-once |
| SMS | Twilio or local GSM modem | No (Plane 1+) | At-most-once |
| Webhook | HTTP POST to configured endpoint | Queued only | At-least-once |

### Template system

Notifications use Go text/templates stored in the content repository at `zs-content-notifications`. Templates support parameter substitution:

```yaml
# template: module-deployed.yaml
subject: "Module {{.ModuleName}} deployed to Plane {{.Plane}}"
body: |
  The module "{{.ModuleName}}" version {{.Version}} has been
  deployed to Plane {{.Plane}} by {{.Actor}} at {{.Timestamp}}.
  Status: {{.Status}}
```

Templates are versioned and managed through → **[010-zs-ecosystem/005-forms-spec.md](010-zs-ecosystem/005-forms-spec.md)** conventions.

### Delivery guarantees

- **In-app** — notifications are stored in the database and displayed on next Console load
- **Email** — retry with exponential backoff (3 attempts, 5-minute intervals). Dead letter queue after 3 failures.
- **SMS** — single attempt. No retry (SMS is best-effort).
- **Webhook** — retry with exponential backoff (5 attempts). Signed with HMAC-SHA256 for authenticity verification.

## 11. Export service

The Export service provides bulk data extraction in multiple open formats.

### Supported formats

| Format | File extension | Compression | Use case |
|---|---|---|---|
| FHIR R5 NDJSON | `.ndjson` | gzip | Interoperability with other health systems |
| JSON | `.json` | gzip | General-purpose data exchange |
| CSV | `.csv` | gzip | Spreadsheet analysis, pivot tables |
| Parquet | `.parquet` | Snappy | Analytical workloads, big data pipelines |
| HL7v2 (ER7) | `.hl7` | gzip | Legacy health system integration |
| Flat XML | `.xml` | gzip | Legacy enterprise system export |

### Export configuration

```yaml
export:
  format: "fhir"
  module: "nutrition-screening"
  fields: ["patient_id", "muac_score", "screening_date", "nutrition_status"]
  filter:
    date_from: "2026-01-01"
    date_to: "2026-06-11"
    status: "completed"
  compression: true
  split_size_mb: 500
```

### Scheduling

Exports can be scheduled via the Console or CLI:

```bash
zs data export --module nutrition-screening --format parquet --schedule "0 2 * * *"
```

Scheduled exports support incremental (delta-only) mode, exporting only records modified since the last successful export.

### Plane 0 export

At Plane 0, exports write directly to the local filesystem or to a connected USB drive. The export directory is configurable in the System layer's config store. Maximum file size for Plane 0 is 2 GB.

## 12. Integration service

The Integration service connects ZarishSphere with external systems through webhooks, API proxies, and transformation adapters.

### Webhook management

External systems register webhook endpoints to receive real-time event notifications:

```bash
zs integration webhook create \
    --url "https://external-system.org/webhooks/zarishsphere" \
    --events "module.deployed,data.exported" \
    --secret "whsec_..."
```

Webhook payloads are signed with HMAC-SHA256 using the configured secret. The receiving system verifies the signature using the `X-ZS-Signature` header.

### Supported events

| Event | Triggered when |
|---|---|
| `module.deployed` | Module deployment completes |
| `module.removed` | Module is removed from a plane |
| `data.exported` | Export operation completes |
| `data.synced` | Sync operation completes |
| `user.created` | New user account is created |
| `alert.triggered` | System alert threshold is crossed |

### Transformation adapters

The Integration service supports transformation adapters for non-standard data formats:

- **CSV mapper** — map CSV columns to FHIR resource fields
- **XML translator** — transform legacy XML payloads to JSON API requests
- **JSON path filter** — extract specific fields from incoming webhook payloads
- **Custom adapter** — user-defined Lua scripts for complex transformations

Adapters are configured through the Console and stored in the content repository.

## 13. Health and readiness endpoints

Every service exposes standard health endpoints:

### Liveness endpoint

```
GET /healthz
```

Response (HTTP 200):

```json
{
  "status": "alive",
  "service": "identity",
  "version": "1.0.0",
  "uptime_seconds": 86400
}
```

### Readiness endpoint

```
GET /readyz
```

Response (HTTP 200 when ready, 503 when not):

```json
{
  "status": "ready",
  "dependencies": {
    "postgres": "connected",
    "nats": "connected",
    "keycloak": "connected"
  },
  "last_sync": "2026-06-11T10:00:00Z"
}
```

### Metrics endpoint

```
GET /metrics
```

Prometheus-format metrics at `/metrics` endpoint:

```
# HELP zs_requests_total Total requests processed
# TYPE zs_requests_total counter
zs_requests_total{service="identity",method="GET",path="/api/v1/users"} 15234
# HELP zs_request_duration_seconds Request duration in seconds
# TYPE zs_request_duration_seconds histogram
zs_request_duration_seconds_bucket{le="0.1"} 14000
zs_request_duration_seconds_bucket{le="0.5"} 15000
zs_request_duration_seconds_bucket{le="1"} 15200
zs_request_duration_seconds_bucket{le="+Inf"} 15234
```

All endpoints are defined in → **[010-zs-ecosystem/008-api-spec.md](010-zs-ecosystem/008-api-spec.md)** and follow the → **[003-zs-platform/006-api-design.md](003-zs-platform/006-api-design.md)** conventions.

## 14. Cross-references

→ **010-zs-ecosystem/008-api-spec.md** — API specification: the endpoints each service exposes
→ **010-zs-ecosystem/013-system-spec.md** — System specification: the base operating environment the services run on
→ **010-zs-ecosystem/012-engine-spec.md** — Engine specification: the shared Go libraries all services are built from
→ **010-zs-ecosystem/005-forms-spec.md** — Forms specification: notification templates follow its versioning conventions
→ **010-zs-ecosystem/019-fhir-hub-spec.md** — FHIR Integration Hub specification: external integration scope alongside the Integration service
→ **003-zs-platform/006-api-design.md** — API design: conventions for service endpoint contracts
→ **003-zs-platform/003-deployment-planes.md** — Deployment plane model: Plane 0 service consolidation and offline behaviour

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
