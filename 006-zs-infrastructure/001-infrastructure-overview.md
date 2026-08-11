# 001-infrastructure-overview.md
## Enterprise infrastructure and zero-touch deployment mapping
### All infrastructure layers, design principles, and deployment automation

**Document type:** Architecture
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Draft

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Infrastructure design principles](#2-infrastructure-design-principles)
3. [Layer architecture](#3-layer-architecture)
4. [Infrastructure as Code approach](#4-infrastructure-as-code-approach)
5. [GitHub as government](#5-github-as-government)
6. [Cloudflare services summary](#6-cloudflare-services-summary)
7. [Plane 0 and infrastructure choices](#7-plane-0-and-infrastructure-choices)
8. [Document map](#8-document-map)
9. [Cross-references](#9-cross-references)

---

## 1. Purpose

This document provides the top-level map of all ZarishSphere infrastructure. Every component described here has a dedicated document in this folder with full configuration details. Use this overview to understand how the pieces fit together before reading individual infrastructure specs.

The entire ZarishSphere infrastructure runs on two providers — GitHub and Cloudflare — both on free tiers. No paid services are required for any deployment plane. The infrastructure is designed so that Plane 0 (air-gapped) deployments can operate with zero cloud dependencies while sharing the same codebase and toolchain as the global SaaS tier.

---

## 2. Infrastructure design principles

### 2.1 Zero-cost

Every infrastructure component operates within GitHub and Cloudflare free-tier limits. No monthly bill is required to run the ZarishSphere ecosystem.

> **Constraint:** No infrastructure component may require a paid tier on any provider. If a free-tier component reaches its limit (e.g., Cloudflare Workers 100,000 requests/day), the system must degrade gracefully rather than fail or require payment.

### 2.2 Zero-touch

Infrastructure provisioning is automated through GitHub Actions and Cloudflare APIs. A push to the default branch triggers validation, build, and deployment without human intervention.

### 2.3 Offline-first (Plane 0 compatible)

All infrastructure decisions respect Plane 0. Cloud services are enhancements, not requirements. The system must function identically when cloud connectivity is absent:

| Cloud service | Plane 0 equivalent |
|---|---|
| Cloudflare CDN | Local static file serving |
| Cloudflare DNS | Local hosts file or mDNS |
| Cloudflare Workers | Go-native API binary |
| Cloudflare Email Routing | Local SMTP relay or queue |
| Cloudflare Pages | Local web server (Caddy or NGINX) |
| GitHub (source of truth) | USB bundle with git bundle |
| GitHub Actions | Local shell scripts |

### 2.4 Documentation-first

Every infrastructure configuration is documented in this folder before being applied to any provider. The documentation is the source of truth — the Cloudflare Dashboard and GitHub settings mirror what is written here.

### 2.5 Auditability

All infrastructure changes flow through GitHub pull requests. Every change to DNS records, WAF rules, or deployment configuration is tracked, reviewed, and versioned.

---

## 3. Layer architecture

The infrastructure stack has five layers, each documented in its own file:

```
┌──────────────────────────────────────────────────────────────┐
│                    LAYER 5: CI/CD                            │
│  GitHub Actions — test, build, deploy, validate             │
│  006-ci-cd-architecture.md                                  │
├──────────────────────────────────────────────────────────────┤
│                    LAYER 4: Email                            │
│  Cloudflare Email Routing — receive, forward, security      │
│  005-email-architecture.md                                  │
├──────────────────────────────────────────────────────────────┤
│                    LAYER 3: Domains                          │
│  zarishsphere.com + subdomains — DNS records, routing       │
│  004-domain-architecture.md                                 │
├──────────────────────────────────────────────────────────────┤
│                    LAYER 2: Edge                             │
│  Cloudflare — CDN, WAF, Pages, Workers, R2, SSL            │
│  003-cloudflare-architecture.md                             │
├──────────────────────────────────────────────────────────────┤
│                    LAYER 1: Source of truth                  │
│  GitHub — repos, org, teams, branches, secrets, pages      │
│  002-github-org-architecture.md                             │
└──────────────────────────────────────────────────────────────┘
```

### 3.1 Data flow between layers

```
GitHub (source of truth)
  │  Push to main triggers:
  ▼
GitHub Actions (CI/CD)
  │  Builds and deploys:
  ▼
Cloudflare Pages / Workers (edge)
  │  Serves content via:
  ▼
Cloudflare DNS + CDN (domains)
  │  Email routing:
  ▼
Cloudflare Email Routing (email)
```

---

## 4. Infrastructure as Code approach

ZarishSphere manages all infrastructure as documented configuration. The practice is:

| Concern | Managed as | Tool |
|---|---|---|
| Source code and docs | Git repositories | GitHub |
| CI/CD pipelines | YAML workflow files | GitHub Actions |
| DNS records | Cloudflare API / Dashboard | Cloudflare |
| WAF rules | Cloudflare Dashboard | Cloudflare |
| Static site hosting | Cloudflare Pages connected to GitHub | Cloudflare |
| API routing | Cloudflare Workers | Cloudflare |
| Object storage | Cloudflare R2 | Cloudflare |
| Email routing | Cloudflare Email Routing rules | Cloudflare |
| SSL/TLS certificates | Cloudflare Universal SSL (auto) | Cloudflare |

> **Constraint:** No configuration may be applied through a provider's GUI unless it is also documented in the corresponding file in `006-zs-infrastructure/`. The documentation is the authoritative specification.

---

## 5. GitHub as government

The ZarishSphere Foundation operates on a GitHub-as-government model (ADR-003). In this model:

- **GitHub is the control plane.** Every decision, every policy, every configuration change flows through GitHub. There is no external decision system.
- **Pull requests are legislation.** A PR that changes a governance document, infrastructure config, or ADR is the equivalent of a legislative act. It must be reviewed and merged before taking effect.
- **The `main` branch is law.** Whatever is on `main` is the current state of the ecosystem. There is no staging environment for governance.
- **Git history is the legal record.** Every change is timestamped, attributed, and reversible.

This model directly affects infrastructure:
- GitHub repository settings, branch protection, and secrets are the enforcement layer.
- GitHub Actions runs the CI/CD pipeline that deploys infrastructure.
- GitHub Pages hosts the published documentation that describes the infrastructure.
- No external tool is used for governance or infrastructure management outside of GitHub and Cloudflare.

---

## 6. Cloudflare services summary

All Cloudflare services run on the free plan. The following services form the ZarishSphere edge infrastructure:

| Service | Purpose | Free tier limit |
|---|---|---|
| **CDN** | Global content delivery, caching, DDoS protection | Unlimited bandwidth |
| **WAF** | Web application firewall — OWASP rules, rate limiting | Basic rules free |
| **DNS** | Authoritative DNS for zarishsphere.com and subdomains | Unlimited records |
| **SSL/TLS** | Universal SSL certificates (auto-renewed) | Unlimited domains |
| **Pages** | Static site hosting for console, marketplace, docs | 500 builds/month |
| **Workers** | Serverless API routing and backend logic | 100,000 req/day |
| **Email Routing** | Receive and forward email for all @zarishsphere.com | Unlimited inbound-only aliases |
| **R2** | Object storage for standards index artifacts | 10 GB storage |
| **DDoS** | Layer 3/4/7 DDoS mitigation | Included |
| **Analytics** | Web analytics (privacy-first) | Free tier |

---

## 7. Plane 0 and infrastructure choices

Plane 0 (air-gapped) is the forcing function for every infrastructure decision. The ZarishSphere Platform must function on a 4 GB device with zero internet connectivity. This constraint shapes infrastructure in specific ways:

### 7.1 What changes at Plane 0

| Infrastructure component | Plane 0 behavior |
|---|---|
| Source of truth | USB bundle containing a `git bundle` of all repos |
| CI/CD | Manual execution of local validation scripts |
| DNS | Local hosts mapping or mDNS |
| CDN | No CDN — static files served by local HTTP server |
| SSL/TLS | Self-signed certificates for local-only TLS |
| Email | No email — messages queued for sync |
| Cloudflare Workers | Go-native binary replaces Worker routing |
| WAF | No WAF — machine is air-gapped |
| R2 storage | Local file system storage |

### 7.2 Branch strategy for air-gapped updates

Plane 0 deployments use a dedicated `release/plane-0` branch that contains a snapshot of all artifacts needed for offline operation. Updates are delivered via USB bundle containing `git bundle` files that can be applied without internet access.

---

## 8. Document map

This folder contains ten documents that together form the complete infrastructure specification:

| # | Document | Layer | Purpose |
|---|---|---|---|
| 001 | This document | All layers | Overview, design principles, document map |
| 002 | [002-github-org-architecture.md](006-zs-infrastructure/002-github-org-architecture.md) | Layer 1 — Source of truth | Repos, teams, branches, secrets |
| 003 | [003-cloudflare-architecture.md](006-zs-infrastructure/003-cloudflare-architecture.md) | Layer 2 — Edge | CDN, WAF, Pages, Workers, R2, SSL |
| 004 | [004-domain-architecture.md](006-zs-infrastructure/004-domain-architecture.md) | Layer 3 — Domains | DNS zones, subdomains, routing |
| 005 | [005-email-architecture.md](006-zs-infrastructure/005-email-architecture.md) | Layer 4 — Email | Email routing, SPF/DKIM/DMARC |
| 006 | [006-ci-cd-architecture.md](006-zs-infrastructure/006-ci-cd-architecture.md) | Layer 5 — CI/CD | GitHub Actions, pipelines, environments |
| 007 | [007-security-architecture.md](006-zs-infrastructure/007-security-architecture.md) | Cross-cutting | Defense-in-depth security architecture |
| 008 | [008-security-policies.md](006-zs-infrastructure/008-security-policies.md) | Cross-cutting | Security policies (access control, encryption, secrets) |
| 009 | [009-compliance-controls.md](006-zs-infrastructure/009-compliance-controls.md) | Cross-cutting | Compliance controls (HIPAA, GDPR, data residency) |
| 010 | [010-threat-models.md](006-zs-infrastructure/010-threat-models.md) | Cross-cutting | Threat models (API, auth, data, mobile/offline) |

---

## 9. Cross-references

→ **006-zs-infrastructure/002-github-org-architecture.md** — Layer 1: GitHub repos, teams, branches, and secrets
→ **006-zs-infrastructure/003-cloudflare-architecture.md** — Layer 2: CDN, WAF, Pages, Workers, R2, SSL
→ **006-zs-infrastructure/004-domain-architecture.md** — Layer 3: DNS zones, subdomains, and routing
→ **006-zs-infrastructure/005-email-architecture.md** — Layer 4: email routing, SPF/DKIM/DMARC
→ **006-zs-infrastructure/006-ci-cd-architecture.md** — Layer 5: GitHub Actions pipelines and environments
→ **006-zs-infrastructure/007-security-architecture.md** — Defense-in-depth security architecture
→ **006-zs-infrastructure/008-security-policies.md** — Security policies (access control, encryption, disclosure)
→ **006-zs-infrastructure/009-compliance-controls.md** — Compliance controls (HIPAA, GDPR, data residency)
→ **006-zs-infrastructure/010-threat-models.md** — Threat models (API, auth, data, mobile/offline)
→ **008-zs-adrs/002-adr-cloudflare-as-edge-platform.md** — ADR-002: Cloudflare as the sole edge provider
→ **008-zs-adrs/003-adr-github-as-government.md** — ADR-003: the GitHub-as-government model
→ **003-zs-platform/003-deployment-planes.md** — Five-plane deployment model

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
