# 009-compliance-controls.md
## Compliance controls
### HIPAA, GDPR, data residency, and compliance matrix

**Document type:** Architecture
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Draft

---

## Table of contents

1. [Purpose](#1-purpose)
2. [HIPAA technical safeguards](#2-hipaa-technical-safeguards)
3. [GDPR compliance](#3-gdpr-compliance)
4. [Data residency policy](#4-data-residency-policy)
5. [Compliance control matrix](#5-compliance-control-matrix)
6. [Audit and review schedule](#6-audit-and-review-schedule)
7. [Plane 0 compliance](#7-plane-0-compliance)
8. [Cross-references](#8-cross-references)

---

## 1. Purpose

This document defines the compliance controls that the ZarishSphere ecosystem implements to meet regulatory requirements across multiple jurisdictions. While the ecosystem was originally architected for health domains, its compliance architecture is domain-agnostic and applies equally to education, finance, governance, and all 40 domains defined in the ZarishSphere taxonomy.

The controls documented here implement Constitution Law 4 (Identity without surveillance) and satisfy the technical safeguard requirements of HIPAA, the data subject rights of GDPR, and data sovereignty mandates across multiple countries.

---

## 2. HIPAA technical safeguards

### 2.1 Overview

The ZarishSphere platform implements all required and addressable HIPAA Technical Safeguards (45 CFR § 164.312) to protect sensitive data at rest and in transit. These controls serve as the baseline for all domain deployments, providing a security floor that exceeds most regulatory requirements.

### 2.2 § 164.312(a) — Access control

> *Implement technical policies and procedures for electronic information systems that maintain sensitive information to allow access only to those persons or software programs that have been granted access rights.*

| HIPAA sub-rule | ZarishSphere control | Implementation |
|---------------|---------------------|----------------|
| Unique user identification (R) | Keycloak 26.7.0 user accounts | Every user has a unique `sub` claim in JWT |
| Emergency access procedure (R) | Break-glass role in Keycloak | `zs-emergency-access` role, auto-expires 4h |
| Automatic logoff (A) | Keycloak session timeout | Default: 8 hours inactive → session revoked |
| Encryption and decryption (A) | AES-256 via HashiCorp Vault | All sensitive fields encrypted at field level |

**R = Required | A = Addressable**

SMART on FHIR 2.1 scope enforcement applies to all FHIR API access:

```
domain/*.read   — read any resource within scope
domain/*.write  — write any resource within scope
user/*.read     — read all resources (practitioner)
system/*.read   — read all resources (system service)
```

Scope violations return HTTP 403 Forbidden and generate an AuditEvent.

### 2.3 § 164.312(b) — Audit controls

> *Implement hardware, software, and/or procedural mechanisms that record and examine activity in information systems that contain or use sensitive information.*

Every data access in the ZarishSphere platform generates a FHIR R5 `AuditEvent` resource stored in the `audit.events` table:

```sql
INSERT INTO audit.events (
    event_type,      -- 'rest'
    resource_type,   -- 'Patient', 'Observation', etc.
    resource_id,     -- FHIR resource ID
    user_id,         -- Keycloak user ID from JWT
    action,          -- 'create', 'read', 'update', 'delete'
    tenant_id,       -- Organization/facility identifier
    recorded_at,     -- UTC timestamp
    resource         -- Full FHIR AuditEvent JSON
);
```

**Audit log retention:** 7 years minimum
**Tamper protection:** Audit table has no UPDATE or DELETE permissions for application users

### 2.4 § 164.312(c) — Integrity

> *Implement policies and procedures to protect sensitive information from improper alteration or destruction.*

| Mechanism | Implementation |
|-----------|---------------|
| FHIR version history | Every resource update creates a history record |
| Database WAL | Write-ahead log for crash recovery |
| Backup verification | Monthly restore test to verify backup integrity |
| Soft delete only | Records are never physically deleted (`deleted_at` timestamp) |
| Digital signatures | FHIR `Provenance` resources link edits to user JWT |

### 2.5 § 164.312(d) — Authentication

> *Implement procedures to verify that a person or entity seeking access to sensitive information is the one claimed.*

| Mechanism | Implementation |
|-----------|---------------|
| Identity provider | Keycloak 26.7.0 (OIDC 1.0, MFA support) |
| Token standard | JWT (RS256 signed by Keycloak) |
| Token validation | Every request validates JWT signature against Keycloak JWKS endpoint |
| MFA | TOTP (Google Authenticator compatible), configurable per realm |
| Session management | Refresh tokens with rolling expiry |

### 2.6 § 164.312(e) — Transmission security

> *Implement technical security measures to guard against unauthorized access to sensitive information that is being transmitted.*

| Mechanism | Implementation |
|-----------|---------------|
| TLS in transit | TLS 1.3 minimum enforced by API gateway |
| Certificate management | cert-manager auto-renews Let's Encrypt certificates |
| mTLS between services | Service mesh enforces mTLS between all microservices |
| API gateway | All external traffic through gateway with WAF headers |
| No sensitive data in URLs | Search uses POST + request body, never GET query params for sensitive data |

---

## 3. GDPR compliance

The ZarishSphere platform is designed to comply with GDPR principles as a global baseline for data protection, regardless of primary deployment geography.

### 3.1 Lawful basis for processing

The platform processes data under:

- **Art. 6(1)(e):** Task in the public interest (government programs, essential services)
- **Art. 9(2)(h):** Provision of services (healthcare, education, social services)

### 3.2 Data subject rights

#### Article 15 — Right of access

**Implementation:** The FHIR `$everything` operation returns all data linked to a subject:

```
GET /fhir/R5/Patient/{id}/$everything
Authorization: Bearer {subject-token}
```

Response: FHIR Bundle containing all resources linked to the subject.

**Process:** Subject requests data → Authorized staff runs `$everything` → Exports to standard format → Delivered within 30 days.

#### Article 16 — Right to rectification

**Implementation:** FHIR `PUT` operation with `meta.lastUpdated` and `Provenance` resource documenting who corrected what:

```
PUT /fhir/R5/Patient/{id}
Body: corrected Patient resource
```

All corrections are logged in `audit.events` and a `Provenance` resource is created.

#### Article 17 — Right to erasure (right to be forgotten)

**Implementation:** The platform uses pseudonymisation, not erasure, for operational records that may be legally required to be retained.

**Process:**
1. Subject requests erasure
2. Authorized lead verifies legal retention requirements
3. If erasure permitted: `zs-pkg-go-crypto` pseudonymises all identifying fields
4. Original identifiers replaced with irreversible hash
5. Subject cannot be re-identified from remaining data

FHIR resources are never physically deleted — only soft-deleted (`deleted_at` timestamp). Physical erasure of identified data is handled by the pseudonymisation procedure above.

#### Article 20 — Right to data portability

**Implementation:** FHIR Bulk Export (`$export`) provides all subject data in NDJSON format, importable by any FHIR R4/R5 system:

```
POST /fhir/R5/Patient/$export
Parameters: _type=Patient,Observation,Condition,Medication
```

#### Article 33 — Data breach notification

Breaches affecting personal data must be reported to the supervisory authority within 72 hours. See the vulnerability disclosure process in → **006-zs-infrastructure/008-security-policies.md**.

### 3.3 Data protection by design

| Principle | ZarishSphere implementation |
|-----------|----------------------------|
| Data minimisation | Forms only collect fields with operational purpose and FHIR mapping |
| Purpose limitation | `tenant_id` scoping prevents cross-tenant data access |
| Storage limitation | Retention policies configurable per country deployment |
| Integrity/confidentiality | AES-256 field encryption, TLS 1.3 in transit |
| Pseudonymisation | `zs-pkg-go-crypto` for right-to-erasure workflow |

### 3.4 Data processing agreements

Deploying entities are the **data controllers**. The ZarishSphere core team acts as **data processor** only during onboarding support. After Plane 2 (district server) or higher, deploying entities are fully autonomous data controllers with complete data sovereignty.

---

## 4. Data residency policy

### 4.1 Policy statement

**Data generated in a jurisdiction stays in that jurisdiction.**

The ZarishSphere platform is designed for digital sovereignty. Subject records, operational data, and program data must reside within the jurisdiction of the deploying entity unless an explicit cross-jurisdiction data sharing agreement exists.

### 4.2 Jurisdictional requirements

| Jurisdiction | Data must reside in | Enforcement mechanism |
|-------------|--------------------|-----------------------|
| Bangladesh | Bangladesh or equivalent jurisdiction | Deployment to local or nearest-region infrastructure |
| India | India (DPDPA 2023 requirement) | Deployment to India region |
| Myanmar | Myanmar or ASEAN region | Country choice |
| Pakistan | Pakistan (PDPA 2010) | Country-hosted preferred |
| Thailand | Thailand (PDPA 2022 requirement) | Deployment to Thailand region |
| European Union | EU member state (GDPR) | Deployment to EU region |
| United States | US jurisdiction (HIPAA) | Deployment to US region |

### 4.3 Technical implementation

#### Database residency

Each deployment's infrastructure configuration specifies the region or on-premise datacenter where the database is deployed:

```hcl
# Infrastructure configuration example
variable "region" {
  default = "me-jeddah-1"  # Nearest cloud region
}
```

#### API data residency

The platform's `tenant_id` scoping ensures data is always queried with `WHERE tenant_id = {jurisdiction_code}`, preventing cross-jurisdiction data access even if multiple jurisdictions share infrastructure.

#### Backup residency

Backups are stored with jurisdictional restriction:

- Each jurisdiction configures their own storage bucket
- Jurisdictional restriction settings prevent cross-border data storage
- Backups are encrypted before upload

#### Plane 0 residency

On Plane 0 (air-gapped), data physically resides on the device. No data leaves the device unless explicitly synced. This provides the strongest possible data residency guarantee.

### 4.4 Prohibited data flows

The following data flows are prohibited:

1. Replication of production data outside the jurisdiction without explicit legal agreement
2. Processing of identifiable data by ZarishSphere core team without a signed Data Processing Agreement
3. Aggregation of identifiable data across jurisdictions (anonymised/aggregate data is permitted)
4. Transfer of data to any entity not covered by the deploying entity's agreement

### 4.5 Audit of data residency

Deploying entities must verify data residency annually:

1. Confirm database is running in the correct region: `SELECT inet_server_addr();`
2. Confirm storage bucket jurisdiction in the provider dashboard
3. Document confirmation in the deployment status record

---

## 5. Compliance control matrix

The following matrix maps technical controls to relevant regulatory requirements:

| Control | HIPAA §164.312 | GDPR Article | Implementation |
|---------|---------------|-------------|----------------|
| Unique user identity | (a)(2)(i) | 5(1)(f) | Keycloak 26.7.0 UUID per user |
| Emergency access | (a)(2)(ii) | — | Break-glass role, 4h expiry |
| Automatic logoff | (a)(2)(iii) | — | 8h session timeout |
| Encryption at rest | (a)(2)(iv) | 32 | AES-256 via Vault Transit |
| Audit logging | (b) | 5(2), 30 | FHIR AuditEvent on all data access |
| Data integrity | (c)(1) | 5(1)(f) | FHIR version history, soft delete |
| Authentication | (d) | 5(1)(f) | JWT + SMART on FHIR 2.1 |
| TLS in transit | (e)(2)(ii) | 32(1) | TLS 1.3 enforced by API gateway |
| mTLS service-to-service | (e)(2)(ii) | 32(1) | Service mesh mTLS |
| Backup integrity | (c)(2)(ii) | 32 | Database PITR to encrypted storage |
| Access control | (a) | 25 | RBAC + PostgreSQL RLS |
| Breach notification | — | 33 | Incident response policy (72h) |
| Right of access | — | 15 | FHIR `$everything` operation |
| Right to portability | — | 20 | FHIR `$export` (NDJSON) |
| Right to erasure | — | 17 | Pseudonymisation workflow |
| Data minimisation | — | 5(1)(c) | FHIR-mapped fields only |

---

## 6. Audit and review schedule

| Activity | Frequency | Owner |
|----------|-----------|-------|
| Access log review | Monthly | Operations |
| User access rights review | Quarterly | Jurisdiction focal point |
| Encryption key rotation | Annual | Technical lead |
| Full compliance control review | Annual | All owners |
| Backup restore test | Monthly | Operations |
| Data residency verification | Annual | Jurisdiction focal point |
| Incident response drill | Semi-annual | Operations |

---

## 7. Plane 0 compliance

On Plane 0 (air-gapped), compliance controls are maintained as follows:

| Cloud-dependent control | Plane 0 equivalent |
|------------------------|-------------------|
| Keycloak (central auth) | Local Keycloak instance on device |
| Vault (secrets management) | Local Vault instance on device |
| TLS certificates | Self-signed certificates for local TLS |
| Audit log aggregation | Local zerolog files with rotation |
| Cloudflare WAF | Physical air gap (no network attack surface) |
| External backup storage | Encrypted local storage + USB bundle |

Compliance documentation is stored in the local git repository and synced when connectivity is restored. All access controls, encryption requirements, and audit logging policies remain identical in function.

### 7.1 Related architecture

These compliance controls implement Constitution Law 4 (Identity without surveillance) as specified in → **008-zs-adrs/011-adr-privacy-by-architecture.md** and are enforced by the controls defined in → **006-zs-infrastructure/007-security-architecture.md** and → **006-zs-infrastructure/008-security-policies.md**.

---

## 8. Cross-references

→ **006-zs-infrastructure/007-security-architecture.md** — Security architecture implementing these controls
→ **006-zs-infrastructure/008-security-policies.md** — Security policies and vulnerability disclosure
→ **006-zs-infrastructure/010-threat-models.md** — Threat models for data-layer and API risks
→ **008-zs-adrs/011-adr-privacy-by-architecture.md** — ADR-011: privacy by architecture
→ **003-zs-platform/005-fhir-architecture.md** — FHIR operations used for data subject rights
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 4 (Identity without surveillance)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
