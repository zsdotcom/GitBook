# 007-security-architecture.md
## Defense-in-depth security architecture
### Seven security layers, secrets management, encryption, and audit

**Document type:** Architecture
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Draft

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Design principles](#2-design-principles)
3. [Defense-in-depth layers](#3-defense-in-depth-layers)
4. [Secrets management](#4-secrets-management)
5. [Encryption strategy](#5-encryption-strategy)
6. [Audit controls](#6-audit-controls)
7. [Plane 0 security model](#7-plane-0-security-model)
8. [Security operations](#8-security-operations)
9. [Cross-references](#9-cross-references)

---

## 1. Purpose

This document defines the defense-in-depth security architecture for the entire ZarishSphere ecosystem. Security is enforced at every layer — from the Cloudflare edge to individual database fields — so that no single vulnerability can compromise the system. The architecture is designed to function identically across all five deployment planes, with Plane 0 (air-gapped) providing the maximum security posture through physical isolation.

The ZarishSphere Constitution Law 4 (Identity without surveillance) mandates privacy by architecture. This document details the technical controls that make that mandate enforceable.

---

## 2. Design principles

### 2.1 Defense in depth

No single security control is trusted. Every layer provides redundancy — if Cloudflare WAF is bypassed, Traefik enforces auth. If Traefik is bypassed, PostgreSQL RLS prevents cross-tenant access. If the database is compromised, field-level encryption protects sensitive data.

### 2.2 Least privilege

Every user, service, and system component is granted only the minimum access needed to perform its function. Access is denied by default and explicitly granted.

### 2.3 No secrets in git

No credential, API key, token, or password may ever be committed to any ZarishSphere repository. GitGuardian scans every commit. Violations result in immediate PR block and required secret rotation.

### 2.4 Privacy by architecture

Security controls are structural, not policy-based. Individual surveillance is technically impossible, not merely contractually prohibited. This aligns with Constitution Law 4, as implemented by ADR-011.

> **Constraint:** Every security control must have a documented Plane 0 equivalent. Cloud-dependent controls must degrade gracefully rather than fail open.

### 2.5 Audit everything

Every access to sensitive data creates an immutable AuditEvent record. Audit logs cannot be modified or deleted by application users.

---

## 3. Defense-in-depth layers

The security architecture is organised as seven sequential layers, each protecting against the failure of the layer before it.

```
Layer 1: Cloudflare Edge
  DDoS protection, WAF, rate limiting, bot management
        │
Layer 2: Traefik API Gateway
  TLS 1.3 termination, auth header validation, rate limits
        │
Layer 3: SMART on FHIR Scopes
  Fine-grained access control per resource type and action
        │
Layer 4: FHIR Resource RBAC
  Role-based access per resource type, ID, and compartment
        │
Layer 5: PostgreSQL Row-Level Security
  Tenant isolation enforced at the database level
        │
Layer 6: Field-Level Encryption
  AES-256-GCM encryption for sensitive fields before storage
        │
Layer 7: AuditEvent Logging
  Every access logged — immutable, time-partitioned, 7-year retention
```

### 3.1 Layer 1 — Cloudflare Edge

| Control | Implementation | Free tier |
|---------|---------------|-----------|
| DDoS protection | Layer 3/4/7 mitigation | Included |
| WAF | OWASP core rules, custom rules | Basic rules |
| Rate limiting | 100 req/min per IP (configurable) | 10 rules |
| Bot management | Bot Fight Mode | Included |
| Geo-blocking | Country-level allow/deny | Included |

Configuration details are in → **006-zs-infrastructure/003-cloudflare-architecture.md** — Cloudflare WAF and DDoS configuration.

### 3.2 Layer 2 — Traefik API Gateway

Traefik 3.7.10 terminates all TLS connections and enforces:

- **TLS 1.3 minimum** — No older protocol versions permitted
- **Auth header forwarding** — Validates JWT presence before routing to services
- **Rate limiting** — Per-service rate limits, distinct from Cloudflare edge limits
- **CORS enforcement** — Strict origin validation per service
- **Request size limits** — Bundles capped at 10 MB

### 3.3 Layer 3 — SMART on FHIR Scopes

Every FHIR API request carries a JWT with SMART App Launch 2.2.0 scopes. The FHIR engine validates scopes on every request:

| Scope pattern | Meaning |
|--------------|---------|
| `patient/*.read` | Read any resource for accessible patients |
| `patient/*.write` | Write any resource for accessible patients |
| `patient/Observation.read` | Read only Observation resources |
| `user/*.read` | Read all resources (clinician scope) |
| `system/*.write` | Full system access (service accounts only) |

Scope violations return HTTP 403 and generate an AuditEvent.

### 3.4 Layer 4 — FHIR Resource RBAC

Beyond SMART scopes, the FHIR engine enforces:

- **Resource-type permissions** — Which roles can access which FHIR resource types
- **Compartment-based access** — Access limited to resources linked to the user's assigned patients
- **Operation-level permissions** — Read, write, delete, and history controlled separately per role

Role hierarchy (canonical 7-role model, defined in → **006-zs-infrastructure/008-security-policies.md** §2.2):

| Role | Access level |
|------|-------------|
| Field Worker | `patient/Patient.read`, `patient/Observation.write` |
| Practitioner | `patient/*.read`, `patient/Observation.write`, `patient/Encounter.write` |
| Senior Practitioner | `patient/*.read`, `patient/*.write` |
| Supervisor | `patient/*.read` (read-only) |
| Facility Admin | `user/*.read`, `user/*.write` (facility-scoped user management) |
| Country Admin | `user/*.read`, `user/*.write`, `system/Organization.read` (country-scoped) |
| Platform Owner | Global ecosystem administration — GitHub org and infrastructure control; no default clinical resource scope |

### 3.5 Layer 5 — PostgreSQL Row-Level Security

Even if the application layer has a bug that omits tenant filtering, PostgreSQL RLS enforces tenant isolation:

```sql
ALTER TABLE fhir.resources ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON fhir.resources
    USING (tenant_id = current_setting('app.tenant_id', true));
```

The application sets `app.tenant_id` from the verified JWT before each query. PostgreSQL rejects any query that does not match the tenant context.

### 3.6 Layer 6 — Field-Level Encryption

Sensitive fields are encrypted at the application layer before being written to PostgreSQL JSONB. Decryption occurs only when authorized:

```go
// Fields encrypted before storage
EncryptedFields = []string{
    "Patient.name",
    "Patient.birthDate",
    "Patient.identifier",
    "Patient.address",
    "Patient.telecom",
    "RelatedPerson.name",
    "RelatedPerson.telecom",
}

// Clinical measurements (Observation.value) are NOT encrypted
// to enable analytics on de-identified data
```

Encryption uses AES-256-GCM via HashiCorp Vault Transit. Each tenant can have a distinct encryption key. Decryption keys never leave Vault.

### 3.7 Layer 7 — AuditEvent Logging

Every access to sensitive data generates a FHIR R5 AuditEvent resource:

| Field | Source |
|-------|--------|
| `eventType` | `rest` |
| `resourceType` | FHIR resource type accessed |
| `resourceId` | Specific FHIR resource ID |
| `userId` | Keycloak `sub` claim from JWT |
| `action` | `create`, `read`, `update`, `delete` |
| `tenantId` | Organization/facility identifier |
| `recordedAt` | UTC timestamp |
| `resource` | Full FHIR AuditEvent JSON |

**Retention:** 7 years minimum
**Tamper protection:** Audit table has no UPDATE or DELETE permissions for application users

---

## 4. Secrets management

### 4.1 HashiCorp Vault

All secrets are managed by HashiCorp Vault (self-hosted, open-source OSS edition). Vault runs in the `zs-auth` namespace and provides:

- **Dynamic database credentials** — PostgreSQL credentials auto-rotated every 24 hours
- **Static secrets** — API keys, NATS auth, Typesense keys, SMTP credentials (rotated every 90 days)
- **TLS certificates** — Managed by cert-manager with Let's Encrypt (auto-renewed every 90 days)
- **Encryption keys** — Field-level encryption keys managed via Vault Transit

### 4.2 Vault path structure

```
secret/zarishsphere/
  prod/
    postgres/      ← Dynamic DB credentials
    keycloak/      ← Keycloak admin password
    nats/          ← NATS authentication
    typesense/     ← Typesense API key
  staging/
    ...
  dev/
    ...
```

### 4.3 Secret delivery

Services receive secrets through one of two mechanisms:

1. **Vault Agent sidecar** — Injects secrets as environment variables at pod startup
2. **External Secrets Operator** → Kubernetes Secret → Pod environment variables

### 4.4 No secrets in git

- Every repository has `.env.example` with placeholder values
- `.env` is in every repository's `.gitignore`
- GitGuardian scans every commit for credential leakage
- Secrets accidentally committed trigger immediate revocation and rotation

### 4.5 Emergency secret rotation

If a secret is compromised:

1. **Immediately:** Revoke the secret at the source (GitHub → Settings → Tokens, or Vault)
2. **Within 1 hour:** Generate a new secret and update in Vault or GitHub Org Secrets
3. **Within 24 hours:** Force-push to remove from git history (coordinate with owners)
4. **Document:** Record in security incident log

---

## 5. Encryption strategy

### 5.1 Encryption at rest

| Data | Encryption | Key management |
|------|-----------|----------------|
| PostgreSQL data files | AES-256 disk encryption | Cloud provider managed |
| Sensitive FHIR fields | AES-256-GCM (Vault Transit) | Vault-managed key |
| Valkey cache entries with PHI | AES-256-GCM | Vault-managed key |
| Cloudflare R2 backups | AES-256-GCM before upload | Vault-managed key |
| SQLite (mobile/offline) | SQLCipher AES-256 | Device secure enclave |
| SQLite (Raspberry Pi / Plane 0) | SQLCipher AES-256 | Local secure storage |

### 5.2 Encryption in transit

| Connection | Protocol | Version |
|-----------|----------|---------|
| Client → Traefik | TLS | 1.3 |
| Service → Service | mTLS (Cilium) | 1.3 |
| Application → PostgreSQL | TLS | 1.3 |
| Application → NATS | TLS | 1.3 |
| Application → Keycloak | TLS | 1.3 |
| Mobile → FHIR API | TLS + certificate pinning | 1.3 |
| Backup → R2 | TLS | 1.3 |

### 5.3 Key rotation schedule

| Key | Rotation frequency | Method |
|-----|-------------------|--------|
| Vault Transit key (sensitive fields) | Annual | Vault key rotation (old key still decrypts old data) |
| TLS certificates | Every 90 days | cert-manager automatic (Let's Encrypt) |
| PostgreSQL dynamic credentials | Every 24 hours | Vault automatic |
| JWT signing key (Keycloak) | Every 90 days | Keycloak admin |

---

## 6. Audit controls

### 6.1 Audit scope

Every access to the following operations generates an AuditEvent:

- All FHIR resource CRUD operations
- Authentication events (login, logout, failed login)
- Scope validation failures
- Emergency access (break-glass) activations
- Configuration changes to WAF, DNS, or routing rules
- Secrets rotation events

### 6.2 Audit log architecture

```
Service (HTTP request)
  │  zerolog → structured JSON
  ▼
Loki (log aggregation)
  │  7-year retention, time-partitioned
  ▼
Grafana (log exploration and alerting)
```

### 6.3 Audit review schedule

| Review | Frequency | Owner |
|--------|-----------|-------|
| Access log review | Monthly | Operations |
| User access rights review | Quarterly | Country focal point |
| Encryption key rotation | Annual | Technical lead |
| Full security architecture review | Annual | All owners |

---

## 7. Plane 0 security model

Plane 0 (air-gapped) deployments operate with zero cloud dependencies. The security architecture adapts as follows:

| Cloud control | Plane 0 equivalent |
|---------------|-------------------|
| Cloudflare DDoS/WAF | Physical air gap — no network attack surface |
| Cloudflare TLS | Self-signed certificates for local TLS |
| HashiCorp Vault | Local Vault instance on device |
| Keycloak auth | Local Keycloak instance on device |
| GitGuardian scanning | Pre-deployment audit before USB bundle creation |
| AuditEvent to Loki | Local zerolog files + rotation |

> **Plane 0 security advantage:** Physical air gap eliminates the most common attack vectors. The primary threats shift from network-based to physical access attacks. All local storage is encrypted with SQLCipher AES-256.

For full Plane 0 deployment specifications, see → **003-zs-platform/003-deployment-planes.md**.

---

## 8. Security operations

### 8.1 Vulnerability management

- **SCA scanning:** Trivy + osv-scanner on every CI build
- **Dependency updates:** Renovate bot auto-updates dependencies; security patches merged within 7 days
- **SAST:** golangci-lint with security rules
- **Secret scanning:** GitGuardian on every push

### 8.2 Incident response

Security incidents follow the process defined in → **006-zs-infrastructure/008-security-policies.md** — vulnerability disclosure and incident response.

### 8.3 Security testing

| Test type | Frequency | Tool |
|-----------|-----------|------|
| Dependency scanning | Every CI build | Trivy, osv-scanner |
| Static analysis | Every CI build | golangci-lint |
| Secret scanning | Every push | GitGuardian |
| Penetration testing | Annual | External researcher (coordinated) |

### 8.4 Related architecture

The security architecture directly integrates with the FHIR security model → **003-zs-platform/005-fhir-architecture.md** and relies on Cloudflare WAF/DDoS protection → **006-zs-infrastructure/003-cloudflare-architecture.md** for edge-level threat mitigation. For the governance framework that enforces these controls, see → **006-zs-infrastructure/002-github-org-architecture.md**.

---

## 9. Cross-references

→ **006-zs-infrastructure/008-security-policies.md** — Security policies implementing these controls
→ **006-zs-infrastructure/009-compliance-controls.md** — Compliance verification of these controls
→ **006-zs-infrastructure/010-threat-models.md** — Threat models mitigated by these seven layers
→ **006-zs-infrastructure/003-cloudflare-architecture.md** — Cloudflare WAF and DDoS edge controls
→ **003-zs-platform/005-fhir-architecture.md** — FHIR security model and SMART scopes
→ **003-zs-platform/003-deployment-planes.md** — Plane 0 deployment specifications
→ **008-zs-adrs/011-adr-privacy-by-architecture.md** — ADR-011: privacy by architecture
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 4 (Identity without surveillance)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
