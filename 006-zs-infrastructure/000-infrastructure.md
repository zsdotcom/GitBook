---
id: "ZS-INDEX-006"
title: "zarishsphere-infrastructure"
domain: "006-zs-infrastructure"
doc-type: "index"
entity-type: "landing-page"
description: >-
  Landing page for the Infrastructure part of the ZarishSphere documentation. Indexes the ten documents covering the infrastructure overview, GitHub org architecture, Cloudflare architecture, domain and email architecture, CI/CD, security architecture and policies, compliance controls, and threat models.
version: "1.0.0"
status: "stable"
tags:
  - "index"
  - "infrastructure"
  - "cloudflare"
  - "security"
isolation_tier: "global"
canonical: true
audience:
  - "contributors"
  - "deployers"
  - "ai-agents"
last_updated: "2026-08-01"
---
# 000-infrastructure.md
## Infrastructure — GitHub, Cloudflare, domains, security

The five-layer infrastructure running the entire ecosystem on GitHub and Cloudflare free tiers, with zero-cost, zero-touch, and offline-first principles, security and compliance controls, and threat models. Ten documents cover the stack from the top-level map through consolidated threat models.

## Document index

| # | File | Description | Type | Status |
|---|---|---|---|---|
| 001 | [001-infrastructure-overview.md](001-infrastructure-overview.md) | Top-level map: five layers on GitHub and Cloudflare free tiers, zero-cost and offline-first principles, Plane 0 air-gapped adaptation, document map | Overview | Stable |
| 002 | [002-github-org-architecture.md](002-github-org-architecture.md) | GitHub org architecture for github.com/zarishsphere: repository inventory, admin and dev team model, branch protection, templates, secret rotation | Specification | Stable |
| 003 | [003-cloudflare-architecture.md](003-cloudflare-architecture.md) | Full Cloudflare free-tier configuration: service limits, subdomain and DNS inventory, WAF rules, Pages projects, Workers routes, R2 buckets, email routing | Specification | Stable |
| 004 | [004-domain-architecture.md](004-domain-architecture.md) | Domain hierarchy for zarishsphere.com: public, API, infra, and reserved subdomains; DNS record types and TTL; CAA, SPF, DKIM, DMARC; Plane 0 local-host resolution | Specification | Stable |
| 005 | [005-email-architecture.md](005-email-architecture.md) | Email architecture: Cloudflare Email Routing (inbound-only), forwarding aliases, SPF and DKIM and DMARC, notification plans, anti-spam policy, Plane 0 queuing | Specification | Stable |
| 006 | [006-ci-cd-architecture.md](006-ci-cd-architecture.md) | CI/CD via GitHub Actions: 4-stage pipeline (validate, build, deploy, notify), per-repo workflow tables, environments, plane-aware release and plane-* branches | Specification | Stable |
| 007 | [007-security-architecture.md](007-security-architecture.md) | Defense-in-depth security: 7 layers from Cloudflare edge to AuditEvent, Vault secrets, encryption strategy, audit controls, Plane 0 model | Specification | Stable |
| 008 | [008-security-policies.md](008-security-policies.md) | Security policies: least-privilege access with the 7-role hierarchy, Keycloak 26.7.0 authentication, encryption at rest and in transit, vulnerability disclosure, secret management | Policy | Stable |
| 009 | [009-compliance-controls.md](009-compliance-controls.md) | Compliance controls: HIPAA §164.312 technical safeguards, GDPR Articles 15/16/17/20/33, per-jurisdiction data residency, consolidated control matrix | Policy | Stable |
| 010 | [010-threat-models.md](010-threat-models.md) | Four consolidated STRIDE threat models (FHIR API, authentication, data, mobile and offline) with threat, impact, mitigation, and detection tables | Reference | Stable |

## See also

→ **README.md** — Space landing page and how the ten domains fit together
→ **008-zs-adrs/000-architecture-decision-records.md** — Decisions behind the zero-cost toolchain (ADR-006), GitHub-as-government (ADR-003), and Cloudflare-as-edge (ADR-002)
→ **007-zs-tech-stack/006-oss-tool-catalog.md** — The pinned tool versions this infrastructure runs
