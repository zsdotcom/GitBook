# 001-tech-stack-master.md
## Master production tech stack mapping
### Complete technology inventory across backend, frontend, infrastructure, data, and no-code layers

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

> **Alignment note:** This document is maintained in alignment with → **007-zs-tech-stack/006-oss-tool-catalog.md**, which is authoritative for version pinning. Where any version in this document differs from the catalog, the catalog prevails.

---

## Table of contents

1. [Purpose and scope](#1-purpose-and-scope)
2. [Technology inventory](#2-technology-inventory)
3. [Backend stack](#3-backend-stack)
4. [Frontend stack](#4-frontend-stack)
5. [Infrastructure stack](#5-infrastructure-stack)
6. [Data pipeline stack](#6-data-pipeline-stack)
7. [No-code stack](#7-no-code-stack)
8. [Zero-cost justification](#8-zero-cost-justification)
9. [Cross-references](#9-cross-references)

---

## 1. Purpose and scope

This document defines every technology choice in the ZarishSphere ecosystem. It is the single authoritative reference for what tools, frameworks, libraries, and services are used across all layers of the platform.

All technology decisions are governed by three binding constraints:

> **Constraint:** Every dependency must be zero-cost, open-source, or available on a free tier. No paid tools, licenses, or services are permitted. See ADR-006.

> **Constraint:** No JVM-based dependency may be introduced anywhere in the platform. Go-native alternatives are required for all server-side components. See Law 11 of the constitution and ADR-004.

> **Constraint:** FHIR R5 is the canonical health data standard. R4 compatibility is provided through a translation layer, not by running R4 natively. See ADR-005.

The stack is optimized for a single constraint environment: a Lenovo i3 laptop with 8 GB RAM running Ubuntu (founder development machine) and Raspberry Pi 5 deployments (Plane 1). Every tool choice must function under these limits.

---

## 2. Technology inventory

### 2.1 Master technology table

| Layer | Technology | Version | Purpose | Zero-cost | License |
|---|---|---|---|---|---|
| Backend language | Go | 1.26.5 | All server-side services, FHIR server, G2A Engine | Native binary, no runtime cost | BSD-3-Clause |
| Backend router | Chi (go-chi) | v5.2.1 | HTTP routing for all Go services | Open source | MIT |
| Backend database | PostgreSQL | 18.4 | Primary data store for Planes 2-4 (JSONB + GIN, TimescaleDB 2.29.0, RLS) | Open source | PostgreSQL License |
| Fallback database | SQLite | 3.53.4 | Embedded data store on Planes 0-1 (offline, air-gapped) | Public domain | Public domain |
| Message broker | NATS JetStream | 2.14.4 | Async messaging backbone on Planes 1+; file transport is the Plane 0 fallback | Open source | Apache 2.0 |
| Cache | Valkey | 9.1.1 | Cache + session store | Open source | BSD-3 |
| FHIR implementation | zs-fhir-server (Go) | V1 | FHIR R5 server built on fhir-toolbox-go; no Java/HAPI | Custom build | Apache 2.0 |
| FHIR R5 engine | fhir-toolbox-go | v0.1.2 | Primary FHIR R5 engine — FHIRPath, REST, search | Open source | MIT |
| FHIR Go models | gofhir-models (fastenhealth) | v0.0.7 | Generated Go structs for FHIR R5 | Open source | Apache 2.0 |
| Frontend framework | React | 19.2.8 | UI component library | Open source | MIT |
| Frontend meta-framework | Next.js | 16.2.12 | App Router, SSR, SSG, API routes | Open source | MIT |
| Frontend language | TypeScript | 7.0.2 | Type-safe frontend code | Open source | Apache 2.0 |
| UI component library | Carbon Design System (@carbon/react) | 1.113.0 | Primary UI component library, WCAG 2.2 AA | Open source | Apache 2.0 |
| Styling | Tailwind CSS | 4.3.3 | Utility-first CSS for content surfaces | Open source | MIT |
| Component library (content surfaces) | Shadcn UI | v4 | Accessible React components for content surfaces | Open source | MIT |
| Package manager | pnpm | 10 | Fast, disk-efficient package management | Open source | MIT |
| Build tool | Vite | 8.2.0 | Frontend build and dev server | Open source | MIT |
| Mobile framework | Flutter / Dart | 3.44.7 / 3.12.2 | Cross-platform mobile (Android, iOS) | Open source | BSD-3 |
| Mobile state | Riverpod | 3.4.2 | Null-safe, testable state management | Open source | MIT |
| Mobile offline sync | PowerSync | 1.23.3 | SQLite offline sync to PostgreSQL | Open source | Apache 2.0 |
| Identity | Keycloak | 26.7.0 | IAM, OAuth 2.1, OIDC, SMART on FHIR | Open source | Apache 2.0 |
| Secrets | Vault | 2.0.3 | Secrets management (BSL 1.1 source-available; OpenBao is the open-source alternative) | Source-available | BSL 1.1 |
| GitOps | Argo CD | 3.4.6 | Declarative GitOps on Plane 2+ | Open source | Apache 2.0 |
| Orchestration | Kubernetes | 1.36 | Container orchestration on Plane 2+ | Open source | Apache 2.0 |
| Reverse proxy | Traefik | 3.7.10 | Cloud-native reverse proxy, auto-HTTPS | Open source | MIT |
| Networking | Cilium | 1.20.0 | eBPF networking, service mesh | Open source | Apache 2.0 |
| Infrastructure as Code | OpenTofu | 1.12.5 | IaC (Linux Foundation fork of Terraform) | Open source | MPL-2.0 |
| Observability | Grafana + Prometheus | 13.1.1 / 3.13.2 | Dashboards and metrics | Open source | AGPL-3.0 / Apache 2.0 |
| Infrastructure DNS | Cloudflare DNS | Free tier | Authoritative DNS, zone management | Free tier | N/A |
| Infrastructure CDN | Cloudflare CDN | Free tier | Global caching and edge delivery | Free tier | N/A |
| Infrastructure hosting | Cloudflare Pages | Free tier | Static sites, Next.js SSR | Free tier | N/A |
| Edge functions | Cloudflare Workers | Free tier | API gateway, auth, transformations | Free tier (100k req/day) | N/A |
| Object storage | Cloudflare R2 | Free tier (10 GB) | File and backup storage | Free tier | N/A |
| Edge database | Cloudflare D1 | Free tier | Edge-cached read replicas | Free tier | N/A |
| Version control | GitHub | Free org | All repositories, issues, Actions | Free tier | N/A |
| CI/CD | GitHub Actions | Free tier (2000 min/mo) | Automated build, test, deploy | Free tier | N/A |
| Container registry | GitHub Container Registry | Free | Docker image hosting | Free tier | N/A |
| ETL processing | Custom Go tooling | V1 | Data ingestion and transformation | Custom build | Apache 2.0 |
| Analytics format | Parquet | Arrow 18 | Columnar analytics storage | Open source | Apache 2.0 |
| Serialization | YAML / JSON | Standard | Configuration, form definitions, workflows | Standard | N/A |
| Offline queue | Custom Go + SQLite | V1 | Local write buffer with sync (Plane 0 fallback) | Custom build | Apache 2.0 |
| Templating | Go html/template | stdlib | Server-side rendered dashboards | Standard (Go stdlib) | BSD-3-Clause |

### 2.2 Explicit non-choices

| Technology | Reason rejected | ADR or law |
|---|---|---|
| Java / JVM | 8 GB RAM limit, Plane 0 constraint | Law 11, ADR-004 |
| HAPI FHIR | Java dependency, heavy footprint | ADR-004 |
| Python (backend) | GIL, slow startup, weak concurrency | ADR-001 |
| Node.js (backend) | Non-native typed concurrency model | ADR-001 |
| Rust | Steep learning curve, slower iteration | ADR-001 (Go preferred for simplicity) |
| Kafka | Heavy dependency, Plane 0 incompatible | ADR-006 (NATS JetStream chosen) |
| MongoDB | Document store unnecessary for FHIR | Chosen: PostgreSQL JSONB for relational + JSON queries |
| Redux | Over-engineered for Plane 0 scope | Chosen: React Context + hooks |
| Zustand / Jotai | Unnecessary for Plane 0 scope | Chosen: React Context + hooks |
| Docker Swarm | Kubernetes ecosystem needed on Plane 2+ | Chosen: Kubernetes on Plane 2+, single-binary on Planes 0/1 |
| Kubernetes (Planes 0/1) | Impractical for 8 GB laptop / RPi dev | Used on Plane 2+ only; Go single-binary pattern below |

---

## 3. Backend stack

### 3.1 Go as primary language

Go is mandated by ADR-001. All backend services — FHIR server, G2A Engine, module runtime, CLI tools — are written in Go.

**Rationale summary:**

- Single binary output — no runtime, no VM, no JIT warmup
- Cross-compilation to ARM64 for Raspberry Pi (Plane 1)
- Built-in concurrency with goroutines for parallel FHIR processing
- Static typing without generics complexity
- Sub-1-second startup times for cold-start deployments
- Memory footprint: 15-150 MB depending on module load

### 3.2 HTTP routing: Chi v5

Chi is the router for all Go HTTP services. Selected over standard `net/http` for middleware chaining and URL parameter support, and over Gin for its explicit `net/http` compatibility.

| Feature | Chi | Gin |
|---|---|---|
| `net/http` compatible | Yes (native) | Wrapper |
| Middleware chaining | Yes (idiomatic) | Custom context |
| Performance | Fast | Slightly faster |
| Community size | Moderate | Large |
| Explicit context | Yes (Go 1.26.5 context) | Custom gin.Context |

### 3.3 Database: PostgreSQL 18.4 primary, SQLite fallback

PostgreSQL 18.4 is the primary database on Planes 2-4. SQLite 3.53.4 is the embedded fallback on Planes 0-1 (offline and air-gapped). Selected because:

- **PostgreSQL (primary):** JSONB columns with GIN indexes for FHIR resource storage, TimescaleDB 2.29.0 for time-series vitals and events, row-level security (RLS) for multi-tenant isolation
- **SQLite (Planes 0-1):** zero configuration, single file per database, WAL mode for concurrent reads during writes, backups are `cp` commands
- Full SQL support with JSON functions for FHIR storage on both engines
- Plane 4 (global SaaS) uses a PostgreSQL fleet with read-replica fan-out

> **Constraint:** The storage layer must run on both PostgreSQL 18.4 and SQLite. Plane-specific SQL is isolated behind the storage package; any feature used in a stored query must be supported by both engines or gated per plane.

### 3.4 FHIR server

The FHIR server is a custom Go implementation (`zs-fhir-server`) using Chi for routing, built on the fhir-toolbox-go FHIR R5 engine, with PostgreSQL 18.4 (Planes 2-4) or SQLite (Planes 0-1) for storage. See → **007-zs-tech-stack/002-go-fhir-server.md** — the complete FHIR server specification.

### 3.5 Backend service architecture

```
zs-fhir-server/     — FHIR R5 REST API
zs-g2a-engine/     — Guideline-to-Action transformation engine
zs-module-runtime/  — Domain module executor
zs-sync/            — Offline sync and conflict resolution
zs-export/          — Data export (CSV, JSON, Parquet, FHIR Bundle)
zs-cli/             — Command-line administration tool
zs-builder-api/     — Backend API for the Builder UI
```

---

## 4. Frontend stack

### 4.1 React 19.2.8 + Next.js 16.2.12

The frontend is built on React 19.2.8 with Next.js 16.2.12 App Router. Selected for server components, streaming, and static generation.

**Key version choices:**

| Package | Version | Purpose |
|---|---|---|
| react | 19.2.8 | UI component library |
| react-dom | 19.2.8 | DOM rendering |
| next | 16.2.12 | Meta-framework with App Router |
| typescript | 7.0.2 | Type safety |
| @carbon/react | 1.113.0 | Primary UI component library (Carbon Design System) |
| tailwindcss | 4.3.3 | Utility-first styling for content surfaces |
| shadcn/ui | v4 | Accessible component primitives for content surfaces |
| vite | 8.2.0 | Build tooling |

### 4.2 Styling and components

- **Carbon Design System (@carbon/react 1.113.0)** is the primary UI component library — Apache 2.0, WCAG 2.2 AA, healthcare-focused components
- **Tailwind CSS 4.3.3** provides utility-first CSS with zero runtime cost for content surfaces
- **Shadcn UI** provides accessible, copy-paste React components built on Radix UI primitives for content surfaces

### 4.3 State management

- **React hooks + Context** for global client state (theme, auth, sidebar) — the Carbon Design System integrates with React Context
- **No Redux, Zustand, or Jotai** — unnecessary for Plane 0 deployments
- **Server state** managed through Next.js server components and fetch caching, with TanStack Query v5.64.0 for FHIR data fetching

### 4.4 Build and development

- **Vite 8.2.0** as the build tool (Next.js uses Turbopack in dev, webpack in production)
- **pnpm 10** as the package manager (faster, disk-efficient, strict)
- **TypeScript 7.0.2 strict mode** enabled in all frontend packages

### 4.5 Frontend applications

| Application | Stack | Purpose |
|---|---|---|
| Console | Next.js 16.2.12, React 19.2.8, Carbon Design System | Browser-based management center |
| Forms engine | React 19.2.8, dynamic component loader | Renders declarative form definitions |
| Public site | Next.js 16.2.12 SSG | Documentation, marketing |
| Marketplace | Next.js 16.2.12, React 19.2.8 | Component discovery and deployment |

See → **007-zs-tech-stack/003-frontend-stack.md** — the complete frontend specification.

---

## 5. Infrastructure stack

### 5.1 GitHub (free tier)

| Service | Usage | Monthly limit | Estimated usage |
|---|---|---|---|
| Public repositories | All code, docs, standards | Unlimited | 15-25 repos |
| GitHub Actions | CI/CD, lint, deploy | 2,000 min/mo | ~500 min/mo |
| GitHub Container Registry | Docker images | Free | ~10 images |
| GitHub Pages | Documentation sites | Free | zs-docs site |
| GitHub Issues + Projects | Issue tracking | Unlimited | Active |

### 5.2 Cloudflare (free tier)

| Service | Usage | Free tier limit | Notes |
|---|---|---|---|
| DNS | Authoritative DNS | Unlimited zones | All zarishsphere.com domains |
| CDN | Global asset caching | Unlimited bandwidth | Standard CDN |
| Pages | Static and SSR hosting | 500 builds/mo, 500 GB bandwidth | All frontend apps |
| Workers | Edge API, auth proxy | 100k req/day | API gateway functions |
| R2 | Object storage | 10 GB storage, 10M reads/mo | Backups, exports, uploads |
| D1 | Edge SQLite | 5 GB storage, 5M reads/mo | Edge read replica cache |

### 5.3 Plane 2+ self-hosted infrastructure

On Planes 2-4 the platform adds a self-hosted Kubernetes stack managed through GitOps:

| Component | Technology | Version | Role |
|---|---|---|---|
| Orchestration | Kubernetes | 1.36 | Container orchestration |
| GitOps | Argo CD | 3.4.6 | Declarative application delivery |
| Networking | Cilium | 1.20.0 | eBPF networking, service mesh |
| Reverse proxy | Traefik | 3.7.10 | Auto-HTTPS, edge routing |
| Infrastructure as Code | OpenTofu | 1.12.5 | Provisioning and configuration |
| Identity | Keycloak | 26.7.0 | IAM, OIDC, SMART on FHIR |
| Secrets | Vault | 2.0.3 | Secrets management (OpenBao alternative) |

### 5.4 Network architecture

```
zarishsphere.com
  ├── Pages: Console, Public Site, Marketplace
  ├── Workers: API Gateway, Auth Proxy, Rate Limiter
  ├── R2: Form uploads, module assets, backups
  └── D1: Edge-cached read replicas
```

See → **006-zs-infrastructure/003-cloudflare-architecture.md** — the complete infrastructure specification.
See → **006-zs-infrastructure/002-github-org-architecture.md** — GitHub configuration.

---

## 6. Data pipeline stack

### 6.1 Core components

| Component | Technology | Function |
|---|---|---|
| ETL processor | Custom Go | Ingests FHIR, CSVs, JSON, ZARISH-INDEX data |
| Async messaging | NATS 2.14.4 JetStream | Inter-service events, pipeline triggers (Planes 1+) |
| Primary storage | PostgreSQL 18.4 / SQLite (Planes 0-1) | Transactional FHIR resources |
| Analytics storage | Parquet files | Columnar format for reporting |
| Export engine | Custom Go | FHIR Bundle, CSV, JSON, Parquet |
| Scheduled sync | systemd timers / cron | Periodic ETL runs |
| Reporting | Go html/template | Lightweight server-side dashboards |

### 6.2 Data flow

```
Data Sources → Go ETL → PostgreSQL / SQLite (transactional) → Export Engine → Parquet/CSV/JSON
                          ↕
               Go Template Dashboards (local browser)
```

> **Constraint:** No Apache Kafka, Flink, Spark, or any JVM-based data processing tool may be introduced. Data pipelines must run on Go single-binary ETL processes. NATS 2.14.4 JetStream is the async backbone on Planes 1+; the file-based offline transport remains the Plane 0 fallback.

See → **007-zs-tech-stack/004-data-pipeline.md** — the complete pipeline specification.

---

## 7. No-code stack

### 7.1 Declarative definition formats

All no-code tools use YAML or JSON declarative formats. No custom scripting language is introduced.

| Asset | Format | Schema |
|---|---|---|
| Forms | YAML / JSON | ZARISH-STANDARDS form schema |
| Workflows | YAML DSL | ZS workflow schema |
| Module manifests | YAML (ZSM format) | ZS module manifest schema |
| Dashboards | YAML | ZS dashboard schema |

### 7.2 Runtime components

| Component | Technology | Purpose |
|---|---|---|
| Forms engine | React 19.2.8 (browser) + Go (server) | Renders forms from YAML/JSON |
| Workflow engine | Go | Executes workflow DSL |
| Module loader | Go | Installs and runs ZSM packages |
| Builder UI | React 19.2.8 | GUI for creating forms and workflows |

See → **007-zs-tech-stack/005-no-code-tools.md** — the complete no-code specification.
See → **010-zs-ecosystem/003-builder-spec.md** — the Builder component specification.
See → **010-zs-ecosystem/005-forms-spec.md** — the Forms engine specification.

---

## 8. Zero-cost justification

Every technology in the stack is selected to meet the zero-cost requirement (ADR-006) and the single-developer constraint. The table below shows the cost justification for each layer.

### 8.1 Cost comparison

| Layer | Selected | Monthly cost | Alternative | Alternative monthly cost |
|---|---|---|---|---|
| Backend runtime | Go binary | $0 | Java VM + server | $50+ (cloud VM) |
| Database | PostgreSQL 18.4 self-hosted + SQLite (Planes 0-1) | $0 | PostgreSQL RDS (managed) | $15+ (managed DB) |
| Messaging | NATS JetStream self-hosted | $0 | Kafka + MSK | $100+/mo |
| Hosting | Cloudflare Pages | $0 | Vercel Pro | $20/mo |
| CI/CD | GitHub Actions | $0 | CircleCI | $30/mo |
| CDN | Cloudflare CDN | $0 | Fastly | $50+ |
| Object storage | Cloudflare R2 | $0 (10 GB) | AWS S3 | ~$3/mo (small use) |
| Container registry | GHCR | $0 | Docker Hub Pro | $5/mo |
| Package manager | pnpm | $0 | N/A | $0 |
| Frontend build | Vite | $0 | Webpack commercial | $0 |
| Monitoring | Grafana + Prometheus self-hosted | $0 | Datadog | $15+/mo |
| **Total** | | **$0** | | **$250+/mo** |

### 8.2 Platform-specific constraints

> **Constraint:** No toolchain component may require a credit card to start. Every free tier must have sufficient capacity for Plane 0 and Plane 1 deployments at full production load.

> **Constraint:** If a free-tier service changes its pricing model and becomes paid, a migration path to an alternative zero-cost tool must be documented within 90 days.

---

## 9. Cross-references

→ **007-zs-tech-stack/006-oss-tool-catalog.md** — Authoritative, version-pinned OSS tool catalog for the ecosystem
→ **007-zs-tech-stack/002-go-fhir-server.md** — Go-native FHIR R5 server specification
→ **007-zs-tech-stack/003-frontend-stack.md** — Frontend framework and UI architecture
→ **007-zs-tech-stack/004-data-pipeline.md** — Data pipeline and analytics specification
→ **007-zs-tech-stack/005-no-code-tools.md** — No-code and low-code tooling specification
→ **008-zs-adrs/013-adr-postgresql-primary-database.md** — ADR-013: PostgreSQL 18.4 primary database
→ **008-zs-adrs/014-adr-nats-jetstream-messaging.md** — ADR-014: NATS JetStream messaging backbone
→ **008-zs-adrs/019-adr-carbon-design-system.md** — ADR-019: Carbon Design System as primary UI library
→ **008-zs-adrs/023-adr-flutter-cross-platform-mobile.md** — ADR-023: Flutter cross-platform mobile
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: zero-cost toolchain
→ **003-zs-platform/001-platform-overview.md** — Platform architecture, deployment planes, and system boundaries
→ **006-zs-infrastructure/003-cloudflare-architecture.md** — Cloudflare edge architecture (DNS, CDN, Pages, Workers, R2)
→ **006-zs-infrastructure/002-github-org-architecture.md** — GitHub organization and repository configuration
→ **010-zs-ecosystem/003-builder-spec.md** — Builder application specification
→ **010-zs-ecosystem/005-forms-spec.md** — Forms engine specification

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
