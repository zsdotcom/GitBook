# 010-free-tier-resource-map.md
## Free-tier resource map
### Every zero-cost service available for ZarishSphere deployment

**Document type:** Reference
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Guiding principles](#2-guiding-principles)
3. [GitHub — core platform](#3-github--core-platform)
4. [Cloudflare — edge layer](#4-cloudflare--edge-layer)
5. [Compute and hosting](#5-compute-and-hosting)
6. [Databases and storage](#6-databases-and-storage)
7. [Search, messaging and communications](#7-search-messaging-and-communications)
8. [CI/CD and automation](#8-cicd-and-automation)
9. [Observability and monitoring](#9-observability-and-monitoring)
10. [Security and scanning](#10-security-and-scanning)
11. [Domain and DNS](#11-domain-and-dns)
12. [Health data standards — free sources](#12-health-data-standards--free-sources)
13. [Terminology sources — free access](#13-terminology-sources--free-access)
14. [Developer experience tools](#14-developer-experience-tools)
15. [Documentation tools](#15-documentation-tools)
16. [Architectural constraints](#16-architectural-constraints)
17. [Cross-references](#17-cross-references)

---

## 1. Purpose

### 1.1 Why this map exists

ADR-006 (Zero-Cost Toolchain) mandates that the entire ZarishSphere ecosystem must be deployable using only open-source and free-tier services. This map documents every free resource available, its capacity limits, and how it fits into the platform architecture.

### 1.2 Who it serves

| Audience | How they use this map |
|---|---|
| **Architects** | Verify that platform design stays within free-tier constraints |
| **Deployers** | Identify which services to sign up for when deploying a new instance |
| **Developers** | Know the limits of each free tier to design within them |
| **Contributors** | Understand the zero-cost commitment and avoid paid-dependency proposals |

### 1.3 Scope

This document covers:

- Free tiers of commercial cloud services
- Open-source software that can be self-hosted at zero licensing cost
- Free-access health data and terminology sources
- Developer and documentation tools available at no cost

It does not cover:

- Paid tiers of any service
- Hardware costs (devices, servers, networking equipment)
- Staff time and labour
- Domain registration fees (zarishsphere.com and country domains)

---

## 2. Guiding principles

| Principle | Application |
|---|---|
| **Free tier must suffice** | Platform features must never require a paid upgrade |
| **Self-hosted is preferred** | Open-source self-hosting avoids vendor free-tier limits entirely |
| **OSS licensing required** | Every dependency must be Apache 2.0, MIT, BSD, MPL-2.0, or similar |
| **No `latest` tag** | All versions pinned explicitly in configuration |
| **Free for dev and prod** | Free tiers must cover both development and production use |

> **Constraint:** ADR-006 applies to the entire platform. Any proposal adding a paid dependency requires an ADR override approved by the Foundation.

---

## 3. GitHub — core platform

GitHub serves as the ZarishSphere control plane (ADR-003). All free-tier features are sufficient for the ecosystem's operating model.

| Feature | Free tier | Limit | Notes |
|---|---|---|---|
| Public repositories | Unlimited | — | All ZarishSphere repositories are public |
| Private repositories | Unlimited | — | For sensitive config, limited use |
| GitHub Actions | 2,000 min/month | Shared across org | CI/CD backbone for all repos |
| GitHub Pages | Free | Unlimited bandwidth | Static site hosting for docs |
| GHCR (container registry) | Free | Unlimited storage | Public container images only |
| GitHub Discussions | Free | — | RFCs and community forum |
| GitHub Projects | Free | — | Kanban boards and roadmaps |
| GitHub Codespaces | 60 hrs/month | Per user | Cloud development environments |
| Dependabot | Free | — | Automated dependency updates |
| CodeQL | Free | Public repos only | SAST security scanning |
| GitHub Wiki | Free | — | Supplementary documentation |

> **Constraint:** The 2,000-minute/month Actions limit applies to the entire org. Optimise workflow concurrency and reuse via `zs-ops-github-actions` shared workflows.

---

## 4. Cloudflare — edge layer

Cloudflare provides the edge network for all ZarishSphere web properties (ADR-002).

| Feature | Free tier | Limit | Notes |
|---|---|---|---|
| DNS | Unlimited | — | zarishsphere.com + all subdomains |
| CDN | Unlimited bandwidth | — | Static asset delivery |
| SSL/TLS certificates | Free | Auto-renew | All subdomains covered |
| Cloudflare Workers | 100,000 req/day | 10 ms CPU per request | Edge authentication, routing |
| Cloudflare Pages | Unlimited deployments | 500 builds/month | Frontend hosting |
| Cloudflare R2 | 10 GB storage | 0 egress fees | Backups, documents, media |
| Email Routing | Free | — | admin@ routing |
| Cloudflare Tunnel | Free | — | Expose local Plane 0/1 services |

> **Constraint:** Cloudflare Workers have a 10 ms CPU time limit per request. Heavy computation must not run at the edge. Use Workers only for auth validation, header rewriting, and routing logic.

---

## 5. Compute and hosting

### 5.1 Serverless and platform hosting

| Provider | Free tier | Use case | Limitations |
|---|---|---|---|
| **Vercel** | Unlimited for OSS projects | Frontend (Next.js) deployment | 100 GB bandwidth/month, 60 sec function timeout |
| **Render** | 750 hrs/month | Backend services (dev/staging) | 512 MB RAM, sleeps after 15 min inactivity |
| **Fly.io** | 3 shared VMs, 3 GB total RAM | Microservices (dev/staging) | 3 VM limit, shared CPU |
| **Railway** | $5 credit/month | Databases and services (dev) | Credit-based, not purely free |
| **Netlify** | 100 GB bandwidth, 300 min build | Static site hosting | 300 build minutes/month |

### 5.2 Cloud VMs (always free)

| Provider | Free tier | Use case | Limitations |
|---|---|---|---|
| **Oracle Cloud Free** | 2 x ARM VMs (4 OCPU, 24 GB RAM) | Production self-hosted K8s | ARM only, fixed shape, limited regions |
| **Google Cloud Free** | 1 x f1-micro VM (0.2 vCPU, 0.6 GB) | Lightweight compute | Minimal capacity, eGPU only in us-* regions |
| **AWS Free Tier** | 750 hrs/month (t2.micro, 1 GB RAM) | Dev/test for 12 months | Time-limited (12 months) |

> **Constraint:** Oracle Cloud's always-free ARM VMs (24 GB RAM) are the only viable free-tier production compute. Platform architecture must target ARM64 as the primary architecture.

### 5.3 Edge compute

| Provider | Free tier | Use case |
|---|---|---|
| **Cloudflare Workers** | 100,000 req/day | Edge functions, auth, routing |
| **Deno Deploy** | 100,000 req/day | Edge functions (alternative) |
| **Vercel Edge Functions** | Included in OSS tier | Next.js edge middleware |

---

## 6. Databases and storage

### 6.1 Managed databases (free tier)

| Provider | Free tier | Use case | Limitations |
|---|---|---|---|
| **Neon** | PostgreSQL 512 MB, compute hours | Development databases | Compute hours limit, auto-suspend |
| **Supabase** | PostgreSQL 500 MB, 1 GB storage | Development/prototype | 500 MB database limit |
| **Upstash** | Redis 10,000 cmd/day, Kafka 10,000 msg/day | Development caching | Very low throughput |
| **PlanetScale** | MySQL 1 GB storage (limited) | Not primary (fallback only) | MySQL only, not FHIR-native |
| **MongoDB Atlas** | 512 MB storage | Not primary (document store) | M0 shared tier, 500 collections |

### 6.2 Self-hosted databases (open source, unlimited)

| System | License | Use case | Plane 0 compatible |
|---|---|---|---|
| **PostgreSQL 18.4** | PostgreSQL | Primary database — FHIR JSONB, GIN indexes | Yes |
| **TimescaleDB** | Apache 2.0 (free) | Time-series — vitals, event trends | Yes |
| **Valkey 9.1.1** | BSD-3 | Cache and session store | Yes |
| **NATS 2.14.4** | Apache 2.0 | Message broker, JetStream persistence | Yes |
| **Typesense 28.0** | GPL-3 | Typo-tolerant search | Yes |
| **SQLite** | Public domain | Embedded database for offline/edge | Yes |
| **MinIO** | AGPL-3 | S3-compatible object storage | Yes |

### 6.3 Object storage (free tier)

| Provider | Free tier | Use case |
|---|---|---|
| **Cloudflare R2** | 10 GB storage, unlimited egress | Backups, media, reports |
| **MinIO (self-hosted)** | Unlimited | S3-compatible, self-hosted on ARM64 |
| **Backblaze B2** | 10 GB free | Backup storage |

> **Constraint:** Self-hosted databases are strongly preferred. Managed free tiers are suitable for development and staging only — never for production workloads.

---

## 7. Search, messaging and communications

### 7.1 Search

| Provider | Free tier | Use case |
|---|---|---|
| **Typesense Cloud** | 10,000 documents | Development search |
| **Algolia** | 10,000 search ops/month | Alternative search (not primary) |
| **Meilisearch Cloud** | 100,000 documents | Alternative search (not primary) |

Self-hosted Typesense is preferred and has no document limit.

### 7.2 Email

| Provider | Free tier | Use case |
|---|---|---|
| **Resend** | 3,000 emails/month | Transactional email |
| **Brevo (Sendinblue)** | 300 emails/day | Email notifications |
| **Cloudflare Email Routing** | Free | Forwarding admin@ to personal inbox |
| **SMTP2GO** | 1,000 emails/month | Alternative transactional email |

### 7.3 SMS and notifications

| Provider | Free tier | Use case |
|---|---|---|
| **Twilio** | Trial credits ($15) | SMS testing (one-time) |
| **Africa's Talking** | Free tier for testing | SMS in African deployments |

> **Constraint:** SMS and email are best handled by self-hosted solution or pay-as-you-go. No service offers unlimited free SMS. Budget for SMS costs in production deployments.

---

## 8. CI/CD and automation

| Tool | License / free tier | Use case |
|---|---|---|
| **GitHub Actions** | 2,000 min/month (free) | Primary CI/CD pipeline |
| **Argo CD** | Apache 2.0 (self-hosted) | GitOps deployment |
| **Renovate** | Free for OSS | Automated dependency updates |
| **pre-commit** | MIT | Git pre-commit hooks |
| **commitlint** | MIT | Conventional commit enforcement |
| **husky** | MIT | Git hooks manager (Node.js) |
| **lefthook** | MIT | Git hooks manager (Go-native, faster) |
| **act** | MIT | Run GitHub Actions locally for testing |

> **Constraint:** The 2,000-minute/month GitHub Actions limit is the primary CI/CD constraint. Use `act` for local testing before pushing to conserve minutes.

---

## 9. Observability and monitoring

### 9.1 Self-hosted (unlimited, zero cost)

| Tool | License | Use case |
|---|---|---|
| **Prometheus 3.13.2** | Apache 2.0 | Metrics collection |
| **Grafana 13.1.1** | AGPL-3 | Dashboards and alerting |
| **Loki 3.4** | AGPL-3 | Log aggregation |
| **Tempo 2.7** | AGPL-3 | Distributed tracing |
| **Alertmanager** | Apache 2.0 | Alert routing |
| **OpenTelemetry Collector** | Apache 2.0 | Telemetry pipeline |
| **Grafana Alloy** | Apache 2.0 | Unified telemetry agent |
| **Uptime Kuma** | MIT | Self-hosted uptime monitoring |

### 9.2 Managed (free tier)

| Provider | Free tier | Use case |
|---|---|---|
| **Grafana Cloud Free** | 14 days retention, 3 users | Managed observability |
| **Sentry** | 5,000 errors/month (OSS) | Error tracking |
| **Better Stack** | 3 users, 1 status page | Uptime monitoring |
| **Checkly** | 5 checks/5 min intervals | Synthetic monitoring |

> **Constraint:** Self-hosted observability is preferred for Plane 0-3 deployments. Grafana Cloud free tier is suitable for Plane 4 (global SaaS) dev/staging.

---

## 10. Security and scanning

| Tool | License / free tier | Use case |
|---|---|---|
| **CodeQL** | Free for public repos | SAST scanning |
| **Trivy** | Apache 2.0 (self-hosted) | Container and filesystem scanning |
| **GitGuardian** | Free for OSS | Secret scanning |
| **Snyk** | Free for OSS (200 tests/month) | Dependency vulnerability scanning |
| **SonarCloud** | Free for OSS | Code quality and security |
| **CodeRabbit** | Free for OSS | AI code review |
| **Dependabot** | Free | Dependency update automation |
| **keycloak** | Apache 2.0 (self-hosted) | IAM, OIDC, SMART on FHIR |
| **HashiCorp Vault** | BSL (free for OSS) | Secrets management |

---

## 11. Domain and DNS

| Service | Free tier | Use case |
|---|---|---|
| **Cloudflare DNS** | Unlimited | All domain DNS management |
| **Cloudflare SSL/TLS** | Free, auto-renew | TLS certificates for all subdomains |
| **Cloudflare Tunnel** | Free | Expose local Plane 0/1 services without public IP |
| **Let's Encrypt** | Free | Alternative TLS certificates (via cert-manager) |

> **Constraint:** Domain registration fees (zarishsphere.com and country-specific domains) are the only unavoidable cost. DNS, SSL, and tunnelling are all free through Cloudflare.

---

## 12. Health data standards — free sources

| Standard | Version | Source | Access | License |
|---|---|---|---|---|
| FHIR R5 | 5.0.0 | hl7.org/fhir/R5 | Web + download | CC0 (specification) |
| SMART App Launch | 2.2.0 | smarthealthit.org | Web | Free |
| CDS Hooks | 2.0 | cds-hooks.org | Web | Free |
| OpenAPI | 3.1 | spec.openapis.org | Web | Free |
| AsyncAPI | 3.0 | asyncapi.com | Web | Free |
| JSON Schema | 2020-12 | json-schema.org | Web | Free (CC0) |
| HL7 v2 | 2.9 | hl7.org | Web | Free |

### 12.1 FHIR and interoperability tools (free)

| Tool | License | Use case |
|---|---|---|
| **FHIR Shorthand (FSH)** | CC0 | Author FHIR profiles and IGs |
| **SUSHI** | Apache 2.0 | FSH compiler to FHIR IG |
| **IG Publisher** | CC0 | Publish FHIR Implementation Guides |
| **Inferno** | Apache 2.0 | FHIR compliance testing |
| **Synthea** | Apache 2.0 | Synthetic patient test data generation |
| **OpenConceptLab** | MPL-2.0 | Terminology hosting (cloud free tier) |
| **Simplifier.net** | Free tier | FHIR profile registry |

---

## 13. Terminology sources — free access

| Terminology | Source | License | Access method | Update cadence |
|---|---|---|---|---|
| **ICD-11** | WHO | Free | REST API + bulk download | Annual (January) |
| **SNOMED CT** | SNOMED International | Free for DPG/OSS | RF2 bulk download | Semi-annual (July/January) |
| **LOINC** | Regenstrief Institute | Free | CSV download | Semi-annual |
| **CIEL** | OpenMRS / Andrew Kanter | Free | OpenMRS wiki download | Quarterly |
| **RxNorm** | NLM (US) | Free | REST API (rxnav.nlm.nih.gov) | Monthly |
| **CVX (vaccines)** | CDC | Free | Flat file download | As needed |
| **ICD-10** | WHO | Free (legacy) | Bulk download | Static |
| **ATC** | WHO | Free | Bulk download | Annual |

> **Constraint:** SNOMED CT free access requires registering as a Developer/OSS project with SNOMED International. This must be maintained annually.

---

## 14. Developer experience tools

| Tool | License | Use case |
|---|---|---|
| **GitHub Codespaces** | 60 hrs/month free | Cloud-based development |
| **devcontainers** | MIT | Reproducible development environments |
| **Bruno** | MIT | API client (OSS Postman alternative) |
| **httpie** | BSD-3 | CLI HTTP client |
| **jq** | MIT | JSON processor |
| **yq** | MIT | YAML processor |
| **mkcert** | ISC | Local HTTPS certificates |
| **Telepresence** | Apache 2.0 | Local development to remote K8s |
| **Skaffold** | Apache 2.0 | K8s development workflow |
| **Tilt** | Apache 2.0 | Alternative K8s development workflow |

---

## 15. Documentation tools

| Tool | License | Use case |
|---|---|---|
| **Docusaurus** | MIT | Documentation site generation |
| **Redoc** | MIT | OpenAPI 3.1 API reference rendering |
| **Asyncapi-react** | Apache 2.0 | AsyncAPI event catalog rendering |
| **Mermaid** | MIT | Diagrams as code (GitHub native) |
| **Excalidraw** | MIT | Collaborative whiteboarding |
| **markdownlint** | MIT | Markdown linting |
| **Vale** | MIT | Prose style linter |
| **mdbook** | MPL-2.0 | Alternative documentation builder |

---

## 16. Architectural constraints

These constraints arise from free-tier limits and directly affect platform architecture.

### 16.1 ARM64 primary architecture

> **Constraint:** Oracle Cloud Free (the only viable always-free production compute) provides ARM64 instances. All Go binaries, containers, and infrastructure must target ARM64. x86 builds remain available for local development.

### 16.2 GitHub Actions budget

> **Constraint:** 2,000 minutes/month across the entire org. CI/CD pipelines must be optimised for speed and reuse. Shared workflows (`zs-ops-github-actions`) prevent duplication. Local testing with `act` reduces push frequency.

### 16.3 Cloudflare Workers CPU limit

> **Constraint:** 10 ms CPU time per request. Edge functions must be lightweight — auth validation, header rewriting, routing. Heavy computation must happen in backend services, not at the edge.

### 16.4 Self-hosted over managed

> **Constraint:** Self-hosted open-source infrastructure (PostgreSQL, Valkey, NATS, MinIO, Prometheus, Grafana) is preferred over managed free tiers. Managed free tiers are acceptable for development and staging but not for production.

### 16.5 R2 storage for backups

> **Constraint:** Cloudflare R2 provides 10 GB with free egress. This is sufficient for configuration backups, small-media storage, and report archives. Large-media storage requires MinIO self-hosting.

### 16.6 No paid API tiers

> **Constraint:** The platform must never require a paid API key. All API integrations must work within free tiers or use self-hosted alternatives.

---

## 17. Cross-references

→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: the zero-cost toolchain mandate this map implements
→ **008-zs-adrs/002-adr-cloudflare-as-edge-platform.md** — ADR-002: Cloudflare as the edge platform (Sections 4 and 11)
→ **008-zs-adrs/003-adr-github-as-government.md** — ADR-003: GitHub as the control plane (Section 3)
→ **007-zs-tech-stack/001-tech-stack-master.md** — Version-pinned inventory of the complete technology stack
→ **007-zs-tech-stack/006-oss-tool-catalog.md** — Open-source tool catalog and licenses
→ **003-zs-platform/003-deployment-planes.md** — Plane-by-plane infrastructure requirements these free tiers serve
→ **003-zs-platform/009-repository-catalog.md** — Repositories that consume the free-tier services listed here

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
