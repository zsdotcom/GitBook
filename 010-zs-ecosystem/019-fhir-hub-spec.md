# 019-fhir-hub-spec.md
## ZarishSphere FHIR Integration Hub specification
### Central FHIR R5 integration gateway

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Architecture](#2-architecture)
3. [Feature set](#3-feature-set)
4. [FHIR version policy](#4-fhir-version-policy)
5. [Security model](#5-security-model)
6. [Deployment](#6-deployment)
7. [Plane 0 operation](#7-plane-0-operation)
8. [Cross-references](#8-cross-references)

---

## 1. Purpose

The ZarishSphere FHIR Integration Hub (`zs-fhir-hub`) is a stateless FHIR R5 proxy and integration gateway. It connects external health systems — EHR platforms, national health information systems, laboratory information systems, and third-party applications — to the ZarishSphere Platform.

Key design principles:

- **Stateless proxy** — no data persistence, no local database
- **R5-only** — strictly FHIR R5, no R4 compatibility in the hub itself
- **Pluggable adapters** — adapter modules for external system protocols
- **Audit-everything** — every request and response is logged to the audit chain
- **Zero-cost** — built entirely with open-source Go tooling

## 2. Architecture

### 2.1 High-level flow

```
External System
      │
      ▼
┌─────────────────┐
│   zs-fhir-hub   │  ← Stateless Go proxy
│  (R5 gateway)   │
└────────┬────────┘
         │
         ├──→ zs-platform (internal FHIR R5 server)
         │
         └──→ Audit log (hash-chain)
              ↓
         NATS / PostgreSQL
```

### 2.2 Core components

| Component | Purpose | Technology |
|---|---|---|
| API proxy | Forward FHIR requests to internal server | Go net/http reverse proxy |
| Auth middleware | Authenticate and authorize external requests | Go + Keycloak integration |
| Transformation pipeline | Convert between external formats and FHIR R5 | Go plugin system |
| Audit logger | Record every request/response pair | Go + NATS JetStream |
| Rate limiter | Protect internal services from overload | Go middleware |
| Health endpoint | Liveness and readiness checks | Go HTTP handler |

### 2.3 Request flow

```
1. External system sends FHIR request → zs-fhir-hub
2. Hub authenticates request (JWT / mTLS / API key)
3. Hub validates FHIR version (R5 only — rejects non-R5)
4. Hub applies rate limiting
5. Hub transforms request if adapter is configured
6. Hub forwards to zs-platform internal FHIR server
7. Hub receives response
8. Hub transforms response if needed
9. Hub logs audit entry (request hash + response hash)
10. Hub returns response to external system
```

## 3. Feature set

### 3.1 FHIR API proxy

- Full FHIR R5 RESTful API passthrough
- Supports all FHIR interaction types: read, search, create, update, delete, patch
- FHIR `_search` and `_filter` parameters forwarded transparently
- Bulk FHIR export (`$export`) passthrough
- FHIR `$validate` operation support

### 3.2 Authentication and authorization

| Method | Use case | Implementation |
|---|---|---|
| JWT (SMART on FHIR) | Third-party app integration | Validate JWT against Keycloak |
| Mutual TLS | System-to-system integration | Client certificate validation |
| API key | Simple integrations | HMAC-signed API key lookup |
| Basic auth | Legacy system bridging | Keycloak token exchange |

### 3.3 Data transformation

The FHIR Hub includes a pluggable transformation pipeline for external system compatibility:

| Adapter | Source version | Target | Description |
|---|---|---|---|
| R4→R5 | FHIR R4 | FHIR R5 | Transforms R4 resources to R5 equivalents |
| DHIS2 | DHIS2 data value sets | FHIR R5 Observation | Aggregate indicator mapping |
| CSV | CSV file | FHIR R5 Bundle | Flat file import |
| HL7v2 | HL7 v2.x messages | FHIR R5 | Legacy system bridge |

Transformations are implemented as Go plugins loaded at startup. Each adapter is a separate module in the repository.

### 3.4 Audit logging

Every transaction through the hub generates an immutable audit log entry:

```
{
  "entry_id": "hash-chain-12345",
  "timestamp": "2026-06-11T12:00:00Z",
  "external_system": "mohes-bd-dghs",
  "request_method": "POST",
  "request_path": "/fhir/r5/Observation",
  "request_hash": "sha256-abc...",
  "response_status": 201,
  "response_hash": "sha256-def...",
  "transformation": "r4-to-r5",
  "previous_entry_hash": "sha256-789..."
}
```

Audit entries are:

- Written to NATS JetStream for real-time streaming
- Persisted to PostgreSQL for long-term storage
- Hash-chained for tamper detection

### 3.5 Rate limiting

| Limit type | Default | Configurable |
|---|---|---|
| Requests per second per client | 100 | Yes |
| Concurrent connections | 50 | Yes |
| Burst allowance | 20 | Yes |
| Daily quota | 100,000 | Yes |

Rate limits are enforced per authenticated client identity. Exceeding limits returns HTTP 429 with a `Retry-After` header.

## 4. FHIR version policy

> **Constraint:** The FHIR Hub processes FHIR R5 exclusively. Per ADR-005, FHIR R5 is the sole canonical FHIR version for the ZarishSphere ecosystem.

- The hub validates the `Accept` header and resource version on every request
- Non-R5 requests are rejected with HTTP 412 (Precondition Failed)
- An R4→R5 transformation adapter is available as a separate plugin for systems that cannot upgrade
- The adapter logs a warning and is intended as a temporary migration bridge

## 5. Security model

### 5.1 Network security

- The hub sits in a DMZ, with no direct access to the internal zs-platform network
- All communication is over TLS 1.3
- mTLS required for internal routing between hub and platform
- IP allowlisting for known external system endpoints

### 5.2 Credential management

- API keys are stored hashed (bcrypt) in the system configuration store
- JWT signing keys are managed by Keycloak
- mTLS certificates are issued by the Foundation CA
- Emergency key revocation is supported via the Console (per Constitution Law 4)

### 5.3 Request validation

- Maximum request body size: 10 MB (configurable)
- Request body schema validated against FHIR R5 resource definitions
- Path traversal attacks prevented by input sanitization
- SQL injection prevented — the hub performs no database queries

## 6. Deployment

### 6.1 Repository structure

```
zs-fhir-hub/
├── cmd/
│   └── zs-fhir-hub/
│       └── main.go
├── internal/
│   ├── proxy/
│   ├── auth/
│   ├── transform/
│   ├── audit/
│   ├── rate-limit/
│   └── config/
├── adapters/
│   ├── r4-to-r5/
│   ├── dhis2/
│   ├── csv/
│   └── hl7v2/
├── pkg/
│   ├── models/
│   └── middleware/
├── config/
│   ├── hub.yaml.example
│   └── adapters/
├── Dockerfile
├── docker-compose.yml
└── README.md
```

### 6.2 Deployment targets

| Plane | Deployment mode |
|---|---|
| Plane 0 | Not deployed (no external systems to integrate at Plane 0) |
| Plane 1 | Single binary, optional if no external integration needed |
| Plane 2 | Docker Compose service alongside platform |
| Plane 3 | Docker Compose or K3s with adapter plugins loaded |
| Plane 4 | Full deployment with all adapters, load-balanced |

## 7. Plane 0 operation

At Plane 0, the FHIR Hub is not required because there are no external systems to integrate. If a Plane 0 deployment later connects to a Plane 1+ network, the FHIR Hub runs on the connected plane while the Plane 0 device acts as a client.

## 8. Cross-references

→ **010-zs-ecosystem/008-api-spec.md** — API specification: the FHIR R5 REST endpoints the hub proxies to
→ **010-zs-ecosystem/009-services-spec.md** — Services specification: the Identity service behind hub authentication and the Audit service behind its hash-chain logging
→ **010-zs-ecosystem/013-system-spec.md** — System specification: the base operating environment for hub deployments
→ **005-zs-standards/005-fhir-r5-conventions.md** — FHIR R5 implementation conventions the hub validates against
→ **003-zs-platform/006-api-design.md** — API design: the authentication and contract conventions the hub's endpoints follow
→ **003-zs-platform/003-deployment-planes.md** — Deployment plane model: the per-plane deployment modes listed in the deployment targets table
→ **006-zs-infrastructure/007-security-architecture.md** — Security architecture: DMZ placement, TLS 1.3, and mTLS practices applied in the hub's security model

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
