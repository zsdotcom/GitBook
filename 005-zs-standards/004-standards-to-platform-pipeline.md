# 004-standards-to-platform-pipeline.md
## ZARISH-STANDARDS — Standards-to-Platform Pipeline
### Real-time Enforcement · G2A Engine Integration · Offline Sync · Audit · V1

**Document type:** Specification — Canonical
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative. Defines the deployment and enforcement pipeline for all ZS-* standards.

---

## Table of Contents

1. [Pipeline Overview](#1-pipeline-overview)
2. [Standards Distribution to the G2A Engine](#2-standards-distribution-to-the-g2a-engine)
3. [Real-time Compliance Checking](#3-real-time-compliance-checking)
4. [Sync Strategy](#4-sync-strategy)
5. [Caching and Offline Operation (Plane 0)](#5-caching-and-offline-operation-plane-0)
6. [Error Scenarios](#6-error-scenarios)
7. [Monitoring and Audit Logging](#7-monitoring-and-audit-logging)

---

## 1. Pipeline Overview

The standards-to-platform pipeline is the final stage of the ZARISH-STANDARDS lifecycle. It takes validated ZS-* standard definitions from the transformation layer and deploys them to the ZarishSphere Platform's G2A Engine, where they become active compliance rules for forms, workflows, and data validation.

### 1.1 High-level data flow

```
ZS-* Record (transformed + validated)
    |
    v
[Standards Registry — zs-zarish-standards/releases]
    |
    v
[G2A Engine Standards Store]
    |
    +---> [Runtime Compliance Checker]
    |         |
    |         v
    |     [Form/Workflow Execution] ---> [Validation Result]
    |
    +---> [Standards Cache (per deployment plane)]
    |         |
    |         v
    |     [Offline Operations (Plane 0)]
    |
    +---> [Audit Log]
              |
              v
         [Compliance Dashboard]
```

### 1.2 Pipeline stages

| Stage | Component | Description |
|---|---|---|
| Publish | Standards Registry Publisher | Publishes ZS-* records from the transformation pipeline as a release artifact |
| Distribute | Distribution Agent | Pushes standards updates to the G2A Engine's standards store and plane caches |
| Load | G2A Engine Startup | Loads active standards from the standards store into memory at engine boot |
| Validate | Runtime Compliance Checker | Validates form and workflow data against loaded standards during execution |
| Report | Audit Logger | Records every compliance check result for monitoring and audit |

### 1.3 Pipeline roles

| Role | Responsibility |
|---|---|
| Standards Registry | Stores the canonical ZS-* records; publishes releases |
| Distribution Agent | Pushes updates to planes; manages sync scheduling |
| G2A Engine | Loads standards; performs runtime validation |
| Deployer | Configures plane-specific sync policies and cache behaviour |
| Auditor | Reviews compliance logs; escalates enforcement failures |

---

## 2. Standards Distribution to the G2A Engine

### 2.1 Release artifact format

The ZARISH-STANDARDS registry publishes releases as a standardised bundle:

| File | Format | Contents |
|---|---|---|
| `standards-registry.json` | JSON | Full array of all active ZS-* records |
| `standards-registry.yaml` | YAML | YAML equivalent for human review |
| `standards-registry-slim.json` | JSON | Active-only records — excludes RETIRED and PENDING |
| `g2a-rulesets.json` | JSON | G2A-specific rulesets extracted from ZS-* records |
| `schema-versions.json` | JSON | Schema version reference for each ZS-* record |
| `MANIFEST.json` | JSON | Release metadata — version, date, record count, checksums |

### 2.2 Release versioning

Standards releases follow the version format: `zs-standards-v[YYYYMMDD].[NN]`

| Example | Meaning |
|---|---|
| `zs-standards-v20260610.01` | First release on June 10, 2026 |
| `zs-standards-v20260610.02` | Second release on the same day (hotfix) |

### 2.3 Distribution channels

| Channel | Mechanism | Latency |
|---|---|---|
| Plane 3-4 (cloud) | Direct push via HTTPS — G2A Engine pulls on startup + subscribes to webhook | Seconds |
| Plane 2 (district server) | Scheduled pull from registry (configurable interval, default 1 hour) | Minutes |
| Plane 1 (RPi edge) | Relay from Plane 2 or direct scheduled pull when connected | Minutes to hours |
| Plane 0 (air-gapped) | Manual side-load via USB or local network transfer | As available |

### 2.4 G2A Engine loading behaviour

On receiving a standards release, the G2A Engine:

1. **Validates checksums** — Confirms the release bundle integrity using SHA-256 hashes from `MANIFEST.json`
2. **Diffs against current store** — Computes additions, modifications, deprecated, and retired records
3. **Applies changes** — Updates the in-memory standards store atomically
4. **Logs the update** — Records the release version, timestamp, and change summary to the audit log
5. **Notifies active sessions** — Sends a signal to all active form/workflow sessions that standards have changed

> **Constraint:** The G2A Engine must never apply a partial standards update. If any validation in step 1 or 2 fails, the engine must reject the entire release and continue with the current standards store. Partial updates would create inconsistent validation state.

---

## 3. Real-time Compliance Checking

### 3.1 How compliance checking works

When a user fills a form or executes a workflow, the G2A Engine's Runtime Compliance Checker validates each data field against the active ZS-* standards. The compliance check operates at three levels:

**Level 1 — Field-level validation**

For each data field in a form or workflow, the compliance checker:
1. Identifies which ZS-* standards apply to that field (determined by domain, field type, and workflow context)
2. Loads the `enforcement_rules` from each applicable standard
3. Validates the field value against the standard's rules
4. Returns a validation result (pass, warning, or error)

**Level 2 — Record-level validation**

For a complete record (e.g., a patient registration, a clinical encounter, a supply order):
1. Verifies that all mandatory standard-referenced fields are present
2. Checks cross-field consistency rules defined by applicable standards
3. Validates that the record conforms to any structural standard (e.g., FHIR resource profile)

**Level 3 — Workflow-level validation**

For a multi-step workflow (e.g., an NCD consultation following WHO PEN):
1. Validates that the workflow sequence matches the standard's procedural requirements
2. Checks that required data is collected at each step
3. Verifies that the workflow output satisfies the standard's reporting requirements

### 3.2 Compliance levels in practice

| Standard | Level applied | Example |
|---|---|---|
| ICD-11 [TYPE-A] | Level 1 — field | Diagnosis code must be a valid ICD-11 code |
| HL7 FHIR R5 [TYPE-B] | Level 2 — record | Patient resource must conform to FHIR Patient profile |
| WHO PEN [TYPE-C] | Level 3 — workflow | NCD workflow must include risk assessment, diagnosis, and treatment plan steps |
| ISO 27001 [TYPE-C] | All levels | Field encryption, record audit trail, workflow access control |

### 3.3 Compliance response

| `validation_type` | Block form save? | Display in UI? | Log level |
|---|---|---|---|
| `strict` | Yes | Error message on field | ERROR |
| `warning` | No | Warning message on field | WARN |
| `informational` | No | Info icon on field | INFO |

### 3.4 Override mechanism

Qualified users (deployers with `standards_override` permission) can override a compliance failure for individual records. Overrides are:
- Time-limited (configurable, default 24 hours)
- Logged to the audit trail with user ID, reason, and timestamp
- Reported in the compliance dashboard for review

---

## 4. Sync Strategy

### 4.1 Push-based sync (default for connected planes)

For Planes 2, 3, and 4, the pipeline uses a push-based model:

| Event | Action |
|---|---|
| New standards release published | Distribution Agent pushes the release to all connected Plane endpoints |
| Plane reconnects after disconnect | Distribution Agent sends the current release on reconnection |
| Active session in progress | G2A Engine applies updates at session boundaries (not mid-form) |

### 4.2 Pull-based sync (fallback for intermittent connectivity)

Planes 1 and 2 in intermittent-connectivity environments use a pull-based model:

| Configuration | Default | Description |
|---|---|---|
| `sync_interval` | 3600 seconds (1 hour) | How often the plane checks for new releases |
| `sync_on_boot` | `true` | Pull current release at G2A Engine startup |
| `sync_on_reconnect` | `true` | Pull current release when network reconnects |
| `sync_timeout` | 30 seconds | Maximum time to wait for sync completion |

### 4.3 Sync scheduling for Plane 0

Plane 0 (air-gapped) has no network sync. Administrators receive standards bundles via:

1. **USB transfer** — Standards release bundle on USB drive from the connected distribution point
2. **Local network** — LAN-based transfer from a Plane 1 or 2 relay within the same facility
3. **Pre-loaded deployment** — Standards bundle baked into the deployment image at provisioning time

Plane 0 sync is manual and must be confirmed by an operator. The G2A Engine logs the operator ID and timestamp for audit purposes.

### 4.4 Delta sync

To minimise bandwidth on constrained connections, the pipeline supports delta sync:

- Only records that changed since the last acknowledged release are transmitted
- The delta is computed by comparing the plane's current `manifest_version` against the current registry version
- Full sync is always available as a fallback

> **Constraint:** Delta sync must never skip records. If the plane cannot verify its current manifest version (e.g., corrupted local state), it must fall back to a full sync.

---

## 5. Caching and Offline Operation (Plane 0)

### 5.1 Standards cache architecture

Every deployment plane maintains a local standards cache:

| Cache component | Contents | Persistence |
|---|---|---|
| Active standards store | All ZS-* records with ACTIVE or BETA lifecycle status | Persistent (SQLite or local DB) |
| Ruleset cache | G2A rulesets extracted from active standards | Persistent |
| Reference cache | Supporting reference data (code systems, value sets) | Persistent |
| Version manifest | Current release version and checksums | Persistent |
| Sync state | Last sync timestamp, last manifest version acknowledged | Persistent |

### 5.2 Offline validation capability

On Plane 0, the G2A Engine operates entirely from the local cache. All compliance checking happens locally without requiring network access:

- Field-level validation uses cached code systems and value sets
- Record-level validation uses cached FHIR profiles and structural rules
- Workflow-level validation uses cached procedural rules from TYPE-C standards

### 5.3 Cache lifecycle

| Event | Cache behaviour |
|---|---|
| New standards release received | Cache updated atomically |
| Plane reboot | Cache loaded from persistent storage |
| Cache corruption detected | Cache rebuilt from most recent available release bundle |
| Standards record deprecated | Record marked in cache but retained for legacy reads |
| Standards record retired | Record removed from cache after grace period (default 30 days) |

### 5.4 Cache health checks

The G2A Engine runs internal cache health checks at startup and every 24 hours:

| Check | Action on failure |
|---|---|
| Checksum verification | Trigger re-sync from current release |
| Record count consistency | Trigger re-sync from current release |
| Reference integrity (all dependency references resolve) | Flag for operator review |
| Schema version alignment | Trigger re-sync from current release |

---

## 6. Error Scenarios

### 6.1 Missing standard reference

**Scenario:** A form field references a standard that does not exist in the loaded standards store.

**Resolution:**
- The G2A Engine logs the missing standard reference with the field ID and workflow context
- The form is rendered with the field in `error` state
- The user sees an error message: *"Validation standard [ZS-* ID] is not available. Contact your system administrator."*
- The compliance dashboard reports the missing standard
- An automatic notification is sent to the deployer's configured alert channel

**Recovery:**
1. Deployer verifies whether the standard should exist
2. If missing from transformation: trigger re-transform of the ZARISH-INDEX entry
3. If missing from pipeline: re-push the standards release
4. If incorrectly referenced: update the form definition to reference the correct standard

### 6.2 Outdated standard reference

**Scenario:** A form field references a standard revision that has been deprecated or retired.

**Resolution:**
- The G2A Engine checks the reference against the current standards store
- If the referenced standard is DEPRECATED: the engine checks for a `replaced_by` field; if found, automatically remaps the reference to the successor
- If no successor exists: the field is shown with a deprecation warning
- If the referenced standard is RETIRED: the field behaves as a missing reference

**Recovery:**
1. G2A Engine logs the auto-remapping event to the audit trail
2. Form designer receives a notification to update the field reference
3. The deprecated reference is replaced in the next form deployment

### 6.3 Standards store corruption

**Scenario:** The local standards store in the G2A Engine becomes corrupted (detected by checksum mismatch or integrity check failure).

**Resolution:**
- G2A Engine enters "safe mode" — continues operation with last known good cache
- A provisioning alert is sent to the deployer
- All compliance checking is performed against the safe-mode cache
- New standards releases are queued but not applied until the store is repaired

**Recovery:**
1. Deployer initiates a full re-sync from the standards registry
2. G2A Engine rebuilds the standards store from the full sync
3. Cache integrity check passes
4. G2A Engine exits safe mode and applies any queued releases

### 6.4 Sync failure for a plane

**Scenario:** A plane cannot reach the standards registry to pull a release.

**Resolution:**
- The plane retries with exponential backoff: 30s, 2m, 5m, 15m, 30m, 60m (stops at 60m intervals)
- After 5 consecutive failures, a connectivity alert is raised
- The plane continues operation with its current standards cache
- Compliance checking continues without interruption

**Recovery:**
1. Link restored between plane and registry
2. Next scheduled sync succeeds or manual sync triggered
3. Delta sync catches up any missed releases

### 6.5 Curator override of validation failure

**Scenario:** A form submission fails compliance validation but the deployer determines the data is valid and must be accepted.

**Resolution:**
- deployer with `standards_override` permission activates the override
- The record is saved with the override timestamp and user ID
- The override is logged to the audit trail
- The compliance dashboard records the event as an override, not a pass

**Limits:**
- Overrides are always time-limited (configurable, default 24 hours)
- Override limits per deployer per day are configurable (default 50)
- Override events trigger a notification to the standards curator

---

## 7. Monitoring and Audit Logging

### 7.1 Audit log events

Every significant pipeline event is recorded in the audit log:

| Event | Data recorded |
|---|---|
| Standards release published | Release version, record count, checksum, timestamp |
| Standards release distributed | Target plane, release version, distribution success/fail |
| Standards store updated (G2A) | Previous release version, new release version, change summary |
| Compliance check performed | Record ID, field ID, standard reference, result (pass/warning/error/override) |
| Compliance override | Record ID, user ID, reason, expiry timestamp |
| Missing standard reference | Field ID, workflow ID, standard ID |
| Auto-remap (deprecated → successor) | Original reference, new reference, workflow ID |
| Safe mode entered/exited | Triggering event, timestamp |
| Sync success/failure | Target plane, release version, duration, result |
| Cache integrity check | Result (pass/fail), affected records if fail |

### 7.2 Audit log format

```json
{
  "event_id": "zs-audit-20260610-00001",
  "event_type": "compliance_check",
  "timestamp": "2026-06-10T14:30:00Z",
  "plane_id": "plane-2-district-01",
  "module_id": "ZS-MOD-health",
  "workflow_id": "wf-ncd-intake-v1",
  "record_id": "ZS-NCD-2025-001234",
  "field_id": "field-diagnosis-code",
  "standard_ref": "ZS-HL-ICD11-2026",
  "compliance_level": 1,
  "validation_type": "strict",
  "result": "error",
  "error_code": "E-VAL-001",
  "message": "Diagnosis code 'XYZ123' is not a valid ICD-11 code",
  "user_id": "user-dr-fatima",
  "override": false
}
```

### 7.3 Compliance dashboard

The ZarishSphere Console provides a compliance dashboard with these views:

| View | Data shown |
|---|---|
| Standards summary | Total active standards, by type, by domain, by governance scope |
| Compliance rate | Percentage of compliance checks passing across all active forms |
| Enforcement breakdown | Count of strict vs warning vs informational enforcement rules applied |
| Override activity | Override events by user, by standard, by domain over the selected period |
| Error trends | Most common validation errors, by standard, by field, by module |
| Sync status | Last sync time and release version for each deployment plane |
| Safe mode alerts | Active safe mode incidents and their duration |

### 7.4 Alert thresholds

| Alert | Threshold | Channel |
|---|---|---|
| Compliance rate drops below 95% | Rolling 1-hour window | Console notification + email |
| Override rate exceeds 5% of total checks | Rolling 24-hour window | Console notification + email |
| Missing standard reference detected | Any single occurrence | Console warning + weekly digest |
| Sync failure for any plane | 3 consecutive failures | Console alert + email |
| Any safe mode event | Any occurrence | Critical alert — console + email + SMS |
| Any cache corruption event | Any occurrence | Critical alert — console + email + SMS |

### 7.5 Log retention

| Environment | Retention period | Storage |
|---|---|---|
| Planes 3-4 (cloud) | 7 years | Centralised audit log store with annual archiving |
| Plane 2 (district server) | 2 years | Local database with monthly export to cloud |
| Plane 1 (RPi edge) | 1 year | Local database with quarterly export when connected |
| Plane 0 (air-gapped) | 2 years | Local database — export on operator schedule |

### 7.6 Integration with ZARISH-INDEX

The pipeline maintains a bi-directional reference connection with ZARISH-INDEX.

- Standards registry references ZARISH-INDEX as the upstream source for every ZS-* record
- When a ZARISH-INDEX entry changes status (e.g., Active → Superseded), the pipeline applies the corresponding lifecycle change to the ZS-* record
- The audit log records all upstream-driven changes with the ZARISH-INDEX version reference

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Strategic direction and three-type framework
→ **005-zs-standards/002-transformation-model.md** — Upstream transformation pipeline
→ **005-zs-standards/003-standards-schema.md** — ZS-* record schemas and validation
→ **003-zs-platform/004-g2a-engine.md** — G2A Engine runtime specification
→ **004-zs-index/005-zarish-index-to-platform.md** — ZARISH-INDEX integration

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
