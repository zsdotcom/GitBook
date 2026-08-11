# 012-api-asyncapi-conventions.md
## AsyncAPI event specification conventions
### NATS subject naming, event envelope, DLQ, retention — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all event specifications in the ZarishSphere Platform

---

## Table of contents

1. [Event infrastructure](#1-event-infrastructure)
2. [Subject naming convention](#2-subject-naming-convention)
3. [Event envelope schema](#3-event-envelope-schema)
4. [AsyncAPI 3.1 schema file](#4-asyncapi-31-schema-file)
5. [Dead letter queue](#5-dead-letter-queue)
6. [Event retention](#6-event-retention)

---

## 1. Event infrastructure

### 1.1 Message broker

ZarishSphere uses NATS 2.14.4 JetStream for all asynchronous service-to-service communication. Every published event MUST have an AsyncAPI 3.1 schema definition.

### 1.2 When to use events vs REST

| Communication pattern | Use |
|---|---|
| Synchronous request-response | REST API (`/fhir/R5/`) |
| Asynchronous service-to-service | NATS event (`zs.{domain}.{resource}.{action}`) |
| Cross-plane replication | NATS event with JetStream mirroring |
| Audit log streaming | NATS event (`zs.audit.event`) |
| Alert propagation | NATS event (`zs.alert.*`) |

---

## 2. Subject naming convention

### 2.1 Subject pattern

```
zs.{domain}.{resource}.{action}
```

| Segment | Rules | Examples |
|---|---|---|
| `zs` | Fixed prefix | `zs` |
| `{domain}` | Lowercase, single word | `fhir`, `alert`, `platform`, `audit` |
| `{resource}` | PascalCase FHIR resource type or lowercase domain entity | `Patient`, `Observation`, `service` |
| `{action}` | Past tense verb | `created`, `updated`, `deleted`, `flagged`, `synced` |

### 2.2 Subject examples

```
zs.fhir.Patient.created
zs.fhir.Observation.updated
zs.fhir.Encounter.deleted
zs.alert.ewars.threshold_exceeded
zs.platform.service.health_check
zs.platform.deployment.status_changed
zs.audit.event.generated
```

### 2.3 Subject rules

| Rule | Description |
|---|---|
| All lowercase | Exception: FHIR resource types use PascalCase (`Patient`, not `patient`) |
| Dots as separators | NATS subject hierarchy uses `.` as delimiter |
| No underscores | Use dots or hyphens where needed |
| Wildcard support | Single level: `zs.fhir.*.created`; Multi-level: `zs.>` |
| Consistency | Same event for same resource/action across all services |

### 2.4 Wildcard patterns

| Pattern | Matches |
|---|---|
| `zs.fhir.*.created` | All FHIR resource create events |
| `zs.alert.>` | All alert events at any depth |
| `zs.>` | All ZarishSphere events |

---

## 3. Event envelope schema

### 3.1 Envelope structure

Every event published to NATS MUST use this envelope:

```json
{
  "$schema": "https://zarishsphere.com/schema/event/v1",
  "id": "0192fbad-xxxx-7xxx-xxxx-xxxxxxxxxxxx",
  "specVersion": "1.0",
  "type": "zs.fhir.Patient.created",
  "source": "zs-svc-patient",
  "subject": "Patient/0192fbad-1234-7abc-def0-123456789012",
  "time": "2026-03-24T10:00:00Z",
  "dataContentType": "application/fhir+json",
  "tenantId": "tenant-org-01",
  "data": {
    "resourceType": "Patient",
    "id": "0192fbad-1234-7abc-def0-123456789012",
    "...": "full FHIR R5 resource"
  }
}
```

### 3.2 Envelope field requirements

| Field | Required | Description |
|---|---|---|
| `$schema` | Yes | Event schema URI |
| `id` | Yes | UUID v7 — unique event identifier |
| `specVersion` | Yes | Event specification version (`1.0`) |
| `type` | Yes | Full subject path matching the NATS subject |
| `source` | Yes | Service name that published the event |
| `subject` | Yes | The resource reference (type/ID) |
| `time` | Yes | ISO 8601 UTC timestamp |
| `dataContentType` | Yes | Media type of the data payload |
| `tenantId` | Yes | Tenant context for the event |
| `data` | Yes | The event payload |

> **Constraint:** Every field in the envelope is mandatory. No optional fields. This ensures that event consumers always have complete context for processing.

---

## 4. AsyncAPI 3.1 schema file

### 4.1 Required file location

Each service that publishes or consumes events MUST include an `asyncapi.yaml` in its `docs/` directory:

```
{repo}/docs/asyncapi.yaml
```

### 4.2 Schema file structure

```yaml
asyncapi: 3.1.0
info:
  title: zs-svc-patient Events
  version: 1.0.0
  description: Events published by the patient registration service

channels:
  patientCreated:
    address: zs.fhir.Patient.created
    messages:
      patientCreated:
        $ref: '#/components/messages/PatientCreated'

operations:
  publishPatientCreated:
    action: send
    channel:
      $ref: '#/channels/patientCreated'

components:
  messages:
    PatientCreated:
      name: PatientCreated
      title: Patient Created
      summary: Event published when a new Patient resource is created
      payload:
        $ref: '#/components/schemas/FHIRPatientEvent'

  schemas:
    FHIRPatientEvent:
      type: object
      required: [id, type, source, time, tenantId, data]
      properties:
        id:
          type: string
          format: uuid
        type:
          type: string
          enum: [zs.fhir.Patient.created]
        source:
          type: string
        time:
          type: string
          format: date-time
        tenantId:
          type: string
        data:
          $ref: '#/components/schemas/FHIRPatient'

    FHIRPatient:
      type: object
      required: [resourceType, id]
      properties:
        resourceType:
          type: string
          enum: [Patient]
        id:
          type: string
          format: uuid
```

### 4.3 Schema file requirements

| Requirement | Description |
|---|---|
| AsyncAPI version | Must be `3.1.0` |
| Channel address | Must match the NATS subject pattern |
| Message name | PascalCase — must match the event type |
| Payload schema | Must reference the envelope structure |
| Action type | `send` for publishers, `receive` for consumers |

---

## 5. Dead letter queue

### 5.1 DLQ routing

Failed message processing routes to a dead letter queue subject:

```
zs.dlq.{original.subject}
```

| Failed event | DLQ subject |
|---|---|
| `zs.fhir.Patient.created` | `zs.dlq.zs.fhir.Patient.created` |
| `zs.alert.ewars.threshold_exceeded` | `zs.dlq.zs.alert.ewars.threshold_exceeded` |

### 5.2 DLQ message format

DLQ messages include the original envelope plus a `dlq` block:

```json
{
  "$schema": "https://zarishsphere.com/schema/event/v1",
  "id": "0192fbad-xxxx-7xxx-xxxx-xxxxxxxxxxxx",
  "type": "zs.fhir.Patient.created",
  "source": "zs-svc-patient",
  "subject": "Patient/0192fbad-...",
  "time": "2026-03-24T10:00:00Z",
  "data": { ... },
  "dlq": {
    "originalSubject": "zs.fhir.Patient.created",
    "failedAt": "2026-03-24T10:05:00Z",
    "reason": "terminology lookup timeout",
    "attemptCount": 5,
    "lastError": "connection timeout after 30s"
  }
}
```

### 5.3 DLQ processing rules

| Rule | Description |
|---|---|
| Max delivery attempts | 5 (configurable per subscription) |
| DLQ retention | 60 days |
| DLQ monitoring | Alert raised when any DLQ subject has > 100 messages |
| DLQ reprocessing | Manual — operator must resolve the issue and replay |

---

## 6. Event retention

### 6.1 JetStream retention configuration

| Subject pattern | Retention | Max messages | Storage type |
|---|---|---|---|
| `zs.fhir.>` | 30 days | 10,000,000 | File (persisted) |
| `zs.alert.>` | 90 days | 1,000,000 | File (persisted) |
| `zs.platform.>` | 7 days | 100,000 | Memory |
| `zs.audit.>` | 30 days | 10,000,000 | File (persisted) |
| `zs.dlq.>` | 60 days | 1,000,000 | File (persisted) |
| `zs.>` (catch-all) | 7 days | 500,000 | Memory |

### 6.2 Stream configuration

```bash
# Example NATS CLI configuration for FHIR events stream
nats stream add zs-fhir \
  --subjects "zs.fhir.>" \
  --storage file \
  --retention limits \
  --max-msgs 10000000 \
  --max-age 30d
```

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/011-api-openapi-conventions.md** — OpenAPI conventions for synchronous APIs
→ **005-zs-standards/008-fhir-audit-policy.md** — Audit event streaming on the `zs.audit.event` subject
→ **005-zs-standards/004-standards-to-platform-pipeline.md** — Enforcement pipeline consuming event-driven standards updates

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
