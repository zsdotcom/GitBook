# 010-threat-models.md
## Threat models
### FHIR API, authentication, data, and mobile/offline threat analysis

**Document type:** Architecture
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Draft

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Methodology](#2-methodology)
3. [Threat model 1 — FHIR API layer](#3-threat-model-1--fhir-api-layer)
4. [Threat model 2 — Authentication layer](#4-threat-model-2--authentication-layer)
5. [Threat model 3 — Data layer](#5-threat-model-3--data-layer)
6. [Threat model 4 — Mobile/offline layer](#6-threat-model-4--mobileoffline-layer)
7. [Residual risks](#7-residual-risks)
8. [Cross-references](#8-cross-references)

---

## 1. Purpose

This document presents four consolidated threat models covering the primary attack surfaces of the ZarishSphere ecosystem. Each threat model uses the STRIDE methodology (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege) to identify, classify, and mitigate threats.

These threat models apply across all domains (health, education, governance, logistics, etc.) and all five deployment planes. Adaptations for Plane 0 (air-gapped) are noted where relevant.

---

## 2. Methodology

### 2.1 STRIDE framework

Each threat is classified using the STRIDE framework:

| Category | Definition | Example |
|----------|-----------|---------|
| **S**poofing | Impersonating a user, service, or system | Fake JWT, credential theft |
| **T**ampering | Modifying data or code without authorization | SQL injection, resource manipulation |
| **R**epudiation | Denying an action was performed | Claiming no access occurred |
| **I**nformation Disclosure | Exposing data to unauthorized parties | Cross-tenant data leak, PHI in logs |
| **D**enial of Service | Disrupting service availability | DDoS, resource exhaustion |
| **E**levation of Privilege | Gaining higher access than authorized | Scope escalation, role privilege abuse |

### 2.2 Threat notation

Each threat is documented with:

- **Threat description** — What the attack looks like
- **Attack vector** — How the attack is executed
- **Impact** — Effect on Confidentiality, Integrity, or Availability (CIA triad)
- **Mitigation** — Controls that prevent or detect the threat
- **Detection method** — How the threat is identified if mitigation fails

---

## 3. Threat model 1 — FHIR API layer

### 3.1 System description

The FHIR R5 API (`zs-core-fhir-engine`) is the primary interface for all data operations. It receives HTTPS requests from web applications, mobile apps, and integration services, authenticates them via SMART on FHIR, and reads/writes resources to the database.

### 3.2 Data flow diagram

```
[Client Application]
        │ HTTPS (TLS 1.3)
        ▼
[Cloudflare WAF + DDoS] ── blocks known attack IPs
        │
[API Gateway] ── rate limiting, CORS, auth headers
        │
[FHIR Engine] ── SMART scope validation, tenant scoping
        │
[PostgreSQL] ── Row-Level Security, encrypted fields
        │
[Message Broker] ── event publishing (async, post-commit)
```

### 3.3 Threat table

#### S — Spoofing

| Threat | Description | Attack vector | Impact | Mitigation | Detection |
|--------|-------------|--------------|--------|------------|-----------|
| JWT forgery | Attacker creates a fake JWT claiming authorized access | Crafting unsigned or weakly-signed tokens | Confidentiality, Integrity | RS256 signature validated against Keycloak JWKS endpoint; short expiry (1h) | Failed JWT validation logged as AuditEvent |
| Tenant spoofing | Attacker crafts a JWT claiming another tenant's ID | Modifying `tenant_id` claim in JWT | Confidentiality | `tenant_id` extracted from verified JWT claim; not from request body | Cross-tenant access attempt logged |
| Service impersonation | Attacker impersonates a microservice | Unauthenticated service-to-service request | Confidentiality, Integrity | mTLS enforced by service mesh; no plaintext service-to-service | Unauthenticated inter-service request blocked at network layer |

#### T — Tampering

| Threat | Description | Attack vector | Impact | Mitigation | Detection |
|--------|-------------|--------------|--------|------------|-----------|
| Resource tampering | Attacker modifies a resource after storage | Direct database access or API manipulation | Integrity | FHIR version history; database WAL; AuditEvent on every write | Version ID mismatch detected on subsequent reads |
| SQL injection | Attacker injects SQL via API parameters | Malicious input in search parameters | Integrity, Confidentiality | All queries use parameterized statements; never string concatenation | Database query monitoring; WAF SQL injection rules |
| Audit log tampering | Attacker deletes or modifies audit trail | Direct database access | Repudiation | Audit table: no DELETE/UPDATE permission for app user; append-only | Periodic audit log hash verification |

#### R — Repudiation

| Threat | Description | Attack vector | Impact | Mitigation | Detection |
|--------|-------------|--------------|--------|------------|-----------|
| Denying access | User claims they did not access a record | No technical countermeasure without logging | Repudiation | FHIR AuditEvent with JWT `sub`, timestamp, action, and resource ID | Audit trail review |
| Denying data change | User claims they did not modify data | No technical countermeasure without versioning | Repudiation | FHIR `meta.versionId` + `meta.lastUpdated` + `Provenance` resource | Version history audit |

#### I — Information Disclosure

| Threat | Description | Attack vector | Impact | Mitigation | Detection |
|--------|-------------|--------------|--------|------------|-----------|
| Cross-tenant data leak | Attacker accesses another tenant's data | Manipulating tenant identifier in request | Confidentiality | PostgreSQL RLS; `tenant_id` in all queries; API search filtered by tenant | RLS violation logged by PostgreSQL |
| Sensitive data in logs | Sensitive fields accidentally logged | Debug-level logging with verbose output | Confidentiality | Log configuration strips sensitive fields; no sensitive data in URLs | Log monitoring for sensitive patterns |
| Sensitive data in error responses | Error response includes data content | Triggering validation errors | Confidentiality | Custom error handler returns only error code, no resource content | Response body inspection |
| Backup exposure | Backup contains unencrypted sensitive data | Storage bucket misconfiguration | Confidentiality | Backup encrypted before upload; key in Vault | Backup encryption verification check |

#### D — Denial of Service

| Threat | Description | Attack vector | Impact | Mitigation | Detection |
|--------|-------------|--------------|--------|------------|-----------|
| Search explosion | Extremely broad search exhausts database | Wildcard or unbounded search query | Availability | Search result size capped at 200 (configurable); timeout at 30s | Query duration monitoring; slow query alert |
| DDoS | Flood of requests overwhelms API | Distributed attack from multiple sources | Availability | Cloudflare DDoS protection; API gateway rate limiting (100 req/min per IP) | Cloudflare DDoS alerts; gateway rate limit counters |
| Large bundle attack | Huge transaction bundle exhausts memory | POST with oversized FHIR Bundle | Availability | Bundle size limited to 1,000 resources and 10 MB | Request size validation at gateway |

#### E — Elevation of Privilege

| Threat | Description | Attack vector | Impact | Mitigation | Detection |
|--------|-------------|--------------|--------|------------|-----------|
| Scope escalation | User requests wider scope than permitted | Modifying scope claim in request | Confidentiality, Integrity | Keycloak enforces scope grants; FHIR engine validates on every request | Scope violation logged as AuditEvent |
| Role privilege escalation | Low-privilege user accesses restricted resources | Exploiting missing authorization checks | Confidentiality, Integrity | Keycloak RBAC with explicit resource type bindings | 403 Forbidden logged on authorization failure |

---

## 4. Threat model 2 — Authentication layer

### 4.1 System description

Authentication is handled by Keycloak 26.7.0 running as a platform deployment. Client applications use SMART on FHIR 2.1 (OAuth 2.1 (RFC 6749 + RFC 9700) + OIDC 1.0) to obtain JWTs. All API requests require a valid JWT.

### 4.2 Threat table

| Threat (STRIDE) | Description | Attack vector | Impact | Mitigation | Detection |
|----------------|-------------|--------------|--------|------------|-----------|
| **S** Credential theft | Username/password stolen | Phishing, credential stuffing, keylogging | Confidentiality | MFA enforced for all practitioner roles | Failed login attempts >5 triggers account lockout; alert sent |
| **S** Token theft | JWT stolen from browser storage | XSS, malware on client device | Confidentiality | Tokens stored in httpOnly cookies (not localStorage); short expiry (1h) | Token revocation list in Keycloak; abnormal token usage patterns |
| **T** Token replay | Stolen token reused after logout | Intercepting and reusing JWT | Confidentiality | Token revocation list; short expiry (1h access, 8h refresh) | Replay detection via `jti` claim tracking |
| **I** User enumeration | Attacker discovers valid usernames | Observing different error messages for valid vs invalid users | Confidentiality | Generic "invalid credentials" error regardless of validity | — |
| **D** Brute force | Automated password guessing | Distributed password attempts | Availability | 5 failed attempts → 15 minute lockout (Keycloak brute force protection) | Failed login rate monitoring |
| **E** Session hijacking | Attacker takes over active session | Session ID theft, fixation | Confidentiality, Integrity | Keycloak session binding to IP (configurable); session rotation on re-auth | Anomalous session behavior alerts |
| **E** Admin console exposure | Keycloak admin console accessible externally | Direct access to admin endpoints | Confidentiality, Integrity | Admin console restricted to internal network only (network policy) | Unauthorized admin console access attempt logged |

### 4.3 SMART on FHIR scope security

Scopes are granted by role, not by user request. The role table follows the canonical 7-role hierarchy defined in → **006-zs-infrastructure/008-security-policies.md** §2.2; the System row is a service account, not part of the human role hierarchy:

| Role | Granted scopes |
|------|---------------|
| Field Worker | `domain/Subject.read domain/Observation.write domain/QuestionnaireResponse.write` |
| Practitioner | `domain/*.read domain/Observation.write domain/Encounter.write` |
| Senior Practitioner | `domain/*.read domain/*.write` |
| Supervisor | `domain/*.read` (read-only) |
| Facility Admin | `user/*.read user/*.write` |
| Country Admin | `user/*.read user/*.write system/Organization.read` |
| Platform Owner | No default domain scope — access via GitHub org and infrastructure control |
| System (service account) | `system/*.read system/*.write` |

Users cannot request wider scopes than their role allows.

---

## 5. Threat model 3 — Data layer

### 5.1 System description

The data layer consists of PostgreSQL (primary database) with Valkey (cache). All sensitive fields are encrypted at the application layer before storage.

### 5.2 Threat table

| Threat (STRIDE) | Description | Attack vector | Impact | Mitigation | Detection |
|----------------|-------------|--------------|--------|------------|-----------|
| **S** Database credential theft | Database username/password compromised | Credential exposure in code, logs, or config files | Confidentiality, Integrity | Dynamic secrets via HashiCorp Vault — no static passwords; auto-rotated every 24h | Vault audit log on credential request |
| **T** Cross-tenant query | Application bug leaks data across tenants | Missing `WHERE tenant_id = ?` clause | Confidentiality | PostgreSQL Row-Level Security (RLS) enforces tenant isolation at database level | RLS violation logged; periodic query audit |
| **T** Direct database access | Attacker bypasses API and connects directly to database | Network-level attack, exposed port | Confidentiality, Integrity | Database bound to localhost only; no external port; access via Vault dynamic credentials | Unauthorized connection attempts logged |
| **I** Backup exposure | Unencrypted backup stolen from storage | Storage bucket misconfiguration | Confidentiality | Backup encrypted with AES-256 before upload; key in Vault | Backup encryption verification check |
| **I** Sensitive data in cache | Cache contains unencrypted subject data | Cache dump or misconfigured eviction | Confidentiality | Sensitive data in cache is encrypted; TTL 15 min; cache bound to internal network | Cache access audit |
| **D** Disk exhaustion | Audit log fills database disk | High volume of write operations | Availability | Automated partition pruning for audit events >7 years; alerting at 80% disk | Disk usage monitoring; alert at 80% |
| **E** Database user privilege escalation | Application user gains superuser | SQL injection, credential theft | Confidentiality, Integrity, Availability | App uses least-privilege database role (SELECT/INSERT/UPDATE on schema only; no DDL) | Grant/revoke audit logging |

### 5.3 PostgreSQL Row-Level Security

```sql
-- Multi-tenancy enforced at database level
ALTER TABLE fhir.resources ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON fhir.resources
    USING (tenant_id = current_setting('app.tenant_id', true));

-- App sets tenant before each query:
-- SET app.tenant_id = 'domain-cc-region-site';
```

Even if the application has a bug that omits a `WHERE tenant_id = ?` clause, PostgreSQL will still enforce the policy.

---

## 6. Threat model 4 — Mobile/offline layer

### 6.1 System description

Mobile applications and Plane 0 deployments operate with local storage (SQLite) and sync data when connectivity is available. This creates unique threat vectors around device theft, offline data at rest, and sync tampering.

### 6.2 Threat table

| Threat (STRIDE) | Description | Attack vector | Impact | Mitigation | Detection |
|----------------|-------------|--------------|--------|------------|-----------|
| **S** Device theft | Device containing subject data stolen | Physical theft of phone, tablet, or Raspberry Pi | Confidentiality | Biometric lock required; tokens in secure storage; SQLite encrypted with SQLCipher AES-256 | Remote wipe capability (when online) |
| **S** Session sharing | Field worker shares device with unauthorized person | Physical access to unlocked device | Confidentiality | Auto-lock after 5 minutes idle; separate user sessions | Session timeout audit |
| **T** Local database manipulation | Attacker edits SQLite file directly | Physical access to device storage | Integrity | SQLCipher encryption; integrity hash verified on sync | Sync integrity check failure triggers alert |
| **I** Sensitive data in crash logs | Crash report contains subject data | Application crash with sensitive data in stack trace | Confidentiality | Crash reporting configured to strip sensitive fields before upload | Periodic crash report audit |
| **I** Network snooping | Attacker intercepts sync traffic | Man-in-the-middle attack on public network | Confidentiality | Certificate pinning on sync connection; TLS 1.3 | Certificate pinning failure triggers disconnect |
| **D** Storage exhaustion | Large dataset fills device storage | Sync rule misconfiguration or excessive data | Availability | Sync rule limits local dataset to current facility/assignment | Storage usage monitoring; low storage warning |
| **E** Rooted/jailbroken device | App runs on compromised device | Rooted Android or jailbroken iOS | Confidentiality, Integrity | Root detection warning (non-blocking for health equity; blocking configurable) | Root detection alert |

### 6.3 Offline security properties

When fully offline (Plane 0 mode, no cloud):

- Subject data stored in encrypted SQLite (AES-256 via SQLCipher)
- Staff access controlled via local Keycloak instance
- Sync log maintains AuditEvent record of all offline access
- When connectivity restored: AuditEvents synced to cloud first, before data sync
- Integrity hashes verified on each sync operation

---

## 7. Residual risks

The following risks are accepted or require ongoing monitoring:

| Risk | Level | Status | Management |
|------|-------|--------|------------|
| Zero-day in FHIR library | Medium | Monitored | Trivy + osv-scanner; Renovate auto-updates dependencies |
| Identity provider vulnerability | Medium | Monitored | Renovate auto-updates Keycloak Helm chart |
| Insider threat (tenant owner) | Low | Accepted | 4 owners, 2 required for sensitive actions; all actions audited |
| Physical access to Plane 0 device | Medium | Partially mitigated | Disk encryption; physical security dependent on deployment site |
| Third-party dependency compromise | Medium | Monitored | Dependency pinning; SCA scanning; Renovate auto-updates |
| Supply chain attack on CI/CD | Low | Mitigated | GitHub Actions attestation; signed commits; branch protection |

### 7.1 Related threat model architecture

The threat models described here are mitigated by the controls defined in → **006-zs-infrastructure/007-security-architecture.md** (7 defense-in-depth layers), enforced by policies in → **006-zs-infrastructure/008-security-policies.md** (access control, encryption, vulnerability disclosure), and verified through compliance controls in → **006-zs-infrastructure/009-compliance-controls.md**. The FHIR API threat model relates specifically to the security model described in → **003-zs-platform/005-fhir-architecture.md**.

---

## 8. Cross-references

→ **006-zs-infrastructure/007-security-architecture.md** — Defense-in-depth controls mitigating these threats
→ **006-zs-infrastructure/008-security-policies.md** — Policies enforced against these threats
→ **006-zs-infrastructure/009-compliance-controls.md** — Compliance verification of threat mitigations
→ **003-zs-platform/005-fhir-architecture.md** — FHIR API security model
→ **008-zs-adrs/011-adr-privacy-by-architecture.md** — ADR-011: privacy by architecture
→ **003-zs-platform/003-deployment-planes.md** — Plane 0 threat adaptations

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
