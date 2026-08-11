---
id: "ZS-INDEX-007"
title: "zarishsphere-technology-stack"
domain: "007-zs-tech-stack"
doc-type: "index"
entity-type: "landing-page"
description: >-
  Landing page for the Technology Stack part of the ZarishSphere documentation. Indexes the six documents covering the master tech stack, the Go FHIR server, the frontend stack, the data pipeline, no-code tools, and the authoritative version-pinned OSS tool catalog.
version: "1.0.0"
status: "stable"
tags:
  - "index"
  - "tech-stack"
  - "tooling"
isolation_tier: "global"
canonical: true
audience:
  - "contributors"
  - "deployers"
  - "ai-agents"
last_updated: "2026-08-01"
---
# 000-technology-stack.md
## Technology stack — the pinned, version-exact toolkit

The pinned technology stack for the ZarishSphere ecosystem. The authoritative version source is the OSS tool catalog (006); the master stack document (001) records the design constraints. Six documents cover the full toolchain from backend to no-code tooling.

## Document index

| # | File | Description | Type | Status |
|---|---|---|---|---|
| 001 | [001-tech-stack-master.md](001-tech-stack-master.md) | Master production tech-stack mapping: Go-native backend, PostgreSQL 18.4 primary with SQLite on Planes 0 and 1, React and Next.js frontend, GitHub and Cloudflare free-tier infrastructure, zero-cost and no-JVM constraints | Reference | Stable |
| 002 | [002-go-fhir-server.md](002-go-fhir-server.md) | Specification for the zero-JVM custom-Go FHIR R5 server (zs-fhir-server) built on fhir-toolbox-go: Chi and PostgreSQL, 9 V1 resources, search params, transaction bundles, per-plane performance targets | Specification | Stable |
| 003 | [003-frontend-stack.md](003-frontend-stack.md) | Frontend specification: React 19.2.8 and Next.js 16.2.12, Carbon Design System as the primary UI library, service-worker offline support, WCAG 2.1 AA, Console dashboard layout | Specification | Stable |
| 004 | [004-data-pipeline.md](004-data-pipeline.md) | Go-native batch ETL specification: PostgreSQL canonical store (SQLite on Planes 0 and 1), file-based transport with NATS 2.14.4 for async, systemd-timer scheduling, CSV and JSON and FHIR-Bundle and Parquet exports | Specification | Stable |
| 005 | [005-no-code-tools.md](005-no-code-tools.md) | No-code and low-code specification: declarative YAML and JSON form definitions, workflow DSL, ZSM module packaging, Builder GUI, Marketplace distribution, full Plane 0 offline operation | Specification | Stable |
| 006 | [006-oss-tool-catalog.md](006-oss-tool-catalog.md) | Version-pinned OSS catalog, the authoritative source for every tool, library, and free-tier service: backend, frontend, mobile, desktop, infrastructure, observability, data and storage, security, terminologies, and licensing | Reference | Stable |

## See also

→ **README.md** — Space landing page and how the ten domains fit together
→ **003-zs-platform/001-platform-overview.md** — The platform these tools build
→ **008-zs-adrs/000-architecture-decision-records.md** — The decisions that pin this stack (ADR-013, ADR-014, ADR-019, ADR-023)
