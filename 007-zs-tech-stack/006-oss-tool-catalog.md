# 006-oss-tool-catalog.md
## Open-source software tool catalog
### Complete reference of every OSS tool, library, and service in the ZarishSphere ecosystem

**Document type:** Reference catalog
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose and governance](#1-purpose-and-governance)
2. [Backend tools (Go)](#2-backend-tools-go)
3. [Frontend tools (TypeScript / React)](#3-frontend-tools-typescript--react)
4. [Mobile tools (Flutter / Dart)](#4-mobile-tools-flutter--dart)
5. [Desktop tools (Tauri / Rust)](#5-desktop-tools-tauri--rust)
6. [Infrastructure tools](#6-infrastructure-tools)
7. [Observability tools](#7-observability-tools)
8. [Data and storage tools](#8-data-and-storage-tools)
9. [Security tools](#9-security-tools)
10. [Clinical and health standards](#10-clinical-and-health-standards)
11. [Reference health systems](#11-reference-health-systems)
12. [Developer experience tools](#12-developer-experience-tools)
13. [Documentation tools](#13-documentation-tools)
14. [Free-tier cloud services](#14-free-tier-cloud-services)
15. [License summary](#15-license-summary)
16. [Zero-cost verification](#16-zero-cost-verification)
17. [Cross-references](#17-cross-references)

---

## 1. Purpose and governance

This catalog is the single authoritative reference for every open-source tool, library, framework, and free-tier service used in the ZarishSphere ecosystem. Every tool listed here is either:

- **Open source** with no licensing cost, or
- **Free-tier** with sufficient capacity for Plane 0 and Plane 1 deployments

This catalog is authoritative for version pinning. → **007-zs-tech-stack/001-tech-stack-master.md** is maintained in alignment with it; where versions differ, this catalog prevails.

### 1.1 Governance rules

> **Constraint:** Every new dependency added to any ZarishSphere repository must first be listed in this catalog with license and free-tier confirmation. See ADR-006.

> **Constraint:** No JVM-based dependency may be introduced. All server-side components must use Go-native alternatives. See ADR-004.

> **Constraint:** Version pins must use explicit version numbers. The marker `current-stable` is the only non-numeric version permitted.

### 1.2 Version convention

All tools are pinned to the version used or evaluated as of August 2026. Tools marked `current-stable` are expected to track the current stable release of that project without requiring documentation updates for patch releases.

---

## 2. Backend tools (Go)

### 2.1 Core language and runtime

| Tool | Version | License | Free tier | Purpose | Import / URL |
|------|---------|---------|-----------|---------|-------------|
| **Go** | 1.26.5 | BSD-3 | Always free | Primary backend language | golang.org |
| **golangci-lint** | 2.12.2 | GPL-3 | Always free | Go linter (50+ linters) | golangci-lint.run |
| **air** | 1.60.0 | MIT | Always free | Live-reload for Go development | github.com/air-verse/air |
| **goreleaser** | 2.8.0 | MIT | Free for OSS | Multi-platform binary releases | goreleaser.com |

### 2.2 FHIR libraries (Go)

| Library | Version | License | Purpose | Import path |
|---------|---------|---------|---------|-------------|
| **fhir-toolbox-go** | v0.1.2 | MIT | Primary FHIR R5 engine, FHIRPath, REST, search | `github.com/damedic/fhir-toolbox-go` |
| **gofhir-models** | v0.0.7 | Apache 2.0 | Generated Go structs for all FHIR R5 resources | `github.com/fastenhealth/gofhir-models` |
| **golang-fhir-models** | 0.4.0 | Apache 2.0 | Alternative FHIR Go models | `github.com/wardsco/golang-fhir-models` |

### 2.3 HTTP and API

| Library | Version | License | Purpose | Import path |
|---------|---------|---------|---------|-------------|
| **chi** | v5.2.1 | MIT | Idiomatic HTTP router, stdlib-compatible | `github.com/go-chi/chi/v5` |
| **chi/middleware** | v5.2.1 | MIT | HTTP middleware (CORS, compress, recover) | `github.com/go-chi/chi/v5/middleware` |
| **chi/render** | v5.2.0 | MIT | Content negotiation (JSON, XML, FHIR) | `github.com/go-chi/render` |
| **gorilla/websocket** | v1.5.3 | BSD-2 | WebSocket support for real-time features | `github.com/gorilla/websocket` |

### 2.4 Database

| Library | Version | License | Purpose | Import path |
|---------|---------|---------|---------|-------------|
| **pgx** | v5.7.2 | MIT | Pure Go PostgreSQL driver + connection pool | `github.com/jackc/pgx/v5` |
| **golang-migrate** | v4.18.0 | MIT | Database migrations, idempotent | `github.com/golang-migrate/migrate/v4` |
| **squirrel** | v1.5.0 | MIT | SQL query builder (safe parameterized queries) | `github.com/Masterminds/squirrel` |
| **sqlc** | v1.28.0 | MIT | Type-safe SQL from .sql files (codegen) | `github.com/sqlc-dev/sqlc` |

### 2.5 Caching and messaging

| Library | Version | License | Purpose | Import path |
|---------|---------|---------|---------|-------------|
| **valkey-go** | v1.0.0 | Apache 2.0 | Valkey / Redis client | `github.com/valkey-io/valkey-go` |
| **go-redis** | v9.7.0 | BSD-2 | Alternative Redis / Valkey client | `github.com/redis/go-redis/v9` |
| **nats.go** | v1.39.0 | Apache 2.0 | NATS + JetStream client | `github.com/nats-io/nats.go` |
| **asynq** | v0.25.0 | MIT | Valkey-backed task queue | `github.com/hibiken/asynq` |

### 2.6 Auth and security

| Library | Version | License | Purpose | Import path |
|---------|---------|---------|---------|-------------|
| **go-oidc** | v3.12.0 | Apache 2.0 | OIDC / SMART on FHIR token validation | `github.com/coreos/go-oidc/v3` |
| **golang-jwt** | v5.2.1 | MIT | JWT parsing, SMART scope enforcement | `github.com/golang-jwt/jwt/v5` |
| **golang.org/x/crypto** | v0.31.0 | BSD-3 | bcrypt, AES, PBKDF2 | `golang.org/x/crypto` |

### 2.7 Observability (Go SDK)

| Library | Version | License | Purpose | Import path |
|---------|---------|---------|---------|-------------|
| **OpenTelemetry Go** | v1.40.0 | Apache 2.0 | Distributed tracing, metrics | `go.opentelemetry.io/otel` |
| **otelhttp** | v0.58.0 | Apache 2.0 | HTTP tracing middleware | `go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp` |
| **prometheus/client_golang** | v1.21.0 | Apache 2.0 | Prometheus metrics | `github.com/prometheus/client_golang` |
| **zerolog** | v1.33.0 | MIT | Zero-allocation structured JSON logging | `github.com/rs/zerolog` |

### 2.8 Configuration and utilities

| Library | Version | License | Purpose | Import path |
|---------|---------|---------|---------|-------------|
| **viper** | v1.20.0 | MIT | Config management (YAML, env, Vault) | `github.com/spf13/viper` |
| **cobra** | v1.9.0 | Apache 2.0 | CLI framework (admin commands) | `github.com/spf13/cobra` |
| **samber/lo** | v1.49.0 | MIT | Generics utilities (Map, Filter, Find) | `github.com/samber/lo` |
| **google/uuid** | v1.6.0 | BSD-3 | UUID generation | `github.com/google/uuid` |
| **testcontainers-go** | v0.35.0 | MIT | Docker containers in tests (PostgreSQL, NATS) | `github.com/testcontainers/testcontainers-go` |
| **testify** | v1.10.0 | MIT | Test assertions and mocking | `github.com/stretchr/testify` |

---

## 3. Frontend tools (TypeScript / React)

### 3.1 Core framework

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **React** | 19.2.8 | MIT | UI framework (Server Components, RSC) |
| **Next.js** | 16.2.12 | MIT | Meta-framework (App Router, Server Actions, Turbopack) |
| **TypeScript** | 7.0.2 | Apache 2.0 | Type-safe JavaScript (strict mode) |
| **Vite** | 8.2.0 | MIT | Frontend build tool (dev server) |

### 3.2 Design and UI

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **Carbon Design System (@carbon/react)** | 1.113.0 | Apache 2.0 | Primary UI component library, WCAG 2.2 AA |
| **Tailwind CSS** | 4.3.3 | MIT | Utility-first CSS (content surfaces) |
| **Tiptap** | 2.11.0 | MIT | Rich text / clinical notes editor |
| **Radix UI** | 4.0.0 | MIT | Accessible unstyled primitives |

### 3.3 Forms and validation

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **React Hook Form** | 7.55.1 | MIT | Performant form state management |
| **Zod** | 4.4.3 | MIT | TypeScript-first schema validation |
| **@rjsf/core** | 5.24.0 | Apache 2.0 | JSON Schema Form renderer (alternative form engine) |

### 3.4 State management

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **TanStack Query** | v5.64.0 | MIT | Server state management, FHIR data fetching |
| **TanStack Table** | v8.20.0 | MIT | Headless table for clinical data grids |

> **Note:** Client-side global state uses React Context + hooks (see → **007-zs-tech-stack/003-frontend-stack.md**). Redux, Zustand, and Jotai are not used anywhere in the ecosystem.

### 3.5 Offline and storage

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **Dexie.js** | 4.4.3 | Apache 2.0 | IndexedDB wrapper — offline FHIR storage |
| **idb** | 8.0.0 | ISC | Low-level IndexedDB promises |
| **workbox** | 7.3.2 | MIT | Service Worker for offline app shell |

### 3.6 Data visualization

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **Apache ECharts** | 5.6.0 | Apache 2.0 | Clinical charts, vitals trends, BI dashboards |
| **OpenLayers** | 10.3.0 | BSD-2 | Geospatial maps, facility maps, outbreak mapping |
| **react-leaflet** | 4.2.1 | Expat | Alternative map library (Leaflet wrapper) |
| **Recharts** | 2.15.0 | MIT | Simple React charts |

### 3.7 i18n and accessibility

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **i18next** | 24.3.0 | MIT | Internationalization — EN, BN, MY, UR, HI, TH |
| **react-i18next** | 15.4.0 | MIT | React bindings for i18next |
| **axe-core** | 4.10.0 | MPL-2.0 | Accessibility testing (WCAG 2.2 AA) |

### 3.8 Testing

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **Vitest** | 3.1.0 | MIT | Unit testing (Vite-native) |
| **Playwright** | 1.52.0 | Apache 2.0 | E2E browser testing |
| **@testing-library/react** | 16.2.0 | MIT | Component testing |
| **MSW (Mock Service Worker)** | 2.7.0 | MIT | API mocking for tests |
| **Storybook** | 8.x | MIT | Component documentation + visual testing |

---

## 4. Mobile tools (Flutter / Dart)

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **Flutter** | 3.44.7 | BSD-3 | Cross-platform mobile (Android, iOS) |
| **Dart** | 3.12.2 | BSD-3 | Flutter's language |
| **Riverpod** | 3.4.2 | MIT | Null-safe, testable state management |
| **PowerSync** | 1.23.3 | Apache 2.0 | SQLite offline sync to PostgreSQL |
| **drift** | 2.25.0 | MIT | SQLite ORM for Flutter (alternative to PowerSync) |
| **dio** | 5.7.0 | MIT | HTTP client for Flutter |
| **fhir** | 0.0.8 | MIT | Dart FHIR models (fhirlabs) |
| **go_router** | 14.8.0 | BSD-3 | Declarative routing |
| **flutter_secure_storage** | 9.2.0 | BSD-3 | Encrypted local storage |
| **flutter_localizations** | built-in | BSD-3 | Internationalization |
| **freezed** | 2.5.0 | MIT | Immutable data models (codegen) |

---

## 5. Desktop tools (Tauri / Rust)

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **Tauri** | 2.11.2 | MIT / Apache 2.0 | Rust-based desktop wrapper (5–10 MB vs Electron's 150 MB) |
| **Rust** | current-stable | MIT / Apache 2.0 | Tauri's native layer |
| **tauri-plugin-store** | 2.2.0 | MIT | Persistent key-value store |
| **tauri-plugin-updater** | 2.3.0 | MIT | Auto-update mechanism |

---

## 6. Infrastructure tools

### 6.1 Containers and orchestration

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **Docker** | 27.5.0 | Apache 2.0 | Container runtime |
| **Docker Compose** | v2.x | Apache 2.0 | Local development stacks |
| **Kubernetes** | 1.36 | Apache 2.0 | Container orchestration (Plane 2+) |
| **Helm** | 3.17.0 | Apache 2.0 | Kubernetes package manager |
| **kubectl** | matching | Apache 2.0 | Kubernetes CLI |
| **k9s** | 0.40.0 | Apache 2.0 | Terminal Kubernetes UI |

### 6.2 Infrastructure as Code

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **OpenTofu** | 1.12.5 | MPL-2.0 | IaC (Linux Foundation fork of Terraform) |
| **Terragrunt** | 0.76.0 | MIT | OpenTofu / Terraform DRY wrapper |
| **Pulumi** | 3.148.0 | Apache 2.0 | Alternative IaC (if needed) |
| **Crossplane** | 1.18.0 | Apache 2.0 | Kubernetes-native IaC |

### 6.3 GitOps and CI/CD

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **Argo CD** | 3.4.6 | Apache 2.0 | Declarative GitOps for Kubernetes |
| **Argo Rollouts** | 1.8.0 | Apache 2.0 | Progressive delivery, canary, blue-green |
| **Flux CD** | 2.5.0 | Apache 2.0 | Alternative GitOps (CNCF graduated) |
| **GitHub Actions** | — | Proprietary | CI/CD (2000 min/month free for public repos) |
| **act** | 0.2.74 | MIT | Run GitHub Actions locally |

### 6.4 Networking and reverse proxy

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **Traefik** | v3.7.10 | MIT | Cloud-native reverse proxy, auto-HTTPS |
| **Cilium** | 1.20.0 | Apache 2.0 | eBPF networking, service mesh, CNCF graduated |
| **cert-manager** | 1.16.0 | Apache 2.0 | Automatic TLS certificate management |

---

## 7. Observability tools

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **Prometheus** | 3.13.2 | Apache 2.0 | Metrics collection |
| **Grafana** | 13.1.1 | AGPL-3.0 | Dashboards, alerting |
| **Loki** | 3.4.0 | AGPL-3.0 | Log aggregation |
| **Tempo** | 2.7.0 | AGPL-3.0 | Distributed tracing |
| **Alertmanager** | 0.28.0 | Apache 2.0 | Alert routing |
| **OpenTelemetry Collector** | 0.120.0 | Apache 2.0 | Telemetry pipeline |
| **Grafana Alloy** | 1.8.0 | Apache 2.0 | Unified telemetry agent |
| **Uptime Kuma** | 2.0.0 | MIT | Self-hosted uptime monitoring |

---

## 8. Data and storage tools

### 8.1 Databases (self-hosted)

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **PostgreSQL** | 18.4 | PostgreSQL | Primary database — FHIR JSONB, GIN search, RLS |
| **TimescaleDB** | 2.29.0 | Apache 2.0 (free) | Time-series extension — vitals trends, events |
| **NATS** | 2.14.4 | Apache 2.0 | Message broker + JetStream persistence |
| **Valkey** | 9.1.1 | BSD-3 | Cache + session store (Redis OSS fork) |
| **Typesense** | 28.0.0 | GPL-3 (OSS) | Typo-tolerant search engine (self-hosted) |
| **SQLite** | 3.53.4 | Public domain | Embedded DB fallback on Planes 0-1 (offline, air-gapped) |

### 8.2 Managed database free tiers

| Provider | Free tier | Use case |
|----------|-----------|----------|
| **Neon** | PostgreSQL 512 MB, branching | Development databases |
| **Supabase** | PostgreSQL 500 MB, 1 GB storage | Development / prototyping |
| **Upstash** | Redis 10k cmd/day, Kafka 10k msg/day | Development caching |
| **PlanetScale** | MySQL (alternative, limited) | Not primary |

---

## 9. Security tools

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| **HashiCorp Vault** | 2.0.3 | BSL 1.1 (source-available) | Secrets management, dynamic credentials |
| **External Secrets Operator** | 0.14.0 | Apache 2.0 | Vault to Kubernetes secret sync |
| **Keycloak** | 26.7.0 | Apache 2.0 | IAM, OAuth 2.1, OIDC, SMART on FHIR |
| **Sonar Cloud** | Free for OSS | Proprietary | Code quality scanning |
| **Snyk** | Free for OSS (200 tests/month) | Proprietary | Dependency vulnerability scanning |
| **GitGuardian** | Free for OSS | Proprietary | Secret scanning in repositories |
| **CodeQL** | Free for public repos | Proprietary | SAST security scanning |
| **Dependabot** | Free | Proprietary | Automated dependency updates |

> **Note:** Vault is licensed under BSL 1.1 (source-available). OpenBao is the open-source (MPL-2.0) alternative and may be substituted on any plane.

---

## 10. Clinical and health standards

### 10.1 Terminology sources

| Terminology | Version | Source | License | Access method |
|-------------|---------|--------|---------|--------------|
| **ICD-11** | 2025-01 | WHO | Free | REST API + bulk download |
| **SNOMED CT** | current-stable | SNOMED International | Free for DPG / OSS | RF2 bulk download |
| **LOINC** | 2.78 | Regenstrief Institute | Free | CSV download |
| **CIEL** | 2025-02 | OpenMRS / Andrew Kanter | Free | OpenMRS wiki download |
| **RxNorm** | current-stable | NLM (US) | Free | REST API (rxnav.nlm.nih.gov) |
| **CVX** | current-stable | CDC | Free | Flat file download |
| **ICD-10** | current-stable | WHO | Free (legacy) | Bulk download |
| **ATC** | current-stable | WHO | Free | Bulk download |

### 10.2 FHIR and interoperability tools

| Tool | License | Purpose |
|------|---------|---------|
| **HAPI FHIR Validator CLI** | Apache 2.0 | FHIR resource validation (CLI only, no server runtime) |
| **FHIR Shorthand (FSH)** | CC0 | Author FHIR profiles and Implementation Guides |
| **SUSHI** | Apache 2.0 | FSH compiler to FHIR IG |
| **IG Publisher** | CC0 | Publish FHIR Implementation Guides |
| **Inferno** | Apache 2.0 | FHIR compliance testing |
| **Synthea** | Apache 2.0 | Synthetic patient test data |
| **OpenConceptLab** | MPL-2.0 | Terminology hosting (free cloud) |
| **Simplifier.net** | Free tier | FHIR profile registry |

### 10.3 Health data standards

| Standard | Version | Access |
|----------|---------|--------|
| **FHIR R5** | 5.0.0 | hl7.org/fhir/R5 (CC0) |
| **SMART on FHIR** | 2.1.0 | smarthealthit.org (free) |
| **CDS Hooks** | 2.0.0 | cds-hooks.org (free) |
| **OpenAPI** | 3.1.0 | spec.openapis.org (free) |
| **AsyncAPI** | 3.0.0 | asyncapi.com (free) |
| **JSON Schema** | 2020-12 | json-schema.org (free) |
| **HL7 v2** | 2.9 | hl7.org (free) |

---

## 11. Reference health systems

These open-source health systems are studied for patterns, integration, and inspiration. They are not replacements for ZarishSphere components.

| System | License | What we learn |
|--------|---------|--------------|
| **OpenMRS** | MPL-2.0 | Concept dictionary, form engine, module system |
| **Bahmni** | AGPL-3.0 | Offline-first EMR, lab integration patterns |
| **DHIS2** | BSD-3 | Aggregate reporting, tracker patterns, metadata |
| **Go.Data** | MIT | Contact tracing, outbreak investigation workflows |
| **OpenIMIS** | LGPL-3.0 | Health insurance, claims, beneficiary management |
| **GNU Health** | GPL-3.0 | Hospital information system patterns |
| **Medplum** | Apache 2.0 | Modern FHIR-native EHR architecture |
| **LibreHealth** | MPL-2.0 | Modular EHR approach |
| **OpenEMR** | GPL-3.0 | Long-running EMR, multilingual support |

---

## 12. Developer experience tools

| Tool | License | Purpose |
|------|---------|---------|
| **GitHub Codespaces** | Free 60 hrs/month | Cloud development environment |
| **devcontainers** | MIT | Reproducible development environments |
| **Bruno** | MIT | API client (Git-friendly, OSS Postman alternative) |
| **httpie** | BSD-3 | CLI HTTP client |
| **jq** | MIT | JSON processor |
| **yq** | MIT | YAML processor |
| **mkcert** | ISC | Local HTTPS certificates |
| **Telepresence** | Apache 2.0 | Local development to remote K8s cluster |
| **Skaffold** | Apache 2.0 | Kubernetes development workflow |
| **Tilt** | Apache 2.0 | Alternative Kubernetes development workflow |
| **pre-commit** | MIT | Git pre-commit hooks |
| **commitlint** | MIT | Conventional commit enforcement |
| **husky** | MIT | Git hooks manager |
| **lefthook** | MIT | Alternative Git hooks (faster) |

---

## 13. Documentation tools

| Tool | License | Purpose |
|------|---------|---------|
| **Docusaurus** | MIT | Documentation site (docs.zarishsphere.com) |
| **Redoc** | MIT | OpenAPI 3.1 API reference rendering |
| **Asyncapi-react** | Apache 2.0 | AsyncAPI event catalog rendering |
| **Mermaid** | MIT | Diagrams as code (GitHub native) |
| **Excalidraw** | MIT | Collaborative whiteboarding |
| **markdownlint** | MIT | Markdown linting |
| **Vale** | MIT | Prose style linter |
| **mdbook** | MPL-2.0 | Alternative documentation builder |

---

## 14. Free-tier cloud services

### 14.1 GitHub (core platform)

| Feature | Free tier | Notes |
|---------|-----------|-------|
| Public repositories | Unlimited | All ZS repos are public |
| GitHub Actions | 2000 min/month | CI/CD backbone |
| GitHub Pages | Free for public repos | Static site hosting |
| GHCR | Free for public images | Container registry |
| GitHub Discussions | Free | RFC + community forum |
| GitHub Projects | Free | Kanban, roadmap |
| GitHub Codespaces | 60 hrs/month | Cloud development environment |
| Dependabot | Free | Vulnerability alerts |
| CodeQL | Free for public repos | SAST security scanning |

### 14.2 Cloudflare (edge layer)

| Feature | Free tier | Notes |
|---------|-----------|-------|
| DNS | Unlimited | zarishsphere.com + all subdomains |
| CDN | Unlimited bandwidth | Static asset delivery |
| SSL/TLS | Free auto-renew | All subdomains |
| Cloudflare Workers | 100000 req/day | Edge functions (auth, routing) |
| Cloudflare Pages | Unlimited deployments | Frontend hosting |
| Cloudflare R2 | 10 GB storage, zero egress | Backups, documents, reports |
| Email Routing | Free | admin@zarishsphere.com routing |
| Cloudflare Tunnel | Free | Expose local services securely |

### 14.3 Hosting (compute)

| Provider | Free tier | Use case |
|----------|-----------|----------|
| **Cloudflare Pages** | Unlimited deployments | Frontend hosting (primary) |
| **GitHub Pages** | Free for public repos | Static documentation sites |
| **Render** | 750 hrs/month | Backend services (dev/staging) |
| **Fly.io** | 3 shared VMs, 3 GB RAM | Microservices (dev/staging) |
| **Railway** | $5 credit/month | Databases (development) |
| **Oracle Cloud Free** | 2 ARM VMs (always free), 24 GB RAM | Production self-hosted K8s |
| **Google Cloud Free** | f1-micro VM, 30 GB HDD | Additional compute |

> **Note:** Frontend hosting is Cloudflare Pages + GitHub Pages only. Vercel is not used anywhere in the ecosystem (see → **007-zs-tech-stack/003-frontend-stack.md**).

### 14.4 Search and communications

| Provider | Free tier | Use case |
|----------|-----------|----------|
| **Typesense Cloud** | 10k documents | Development search |
| **Algolia** | 10k search ops/month | Alternative search |
| **Resend** | 3000 emails/month | Transactional email |
| **Brevo (Sendinblue)** | 300 emails/day | Email notifications |

### 14.5 Monitoring and security

| Provider | Free tier | Use case |
|----------|-----------|----------|
| **Sentry** | 5k errors/month (OSS) | Error tracking |
| **Codecov** | Free for OSS | Code coverage |
| **Sonar Cloud** | Free for OSS | Code quality |
| **Snyk** | Free for OSS (200 tests/month) | Dependency scanning |
| **GitGuardian** | Free for OSS | Secret scanning |
| **CodeRabbit** | Free for OSS | AI code review |

---

## 15. License summary

This table consolidates every open-source license used across all tools in the catalog.

| License | Tools using this license |
|---------|------------------------|
| **MIT** | Chi, pgx, squirrel, testify, zerolog, React, Next.js, Tailwind CSS, Tiptap, Radix UI, React Hook Form, Zod, TanStack Query, TanStack Table, workbox, Recharts, i18next, react-i18next, Vitest, @testing-library/react, MSW, Storybook, Riverpod, drift, dio, fhir, freezed, Tauri plugins, Traefik, act, Uptime Kuma, viper, samber/lo, air, goreleaser, pre-commit, commitlint, husky, lefthook, Bruno, Docusaurus, Redoc, Mermaid, Excalidraw, markdownlint, Vale, jq, yq, mkcert, devcontainers, Terragrunt, Flux CD, gubernator |
| **Apache 2.0** | gofhir-models, golang-fhir-models, valkey-go, nats.go, go-oidc, OpenTelemetry, Prometheus client, cobra, Carbon Design System, @rjsf/core, Dexie.js, Apache ECharts, Playwright, PowerSync, Docker, Docker Compose, Kubernetes, Helm, kubectl, k9s, Pulumi, Crossplane, Argo CD, Argo Rollouts, Cilium, cert-manager, External Secrets Operator, TIMESCALEDB, NATS, Telepresence, Skaffold, Tilt, asynq, SUSHI, Inferno, Synthea, HAPI FHIR Validator CLI, Medplum, Asyncapi-react, Grafana Alloy |
| **BSD-3** | Go, gorilla/websocket, golang.org/x/crypto, google/uuid, Flutter, Dart, go_router, flutter_secure_storage, flutter_localizations, httpie, DHIS2 |
| **BSD-2** | go-redis, OpenLayers |
| **GPL-3** | golangci-lint, Typesense, GNU Health, OpenEMR |
| **LGPL-3** | OpenIMIS |
| **AGPL-3** | Grafana, Loki, Tempo, Bahmni |
| **MPL-2.0** | OpenTofu, OpenMRS, LibreHealth, OpenConceptLab, axe-core, mdbook, OpenBao |
| **BSL 1.1 (source-available)** | HashiCorp Vault |
| **ISC** | idb, mkcert |
| **Expat** | react-leaflet |
| **CC0** | FHIR Shorthand, IG Publisher |
| **Public domain** | SQLite |
| **Proprietary (free tier)** | GitHub Actions, GitHub Codespaces, Cloudflare services, Render, Fly.io, Railway, Neon, Supabase, Upstash, PlanetScale, Typesense Cloud, Algolia, Resend, Brevo, Sentry, Codecov, Sonar Cloud, Snyk, GitGuardian, CodeRabbit, Oracle Cloud Free Tier |

---

## 16. Zero-cost verification

Every tool listed in this catalog satisfies the zero-cost requirement of ADR-006. The verification matrix below confirms the cost model for each category.

| Category | Cost model | Annual cost |
|----------|-----------|-------------|
| Backend (Go) | OSS — no licensing cost | $0 |
| Frontend (React/TypeScript) | OSS — no licensing cost | $0 |
| Mobile (Flutter/Dart) | OSS — no licensing cost | $0 |
| Desktop (Tauri/Rust) | OSS — no licensing cost | $0 |
| Infrastructure (Docker, K8s, OpenTofu) | OSS — no licensing cost | $0 |
| Observability (Prometheus, Grafana) | OSS — no licensing cost | $0 |
| Databases (SQLite, PostgreSQL, NATS) | OSS — no licensing cost | $0 |
| Security (Keycloak, Vault ESO) | OSS — no licensing cost | $0 |
| Clinical terminologies (ICD-11, LOINC) | Free access | $0 |
| CI/CD (GitHub Actions) | Free tier — 2000 min/month | $0 |
| Edge services (Cloudflare) | Free tier — 100k req/day, 10 GB storage | $0 |
| Development hosting (Render, Fly.io) | Free tier for OSS | $0 |
| **Total infrastructure cost** | | **$0** |

> **Constraint:** No toolchain component may require a credit card to start. Every free tier must have sufficient capacity for Plane 0 and Plane 1 deployments at full production load.

> **Constraint:** If a free-tier service changes its pricing model and becomes paid, a migration path to an alternative zero-cost tool must be documented within 90 days.

---

## 17. Cross-references

→ **007-zs-tech-stack/001-tech-stack-master.md** — Master tech stack mapping, maintained in alignment with this catalog
→ **008-zs-adrs/013-adr-postgresql-primary-database.md** — ADR-013: PostgreSQL 18.4 primary database
→ **008-zs-adrs/014-adr-nats-jetstream-messaging.md** — ADR-014: NATS JetStream messaging backbone
→ **008-zs-adrs/019-adr-carbon-design-system.md** — ADR-019: Carbon Design System as primary UI library
→ **008-zs-adrs/023-adr-flutter-cross-platform-mobile.md** — ADR-023: Flutter cross-platform mobile
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: zero-cost toolchain
→ **003-zs-platform/001-platform-overview.md** — Platform architecture, deployment planes, and system boundaries
→ **009-zs-operations/010-sop-incident-response.md** — Incident response SOP (tooling outage handling)
→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS documentation and formatting rules

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
