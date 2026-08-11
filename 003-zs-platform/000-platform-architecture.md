---
id: "ZS-INDEX-003"
title: "zarishsphere-platform-architecture"
domain: "003-zs-platform"
doc-type: "index"
entity-type: "landing-page"
description: >-
  Landing page for the Platform Architecture part of the ZarishSphere documentation. Indexes the ten platform documents covering the platform overview, module architecture, the five deployment planes, the G2A engine, FHIR R5 integration, API design, the data model, the 40-domain registry, the repository catalog, and the free-tier resource map.
version: "1.0.0"
status: "stable"
tags:
  - "index"
  - "platform"
  - "architecture"
isolation_tier: "global"
canonical: true
audience:
  - "contributors"
  - "deployers"
  - "ai-agents"
last_updated: "2026-08-01"
---
# 000-platform-architecture.md
## Platform architecture specifications

Technical architecture for the ZarishSphere Platform — a modular, Plane 0-compatible, Go-native platform of platforms covering ten documents from overview through the free-tier resource map.

## Document index

| # | File | Description | Type | Status |
|---|---|---|---|---|
| 001 | [001-platform-overview.md](001-platform-overview.md) | Technical overview of the deployable, Go-native, Plane 0-compatible platform: architecture principles, system boundaries, five-plane deployment model, component tree, pinned tech stack | Specification | Stable |
| 002 | [002-module-architecture.md](002-module-architecture.md) | Module-architecture rules: module sovereignty (Law 7), standard module layout, manifest.yaml, lifecycle states, API and event-based communication, module registry | Specification | Stable |
| 003 | [003-deployment-planes.md](003-deployment-planes.md) | Detailed specifications for all five deployment planes (air-gapped 0 → global SaaS 4): hardware minimums, per-plane scope, sync mechanisms, Plane 0 USB and QR export, DHIS2 and FDMN data-flow rules | Specification | Stable |
| 004 | [004-g2a-engine.md](004-g2a-engine.md) | Guideline-to-Action Engine: six-stage pipeline (ingest, parse, extract, validate, stage, deploy) converting indexed standards into FHIR R5, DHIS2, JSON, and Markdown assets | Specification | Stable |
| 005 | [005-fhir-architecture.md](005-fhir-architecture.md) | FHIR R5 integration architecture: Go-native zs-fhir-server (HAPI rejected per ADR-004), R5-over-R4 bridge, core zs-* profiles, SMART App Launch 2.2.0, offline FHIR | Specification | Stable |
| 006 | [006-api-design.md](006-api-design.md) | API design principles and contracts: REST, GraphQL, and webhooks; OpenAPI 3.2 specs; URL major-versioning; auth; scope model; rate limits; error format | Specification | Stable |
| 007 | [007-data-model.md](007-data-model.md) | Core data model: ZS-UID format, encrypted cross-reference table, core entities, data sovereignty and FDMN routing, emergency key destruction, export formats | Specification | Stable |
| 008 | [008-domain-registry.md](008-domain-registry.md) | The complete 40-domain classification taxonomy: each domain's ZI prefix, module code, scope, prefix conventions, and domain expansion rules | Specification | Stable |
| 009 | [009-repository-catalog.md](009-repository-catalog.md) | Authoritative catalog of 207 listed repositories (212+ planned) across 11 functional layers with the zs-{layer}-{module} naming convention | Reference | Stable |
| 010 | [010-free-tier-resource-map.md](010-free-tier-resource-map.md) | Map of every zero-cost and free-tier resource by category (GitHub, Cloudflare, compute, databases, CI/CD, observability, terminology sources) with limits and six architectural constraints | Reference | Stable |

## See also

→ **README.md** — Space landing page and how the ten domains fit together
→ **007-zs-tech-stack/006-oss-tool-catalog.md** — Version-pinned open-source tool catalog
→ **008-zs-adrs/000-architecture-decision-records.md** — The decisions behind this architecture
