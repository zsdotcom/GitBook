---
id: "ZS-INDEX-008"
title: "zarishsphere-architecture-decision-records"
domain: "008-zs-adrs"
doc-type: "index"
entity-type: "landing-page"
description: >-
  Landing page for the Architecture Decision Records part of the ZarishSphere documentation. Indexes the twenty-three ADRs recording every significant architectural decision from Go as the primary language through the Flutter cross-platform mobile decision.
version: "1.0.0"
status: "stable"
tags:
  - "index"
  - "adr"
  - "architecture-decision-records"
isolation_tier: "global"
canonical: true
audience:
  - "contributors"
  - "ai-agents"
last_updated: "2026-08-01"
---
# 000-architecture-decision-records.md
## Architecture Decision Records

Every significant architectural decision in the ZarishSphere ecosystem is recorded as an ADR per ZUSS §8.1: Decision, Context, Alternatives Considered, Reason for Decision, Consequences, and Status. Twenty-three records cover the full decision history from primary language to mobile tooling.

## Document index

| # | File | Description | Status |
|---|---|---|---|
| 001 | [001-adr-go-as-primary-language.md](001-adr-go-as-primary-language.md) | Go is the sole backend language (FHIR server, G2A Engine, gateway); frontend stays React and Next.js | Accepted |
| 002 | [002-adr-cloudflare-as-edge-platform.md](002-adr-cloudflare-as-edge-platform.md) | Cloudflare free tier hosts all web properties | Accepted |
| 003 | [003-adr-github-as-government.md](003-adr-github-as-government.md) | GitHub is the sole operational control plane, implementing Constitution Law 1 | Accepted |
| 004 | [004-adr-no-hapi-fhir.md](004-adr-no-hapi-fhir.md) | Reject HAPI FHIR and all JVM-based FHIR; build a custom Go-native FHIR R5 server | Accepted |
| 005 | [005-adr-fhir-r5-over-r4.md](005-adr-fhir-r5-over-r4.md) | FHIR R5 is the sole canonical FHIR version — no R4 compatibility layer | Accepted |
| 006 | [006-adr-zero-cost-toolchain.md](006-adr-zero-cost-toolchain.md) | Every tool and dependency must be open source with a genuine perpetual free tier | Accepted |
| 007 | [007-adr-markdown-first-documentation.md](007-adr-markdown-first-documentation.md) | All documentation is plaintext Markdown plus YAML frontmatter, git-versioned and ZUSS-compliant | Accepted |
| 008 | [008-adr-apache-cc-dual-license.md](008-adr-apache-cc-dual-license.md) | Dual license: Apache 2.0 for code, CC BY 4.0 for documentation | Accepted |
| 009 | [009-adr-no-vendor-lock-in.md](009-adr-no-vendor-lock-in.md) | All infrastructure providers abstracted behind interfaces with at least two implementations each | Accepted |
| 010 | [010-adr-gui-first-ux.md](010-adr-gui-first-ux.md) | Browser-based GUI is the primary interface; CLI and API always secondary | Accepted |
| 011 | [011-adr-privacy-by-architecture.md](011-adr-privacy-by-architecture.md) | Privacy enforced at the infrastructure layer; no outbound data without explicit revocable consent | Accepted |
| 012 | [012-adr-no-single-person-dependency.md](012-adr-no-single-person-dependency.md) | No feature, credential, or process may depend on a single person; every credential has a documented succession path | Accepted |
| 013 | [013-adr-postgresql-primary-database.md](013-adr-postgresql-primary-database.md) | PostgreSQL 18.4 primary database (JSONB, TimescaleDB 2.29.0, RLS) with SQLite fallback on Planes 0 and 1 | Accepted |
| 014 | [014-adr-nats-jetstream-messaging.md](014-adr-nats-jetstream-messaging.md) | NATS JetStream 2.14.4 is the async messaging backbone | Accepted |
| 015 | [015-adr-valkey-for-caching.md](015-adr-valkey-for-caching.md) | Valkey 9.1.1 over Redis and the SSPL as the caching layer | Accepted |
| 016 | [016-adr-opentofu-infrastructure-as-code.md](016-adr-opentofu-infrastructure-as-code.md) | OpenTofu 1.12.5 over Terraform and BSL as the IaC tool | Accepted |
| 017 | [017-adr-argocd-gitops.md](017-adr-argocd-gitops.md) | Argo CD 3.4.6 is the GitOps deployment engine for Kubernetes clusters (Plane 2+) | Accepted |
| 018 | [018-adr-cilium-service-mesh.md](018-adr-cilium-service-mesh.md) | Cilium 1.20.0 (eBPF, no sidecars) replaces both CNI and service mesh layers | Accepted |
| 019 | [019-adr-carbon-design-system.md](019-adr-carbon-design-system.md) | IBM Carbon Design System 1.113.0 is the primary UI component library | Accepted |
| 020 | [020-adr-microfrontend-architecture.md](020-adr-microfrontend-architecture.md) | Vite 8.2.0 Module Federation microfrontends composed at runtime by the Console shell | Accepted |
| 021 | [021-adr-powersync-mobile-offline.md](021-adr-powersync-mobile-offline.md) | Self-hosted PowerSync 1.23.3 for bidirectional SQLite-to-PostgreSQL mobile offline sync | Accepted |
| 022 | [022-adr-typescript-strict-mode.md](022-adr-typescript-strict-mode.md) | TypeScript 7.0.2 strict mode plus Zod 4.4.3 runtime validation for all frontend code | Accepted |
| 023 | [023-adr-flutter-cross-platform-mobile.md](023-adr-flutter-cross-platform-mobile.md) | Flutter 3.44.7 (Dart 3.12.2) for all mobile apps, offline-first via PowerSync, Riverpod state | Accepted |

## See also

→ **README.md** — Space landing page and how the ten domains fit together
→ **003-zs-platform/001-platform-overview.md** — The architecture these decisions shaped
→ **002-zs-foundation/002-governance-model.md** — How decisions are recorded and superseded
