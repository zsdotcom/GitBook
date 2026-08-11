# 008-security-policies.md
## Security policies
### Access control, encryption, vulnerability disclosure, and secret management

**Document type:** Architecture
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Draft

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Access control policy](#2-access-control-policy)
3. [Encryption policy](#3-encryption-policy)
4. [Vulnerability disclosure policy](#4-vulnerability-disclosure-policy)
5. [Secret management policy](#5-secret-management-policy)
6. [Access review schedule](#6-access-review-schedule)
7. [Policy compliance](#7-policy-compliance)
8. [Cross-references](#8-cross-references)

---

## 1. Purpose

This document defines the core security policies that govern all ZarishSphere deployments and operations. Each policy is designed to be implementation-agnostic, applying across all five deployment planes from air-gapped (Plane 0) to global SaaS (Plane 4). The policies implement the security architecture defined in → **006-zs-infrastructure/007-security-architecture.md** and support compliance requirements documented in → **006-zs-infrastructure/009-compliance-controls.md**.

These policies implement Constitution Law 4 (Identity without surveillance) and Law 5 (Open by default, secure by necessity).

---

## 2. Access control policy

### 2.1 Principle of least privilege

Every user, service, and system component is granted only the minimum access needed to perform its function. Access is denied by default and explicitly granted.

### 2.2 Role hierarchy

The ZarishSphere ecosystem defines a role hierarchy that applies uniformly across all domains (health, education, logistics, governance, etc.):

| Role | Description | Scope level |
|------|-------------|-------------|
| **Field Worker** | Frontline data collection | Patient/resident read, domain-specific write |
| **Practitioner** | Service delivery and clinical data entry | Domain resource read/write |
| **Senior Practitioner** | Full domain access, supervision | All domain resource read/write |
| **Supervisor** | Read-only access + reporting | Read-only all resources |
| **Facility Admin** | User management for facility | User management, facility-scoped |
| **Country Admin** | Full country tenant management | User management, system configuration |
| **Platform Owner** | Global ecosystem administration | GitHub org, infrastructure control |

### 2.3 Authentication

All access is authenticated through Keycloak 26.7.0 using:

- **OAuth 2.1 (RFC 6749 + RFC 9700) / OIDC 1.0** — Industry-standard authentication protocol
- **JWT (RS256)** — Signed tokens with verified claims
- **SMART App Launch 2.2.0** — Fine-grained scopes for FHIR resource access
- **MFA** — TOTP (Google Authenticator compatible), configurable per realm

### 2.4 Multi-tenancy access control

Every user is assigned to one or more tenants (organizations, programs, or facilities). JWT tokens include:

```json
{
  "sub": "user-uuid",
  "tenant_id": "domain-cc-region-site",
  "roles": ["practitioner"],
  "facility_ids": ["Location/site-hq"],
  "scope": "domain/*.read domain/Observation.write"
}
```

The FHIR engine enforces:

1. `tenant_id` from JWT matches `tenant_id` in all database queries
2. SMART scope checked against requested resource type and action
3. Facility-level filtering for roles below Country Admin

### 2.5 Service-to-service access

Go microservices communicate via:

1. **mTLS (Cilium)** — Authenticates service identity at the network layer
2. **Service account JWT** — Service-level SMART scopes
3. **Dedicated Keycloak clients** — Each service has minimum required scopes

No service has `system/*.write` unless explicitly required.

### 2.6 Emergency access (break-glass)

An emergency access role (`zs-emergency-access`) is available for exigent circumstances:

- Auto-expires after 4 hours
- Generates an immediate AuditEvent and notification to Country Admin
- All actions logged with timestamp and resource ID

---

## 3. Encryption policy

### 3.1 Encryption at rest

| Data layer | Encryption | Key management |
|-----------|-----------|----------------|
| PostgreSQL data files | AES-256 disk encryption | Infrastructure provider managed |
| Sensitive FHIR fields | AES-256-GCM (Vault Transit) | HashiCorp Vault managed |
| Cache entries with sensitive data | AES-256-GCM | Vault-managed key |
| Remote backups (R2) | AES-256-GCM before upload | Vault-managed key |
| Mobile device storage | SQLCipher AES-256 | Device secure enclave |
| Plane 0 (air-gapped) storage | SQLCipher AES-256 | Local secure storage |

### 3.2 Encryption in transit

| Connection | Protocol | Minimum version |
|-----------|----------|----------------|
| Client → API gateway | TLS | 1.3 |
| Service → Service | mTLS (Cilium) | 1.3 |
| Application → Database | TLS | 1.3 |
| Application → Message broker | TLS | 1.3 |
| Application → Identity provider | TLS | 1.3 |
| Mobile → FHIR API | TLS + certificate pinning | 1.3 |
| Backup → Object storage | TLS | 1.3 |

### 3.3 Field-level encryption

The following FHIR resource fields are encrypted at the application level before storage. This applies to any domain that handles personal or sensitive data:

```go
// zs-pkg-go-crypto implements field-level encryption
// Fields encrypted in JSONB blob before database storage:
EncryptedFields = []string{
    "Patient.name",
    "Patient.birthDate",
    "Patient.identifier",
    "Patient.address",
    "Patient.telecom",
    "RelatedPerson.name",
    "RelatedPerson.telecom",
}
```

Domain-specific measurements (e.g., health `Observation.value`, education `Assessment.score`) are NOT encrypted to enable analytics on de-identified data.

### 3.4 Key rotation schedule

| Key type | Rotation frequency | Method |
|----------|-------------------|--------|
| Vault Transit key (sensitive fields) | Annual | Vault key rotation (old key retained for decryption) |
| TLS certificates | Every 90 days | cert-manager automatic (Let's Encrypt) |
| Database dynamic credentials | Every 24 hours | Vault automatic |
| JWT signing key (Keycloak) | Every 90 days | Keycloak admin |

---

## 4. Vulnerability disclosure policy

### 4.1 Scope

This policy covers all repositories under `github.com/zarishsphere`, including platform components, infrastructure, frontend applications, mobile applications, and content repositories.

### 4.2 Reporting a vulnerability

**DO NOT open a public GitHub issue for security vulnerabilities.**

Instead:

1. **Email:** `security@zarishsphere.com`
2. **Subject:** `[SECURITY] Brief description`
3. **Encrypt (recommended):** PGP key published at `zarishsphere.com/.well-known/security.txt`

Include in the report:

- Description of the vulnerability
- Steps to reproduce
- Affected component and version
- Potential impact
- Suggested fix (optional)

### 4.3 Response timeline

| Milestone | Target |
|-----------|--------|
| Acknowledgement | Within 48 hours |
| Initial severity assessment | Within 5 business days |
| Regular status updates | Every 7 days |
| Fix deployed (critical/high) | Within 30 days |
| Fix deployed (medium/low) | Within 90 days |
| Public disclosure | After fix is deployed |

### 4.4 Severity classification

CVSS v3.1 scoring:

| Severity | Score | Examples |
|----------|-------|---------|
| Critical | 9.0–10.0 | Unauthenticated RCE, mass data export |
| High | 7.0–8.9 | Authenticated RCE, cross-tenant data access |
| Medium | 4.0–6.9 | Privilege escalation within tenant |
| Low | 0.1–3.9 | Information disclosure, minor misconfiguration |

### 4.5 Safe harbour

ZarishSphere commits to:

- Not pursuing legal action against researchers who follow this policy
- Not reporting researchers to law enforcement who follow this policy
- Treating researchers as partners, not adversaries
- Providing credit for discoveries (if desired)

Researchers are asked to:

- Give reasonable time for fixes before public disclosure
- Not access or modify other users' data
- Not perform denial-of-service attacks
- Not use vulnerabilities to access production data

### 4.6 Out-of-scope issues

- Social engineering attacks
- Physical security of server infrastructure
- Third-party dependency vulnerabilities (report to respective maintainers)
- Issues already tracked in the security backlog

---

## 5. Secret management policy

### 5.1 Rule zero

**No secrets, credentials, API keys, or passwords may ever be committed to any ZarishSphere repository.**

GitGuardian scans every commit. Violations result in immediate PR block and required secret rotation.

### 5.2 Secret categories and storage

| Secret type | Storage | Rotation |
|------------|---------|----------|
| Database credentials | HashiCorp Vault (dynamic) | Automatic, every 24 hours |
| Message broker authentication | HashiCorp Vault (static) | Every 90 days |
| Identity provider credentials | HashiCorp Vault | Every 90 days |
| Cloudflare API token | GitHub Org Secret | Every 90 days |
| External service API keys | GitHub Org Secret | Every 90 days |
| SMTP credentials | HashiCorp Vault | Every 90 days |
| Search service API key | HashiCorp Vault | Every 90 days |
| TLS private keys | cert-manager (auto) | Every 90 days (Let's Encrypt) |

### 5.3 Vault configuration

All services access secrets via Vault Agent sidecar or the Vault API:

```go
// Go services use vault-go SDK
// Secrets injected as environment variables at pod startup
// via External Secrets Operator → Kubernetes Secret → Pod env
```

**Vault path structure:**

```
secret/zarishsphere/
  prod/
    postgres/      ← Database credentials
    keycloak/      ← Identity provider admin
    nats/          ← Message broker auth
    typesense/     ← Search API key
  staging/
    ...
  dev/
    ...
```

### 5.4 Repository .env.example pattern

Every repository must have `.env.example` with placeholder values — never real values:

```bash
# .env.example — COPY to .env and fill in real values
# NEVER commit .env to git

DATABASE_URL=postgres://user:PASSWORD_HERE@localhost:5432/zarishsphere
NATS_URL=nats://localhost:4222
VALKEY_URL=redis://localhost:6379
KEYCLOAK_URL=http://localhost:8443
```

`.env` is in `.gitignore`. `.env.example` is committed.

### 5.5 Emergency rotation procedure

If a secret is accidentally committed:

1. **Immediately:** Revoke the secret at the source (GitHub → Settings → Tokens, or Vault)
2. **Within 1 hour:** Generate a new secret and update in Vault / GitHub Org Secrets
3. **Within 24 hours:** Force-push to remove from git history (coordinate with owners)
4. **Document:** Record in security incident log

---

## 6. Access review schedule

| Review | Frequency | Owner |
|--------|-----------|-------|
| User access rights (per facility/tenant) | Quarterly | Country Focal Point |
| Service account scopes | Semi-annual | Technical Lead |
| Platform owner accounts | Annual | Founder |
| Stale account deprovisioning | Monthly (automated) | Keycloak automated bot |
| Security policy review | Annual | All owners |
| Encryption key verification | Annual | Technical Lead |
| Secret rotation audit | Quarterly | Operations |

---

## 7. Policy compliance

### 7.1 Enforcement

These policies are enforced through:

- **GitHub branch protection** — PRs required for all policy changes; status checks mandatory
- **Automated scanning** — GitGuardian, Trivy, golangci-lint in CI/CD pipeline
- **Audit trail** — All policy changes recorded in git history
- **Access reviews** — Scheduled reviews with documented outcomes

For branch protection specifics, see → **006-zs-infrastructure/002-github-org-architecture.md**.

### 7.2 Policy violations

Policy violations are classified and addressed:

| Severity | Response |
|----------|----------|
| Critical (secret committed, data breach) | Immediate revocation + incident report within 24 hours |
| High (access control bypass) | Remediation within 48 hours + access review |
| Medium (outdated key, stale account) | Remediation within 7 days |
| Low (documentation gap) | Remediation within 30 days |

### 7.3 Plane 0 adaptation

All policies apply to Plane 0 with the following adaptations:

- Vault runs locally on the device rather than in a cluster
- GitGuardian scanning occurs before USB bundle creation
- Access reviews are documented on local git repositories and synced when connectivity returns
- Secret rotation uses local scheduling with sync on reconnect

### 7.4 Related policies

These security policies are enforced by the controls in → **006-zs-infrastructure/007-security-architecture.md** and verified through compliance audits documented in → **006-zs-infrastructure/009-compliance-controls.md**. Threat models for each attack surface are maintained in → **006-zs-infrastructure/010-threat-models.md**.

---

## 8. Cross-references

→ **006-zs-infrastructure/007-security-architecture.md** — Defense-in-depth architecture these policies enforce
→ **006-zs-infrastructure/009-compliance-controls.md** — Compliance controls verified against these policies
→ **006-zs-infrastructure/010-threat-models.md** — Threat models each policy mitigates
→ **006-zs-infrastructure/002-github-org-architecture.md** — Branch protection and enforcement layer
→ **008-zs-adrs/011-adr-privacy-by-architecture.md** — ADR-011: privacy by architecture
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 4 and Law 5
→ **003-zs-platform/005-fhir-architecture.md** — SMART App Launch scope enforcement in the FHIR engine

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
