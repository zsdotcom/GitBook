# 013-system-spec.md
## ZarishSphere System specification
### Base operating environment

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [System components](#2-system-components)
3. [Security model](#3-security-model)
4. [Plane 0 adaptation](#4-plane-0-adaptation)
5. [Cross-references](#5-cross-references)

---

## 1. Purpose

The ZarishSphere System is the base operating environment for the entire ecosystem. It provides the foundational infrastructure — identity, security, audit, configuration, monitoring, and backup — that every other component depends on.

## 2. System components

| Component | Purpose | Technology |
|---|---|---|
| Identity and access management | Authentication, authorization, roles | Keycloak + Go middleware |
| Encryption and key management | Data at rest, in transit, emergency key destruction | Vault + AES-256-GCM |
| Configuration management | System and component configuration | GitHub + viper |
| Health monitoring | Service health, alerts, uptime | Grafana + Prometheus |
| Backup and recovery | Automated backup, point-in-time recovery | pg_dump + MinIO |
| Audit infrastructure | Immutable audit log, hash-chain | PostgreSQL + hash-chain |

## 3. Security model

### 3.1 Identity tiers

| Tier | Population | Auth method |
|---|---|---|
| Community worker | All | PIN (4-digit) |
| Clinician | All | Username + password |
| Manager | All | MFA |
| Admin | Host community | MFA + SSO |
| Patient | All | ZS-UID only |

### 3.2 Encryption

- Data at rest: AES-256-GCM (PostgreSQL pgcrypto + Vault-managed keys)
- Data in transit: TLS 1.3 (Cloudflare + mutual TLS for service mesh)
- Emergency key destruction: separate key store, 30-second destruction via GUI

### 3.3 Audit

Every operation generates an immutable audit log entry. Entries are hash-chained: each entry includes the hash of the previous entry. Tampering with any historical entry breaks the chain and triggers an alert.

## 4. Plane 0 adaptation

At Plane 0, the System layer is minimal:
- Identity: basic local auth (no Keycloak)
- Encryption: local key file with emergency destruction
- Config: local YAML file
- Monitoring: basic health endpoint
- Backup: SQLite dump to removable media
- Audit: local SQLite, exported on sync

## 5. Cross-references

→ **010-zs-ecosystem/012-engine-spec.md** — Engine specification: the runtime that runs on top of the System layer
→ **010-zs-ecosystem/009-services-spec.md** — Services specification: the Identity and Audit services that consume System infrastructure
→ **009-zs-operations/006-sop-credential-succession.md** — SOP-006: credential documentation, rotation, and succession procedures for the System layer's secrets
→ **006-zs-infrastructure/007-security-architecture.md** — Security architecture: encryption, key management, and network security
→ **006-zs-infrastructure/008-security-policies.md** — Security policies and controls enforced by the System layer
→ **003-zs-platform/003-deployment-planes.md** — Deployment plane model: Plane 0 System adaptation
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 4: identity without surveillance

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
