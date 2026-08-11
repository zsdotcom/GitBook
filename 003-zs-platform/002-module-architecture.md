# 002-module-architecture.md
## Module architecture
### Sovereignty rules, interfaces, lifecycle

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Module sovereignty principle](#1-module-sovereignty-principle)
2. [Module structure](#2-module-structure)
3. [Module lifecycle](#3-module-lifecycle)
4. [Module communication](#4-module-communication)
5. [Module registry](#5-module-registry)
6. [Cross-references](#6-cross-references)

---

## 1. Module sovereignty principle

Every domain module is independently deployable. No module requires another module to function (Law 7). Modules communicate via standard APIs, not shared databases. Modules can be combined but never coupled.

| Rule | Meaning |
|---|---|
| Independent deployment | A module can be deployed alone without any other module |
| Independent data | Each module has its own database schema, not shared |
| Independent lifecycle | Each module can be updated, rolled back, or removed independently |
| Standard interfaces | Modules communicate through APIs, not direct database access |
| No cross-module coupling | A module must never import or depend on another module's internals |

## 2. Module structure

Each module has the same internal structure:

```
zs-module-[domain]/
├── api/                  ← API definitions (OpenAPI 3.1)
├── cmd/                  ← Entry points
├── internal/
│   ├── handler/          ← HTTP handlers
│   ├── service/          ← Business logic
│   ├── repository/       ← Data access
│   └── model/            ← Domain models
├── migrations/           ← Database migrations
├── forms/                ← Form definitions
├── workflows/            ← Workflow definitions
├── tests/                ← Module tests
├── Dockerfile
├── go.mod
└── README.md
```

### 2.1 Required artefacts

| Artefact | Format | Required |
|---|---|---|
| API specification | OpenAPI 3.1 YAML | Yes |
| Database migrations | SQL (golang-migrate) | Yes |
| Form definitions | FHIR Questionnaire JSON | Yes |
| Module manifest | YAML | Yes |
| Dockerfile | Docker | Yes |
| README | Markdown | Yes |

### 2.2 Module manifest

Every module must have a `manifest.yaml` at its root:

```yaml
id: ZS-MOD-health
name: Health Module
version: 1.0.0
domain: health
description: Domain module for health services
dependencies: []  # Modules must not require other modules
forms:
  - patient-registration
  - ncd-assessment
workflows:
  - referral
  - prescription
apis:
  - rest: https://api.zarishsphere.com/fhir/R5
```

## 3. Module lifecycle

| State | Description |
|---|---|
| Draft | Module specification exists in zs-docs, no implementation |
| Development | Code being written in private branch |
| Alpha | Deployable for testing, not for production |
| Stable | Production-ready, validated, documented |
| Deprecated | Replacement available, migration period active |
| Archived | No longer available, removed from Marketplace |

## 4. Module communication

### 4.1 API-based communication

Modules communicate exclusively through the API gateway. No module directly accesses another module's database.

```
Module A → API Gateway → Module B
         ↕               ↕
     Module A DB      Module B DB
```

### 4.2 Event-based communication

Modules publish and subscribe to NATS JetStream events for asynchronous communication:

| Event type | Purpose | Example |
|---|---|---|
| `domain.[module].created` | Entity created | `domain.health.patient-registered` |
| `domain.[module].updated` | Entity updated | `domain.logistics.shipment-dispatched` |
| `domain.[module].deleted` | Entity deleted | `domain.protection.case-closed` |

### 4.3 Cross-module data access

If Module A needs data from Module B, it must use Module B's API. Caching is permitted but must respect cache invalidation from Module B's events.

## 5. Module registry

All available modules are registered in the module registry, which powers the Marketplace and deployment tooling.

The registry is a single YAML index in the `zs-infra-module-registry` repository. Each module entry includes its manifest, supported planes, and deployment instructions.

## 6. Cross-references

→ **003-zs-platform/001-platform-overview.md** — Platform architecture, system boundaries, and the platform-of-platforms structure
→ **003-zs-platform/008-domain-registry.md** — The 40-domain classification every domain module maps to
→ **003-zs-platform/006-api-design.md** — API contracts and authentication modules communicate through
→ **003-zs-platform/007-data-model.md** — ZS-UID identifier system and cross-module data isolation rules
→ **010-zs-ecosystem/010-modules-spec.md** — Modules component specification: independent deployment and Marketplace integration
→ **001-zs-meta/001-zarishsphere-constitution.md** — Law 7 (module sovereignty) and the constitutional basis for this architecture

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
