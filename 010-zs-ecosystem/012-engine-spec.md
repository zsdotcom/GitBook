# 012-engine-spec.md
## ZarishSphere Engine specification
### Core runtime

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Engine sub-components](#2-engine-sub-components)
3. [Execution model](#3-execution-model)
4. [Plane 0 operation](#4-plane-0-operation)
5. [Cross-references](#5-cross-references)

---

## 1. Purpose

The ZarishSphere Engine is the core runtime that executes all platform operations. It is the execution layer beneath every other component — the Console, Marketplace, Builder, Apps, Forms, Services, and Modules all depend on the Engine.

## 2. Engine sub-components

| Sub-component | Purpose | Technology |
|---|---|---|
| G2A Engine | Guideline-to-action transformation | Go + Python |
| Module runtime | Execution environment for domain modules | Go |
| Form engine | Dynamic form rendering and data capture | Go + React |
| Workflow engine | Approval chains, data pipelines, automation | Go + NATS |
| Sync engine | Offline data synchronization | Go + NATS JetStream |
| Report engine | Dashboard and report generation | Go + Grafana |

## 3. Execution model

```
Request → API Gateway → Engine orchestrator → Sub-component
                                                  ↓
                                           PostgreSQL / NATS / MinIO
                                                  ↓
                                              Response
```

The Engine orchestrator routes requests to the appropriate sub-component based on the request type. Each sub-component is independently deployable.

## 4. Plane 0 operation

At Plane 0, the Engine runs in standalone mode:
- Single binary with embedded sub-components
- SQLite instead of PostgreSQL
- Local file storage instead of MinIO
- NATS in single-node mode
- AI-dependent features disabled
- Forms pre-rendered as static bundles

## 5. Cross-references

→ **010-zs-ecosystem/009-services-spec.md** — Services specification: built on the Engine's shared Go libraries
→ **010-zs-ecosystem/013-system-spec.md** — System specification: the base environment beneath the Engine
→ **010-zs-ecosystem/005-forms-spec.md** — Forms specification: the Form engine sub-component's rendering contract
→ **010-zs-ecosystem/010-modules-spec.md** — Modules specification: the module runtime sub-component's execution contract
→ **010-zs-ecosystem/015-content-protocols-spec.md** — zs-content-protocols specification: the protocol inputs the G2A Engine consumes
→ **010-zs-ecosystem/017-content-reports-spec.md** — zs-content-reports specification: the report templates the Report engine renders
→ **003-zs-platform/004-g2a-engine.md** — G2A Engine: the platform-side architecture of the guideline-to-action transformation
→ **003-zs-platform/003-deployment-planes.md** — Deployment plane model: Plane 0 standalone mode

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
