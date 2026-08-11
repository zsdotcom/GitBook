# 009-fhir-r4-bridge-policy.md
## FHIR R4 bridge policy
### R4↔R5 bidirectional translation rules — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all R4↔R5 translation in the ZarishSphere Platform

---

## Table of contents

1. [Background](#1-background)
2. [When to use the R4 bridge](#2-when-to-use-the-r4-bridge)
3. [R4 → R5 translation (inbound)](#3-r4--r5-translation-inbound)
4. [R5 → R4 translation (outbound)](#4-r5--r4-translation-outbound)
5. [Lossy translation register](#5-lossy-translation-register)
6. [Testing requirements](#6-testing-requirements)
7. [Bridge versioning](#7-bridge-versioning)

---

## 1. Background

The ZarishSphere Platform stores all data as FHIR R5 internally. However, many partner systems — including existing health information systems, hospital EMRs, and national aggregate reporting platforms — use FHIR R4 (4.0.1).

The R4 bridge (`zs-core-fhir-r4-bridge`) provides bidirectional R4↔R5 translation to enable data exchange with these partner systems.

> **Constraint:** FHIR R4 is a bridge standard only. All internal storage, service-to-service communication, and newly developed integrations MUST use FHIR R5. No new integration may be built directly against R4 without explicit architectural review.

---

## 2. When to use the R4 bridge

### 2.1 Approved use cases

| Use case | Direction | Example |
|---|---|---|
| Receiving data from R4 systems | R4 → R5 | OpenMRS sends an R4 Patient resource |
| Sending data to R4-only systems | R5 → R4 | Legacy DHIS2 aggregate reporting endpoint |
| Partner integration | Both | National EMR interoperability gateway |
| Legacy data migration | R4 → R5 | Bulk import of historical patient records |

### 2.2 Prohibited use cases

| Use case | Reason |
|---|---|
| Internal service-to-service communication | Always use R5 directly |
| Data storage | Storage is always R5 |
| FHIR subscriptions | Always use R5 topics |
| SMART App Launch app launches | Use the R5 SMART App Launch flow |
| New API endpoint design | Use R5 native endpoints |

---

## 3. R4 → R5 translation (inbound)

### 3.1 Resource mapping table

When receiving R4 resources from a partner system:

| R4 resource | R5 equivalent | Key transformation |
|---|---|---|
| `Patient` | `Patient` | R5 adds `Patient.link` changes; `Patient.genderIdentity` added |
| `Observation` | `Observation` | R5 component handling differs; `valueQuantity` precision updated |
| `MedicationRequest` | `MedicationRequest` | R5 renames: fields use `medication` directly |
| `Condition` | `Condition` | R5 `clinicalStatus` binding updated with new values |
| `Encounter` | `Encounter` | R5 `Encounter.classHistory` changes |
| `Appointment` | `Appointment` | R5 significant restructure — use bridge with CAUTION |
| `Questionnaire` | `Questionnaire` | R5 adds `Questionnaire.subjectType` enforcement |
| `QuestionnaireResponse` | `QuestionnaireResponse` | Largely compatible with minor structural changes |

### 3.2 Inbound translation process

1. Receive R4 resource on bridge endpoint
2. Validate the R4 resource against R4 schema
3. Apply field mapping rules from the translation table
4. Add `meta.tag` with `system: zs-r4-translated, code: true`
5. Generate an AuditEvent with action code `E` and description "R4→R5 translation"
6. Store the translated R5 resource in the standards store
7. Return the translated R5 resource with HTTP 200

---

## 4. R5 → R4 translation (outbound)

### 4.1 Outbound translation rules

When sending R5 resources to an R4-only system:

1. Translate R5-only features to the closest R4 equivalent
2. Add `meta.tag` with `{ "system": "https://zarishsphere.com/tags/translation", "code": "r5-to-r4" }`
3. Log the translation in an AuditEvent with direction "R5→R4"
4. If translation is lossy, document the dropped fields in the AuditEvent `entity.detail`
5. Return the translated R4 resource

### 4.2 Translation quality levels

| Quality level | Meaning | Audit annotation |
|---|---|---|
| `direct` | All fields map cleanly | No annotation needed |
| `lossy` | One or more R5 fields have no R4 equivalent | List dropped fields |
| `approximate` | R5 field mapped to nearest R4 equivalent | Note the approximation |

---

## 5. Lossy translation register

### 5.1 Known lossy translations

These R5 features have no R4 equivalent. When translating R5→R4, these fields are dropped or approximated:

| R5 field | R4 equivalent | Lossiness |
|---|---|---|
| `Subscription.topic` | None | **Lost** — R4 has no topic-based subscriptions |
| `Appointment.previousAppointment` | None | **Lost** — no R4 concept |
| `Patient.genderIdentity` | None (extension) | **Downgraded** — mapped to a custom extension |
| `Encounter.classHistory` | Partial | **Approximate** — R4 has less granular class history |
| `Observation.hasMember` | `Observation.hasMember` | Compatible |
| `BiologicallyDerivedProduct` | New R5 resource | **Lost** — no R4 equivalent resource exists |

### 5.2 Audit requirements for lossy translations

Every lossy translation MUST:

1. Be recorded in the audit trail with a complete list of dropped fields
2. Include a warning in the API response header: `X-Translation-Lossy: true`
3. Be identifiable via the `meta.tag` with code `r5-to-r4-lossy`

---

## 6. Testing requirements

### 6.1 Mandatory test coverage

The R4 bridge MUST have integration tests covering:

| Test scenario | Description |
|---|---|
| Round-trip translation (R5 → R4 → R5) | Verify no data loss for non-lossy fields |
| Per-resource mapping | All listed resource types in §3.1 |
| Error handling | Malformed R4 input produces appropriate OperationOutcome |
| Audit event generation | Every translation generates a valid AuditEvent |
| Lossy translation logging | Dropped fields are correctly documented |
| Concurrent translation | Multiple simultaneous translations produce correct isolation |

### 6.2 CI validation

```bash
# Run bridge integration tests
go test ./bridge/... -tags=integration

# Expected output:
# --- PASS: TestRoundTripPatient
# --- PASS: TestRoundTripObservation
# --- PASS: TestLossyTranslationLogging
# --- PASS: TestAuditEventGeneration
```

---

## 7. Bridge versioning

### 7.1 Version format

The R4 bridge version is independent of the platform version:

```
{major}.{r4-spec-version}.{r5-spec-version}
```

| Component | Meaning | Example |
|---|---|---|
| `major` | Bridge architecture version | `1` |
| `r4-spec-version` | FHIR R4 spec year supported | `4.0` (for 4.0.1) |
| `r5-spec-version` | FHIR R5 spec year supported | `5.0` (for 5.0.0) |

**Example:** `1.4.0-5.0` (bridge v1, R4 4.0.x, R5 5.0.x)

### 7.2 Version compatibility

| Bridge version | R4 compatible | R5 compatible |
|---|---|---|
| `1.4.0-5.0` | 4.0.1 | 5.0.0 |
| `1.4.0-5.0.1` | 4.0.1 | 5.0.1+ |

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **003-zs-platform/005-fhir-architecture.md** — FHIR architecture and service mesh
→ **008-zs-adrs/005-adr-fhir-r5-over-r4.md** — Decision to use FHIR R5 as canonical version

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
