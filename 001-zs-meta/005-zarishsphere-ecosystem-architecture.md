# 005-zarishsphere-ecosystem-architecture.md
## ZarishSphere Ecosystem Architecture — Master Map
### All components, repositories, relationships, and navigation — V1

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative blueprint for all repository and documentation structure

---

## Table of contents

1. [Conceptual overview](#1-conceptual-overview)
2. [Entity map](#2-entity-map)
3. [GitHub organization inventory](#3-github-organization-inventory)
4. [zs-docs folder architecture](#4-zs-docs-folder-architecture)
5. [zs-zarish-index docs structure](#5-zs-zarish-index-docs-structure)
6. [zs-zarish-standards docs structure](#6-zs-zarish-standards-docs-structure)
7. [Subdomain architecture](#7-subdomain-architecture)
8. [Email architecture](#8-email-architecture)
9. [As-code layer inventory](#9-as-code-layer-inventory)
10. [Navigation rules](#10-navigation-rules)
11. [Initialization sequence](#11-initialization-sequence)

---

## 1. Conceptual overview

ZarishSphere is not one project. It is a complete ecosystem of interconnected components, each with a distinct function, all governed by the ZarishSphere Foundation under the 12-law constitution. Every component lives under a single GitHub organization — `zarishsphere` — and is documented as markdown files in `zs-docs` before any code is written.

### 1.1 High-level architecture

```
                     ┌──────────────────────────────────────┐
                     │        ZarishSphere Foundation       │
                     │     (Governance · Constitution)      │
                     │     Single source: zs-docs           │
                     └──────────────────┬───────────────────┘
                                        │ governs
                    ┌───────────────────┼───────────────────────┐
                    │                   │                       │
          ┌─────────▼─────────┐  ┌──────▼──────────┐  ┌────────▼──────────┐
          │   ZARISH-INDEX    │  │ZARISH-STANDARDS │  │  ZarishSphere     │
          │  (Data engine)    │→ │(Transform layer)│→ │  Platform         │
          │  Indexes ALL      │  │ Converts        │  │  (Platform of     │
          │  global standards │  │ metadata into   │  │   Platforms)      │
          └───────────────────┘  │ deployable      │  └────────┬──────────┘
                                 │ assets          │           │
                                 └─────────────────┘           │
                                        │                      │
                                        ▼                      ▼
                          ┌────────────────────────┐  ┌────────────────┐
                          │   Ecosystem Layer      │  │  User Layer    │
                          │  ┌──────────────────┐  │  │ ┌────────────┐ │
                          │  │ Console (GUI)    │  │  │ │ Apps       │ │
                          │  │ Marketplace      │  │  │ │ Forms      │ │
                          │  │ Builder          │  │  │ │ Workflows  │ │
                          │  │ SDK · CLI · API  │  │  │ │ Dashboards │ │
                          │  │ Modules · Dist.  │  │  │ │ Reports    │ │
                          │  └──────────────────┘  │  │ └────────────┘ │
                          │  ┌──────────────────┐  │  └────────────────┘
                          │  │ Engine · System  │  │
                          │  │ Identity · Audit │  │
                          │  └──────────────────┘  │
                          └────────────────────────┘
```

### 1.2 Data flow

```
ZARISH-INDEX → ZARISH-STANDARDS → ZarishSphere Platform → Ecosystem Layer → User Layer
(raw metadata)  (structured assets)  (deployable services) (Console, Market)  (Apps, Forms)
```

### 1.3 Documentation flow

```
zarishsphere (master documentation for entire ZarishSphere ecosystem)
  ├── 001-zs-meta/              ─── identity, constitution, ZUSS, architecture, glossary
  ├── 002-zs-foundation/        ─── governance, charter, licensing
  ├── 003-zs-platform/          ─── platform architecture, modules, G2A, FHIR
  ├── 004-zs-index/             ─── INDEX cross-reference
  ├── 005-zs-standards/         ── STANDARDS cross-reference
  ├── 006-zs-infrastructure/    ─── Cloudflare, GitHub, domains
  ├── 007-zs-tech-stack/        ─── technology decisions
  ├── 008-zs-adrs/              ─── architecture decision records
  ├── 009-zs-operations/        ─── SOPs and procedures
  └── 010-zs-ecosystem/         ─── component specifications (Console, Marketplace, Builder, etc.)
```

---

## 2. Entity map

### 2.1 Governing entity

| Entity | Type | Role | Primary repo | Docs location |
|---|---|---|---|---|
| ZarishSphere Foundation | Institution | Governance, constitution, policy | `zs-docs` | `zs-docs/002-zs-foundation/` |

### 2.2 Core infrastructure

| Component | Type | Role | Primary repo |
|---|---|---|---|
| ZarishIndex | Data engine | Indexes every global standard across all domains | `zs-index` |
| ZarishStandards | Transformation layer | Converts indexed metadata into deployable assets | `zs-standards` |
| ZarishSphere Platform | Platform-of-platforms | Deployable infrastructure layer | `zs-platform` |
| G2A Engine | Execution engine | Automates standard-to-asset transformation | `zs-g2a-engine` |

### 2.3 Ecosystem management

| Component | Type | Role |
|---|---|---|
| ZarishSphere Console | Management GUI | Browser-based control center for the entire ecosystem |
| ZarishSphere Marketplace | Discovery hub | Browse, discover, and install components in one click |
| ZarishSphere Builder | Creation tool | GUI-based form, workflow, and module builder (no code) |

### 2.4 User-facing

| Component | Type | Role |
|---|---|---|
| ZarishSphere Apps | Pre-built applications | Ready-to-use domain applications |
| ZarishSphere Forms | Form engine | Dynamic forms generated from standards |
| ZarishSphere Services | Backend services | Identity, audit, sync, notification, export, integration |

### 2.5 Developer and integration

| Component | Type | Role |
|---|---|---|
| ZarishSphere SDK | Development kit | Libraries for building on the platform (optional) |
| ZarishSphere CLI | Command-line tool | Terminal access (always secondary to GUI) |
| ZarishSphere API | Programmatic interface | REST and GraphQL APIs |

### 2.6 Modular deployment

| Component | Type | Role |
|---|---|---|
| ZarishSphere Modules | Domain packages | Independently deployable modules (health, education, etc.) |
| ZarishSphere Distributions | Pre-packaged bundles | Ready-to-deploy bundles for specific use cases |
| ZarishSphere Engine | Core runtime | G2A, module, form, workflow, sync, report runtimes |
| ZarishSphere System | Base layer | Identity, security, audit, config, monitoring |

### 2.7 Relationship rules

- ZarishIndex produces metadata. ZARISH-STANDARDS consumes it. One-directional.
- ZarishStandards produces structured standards. ZarishSphere Platform consumes them. One-directional.
- The Platform powers the Ecosystem Layer (Console, Marketplace, Builder) and the User Layer (Apps, Forms).
- The Foundation governs all components. The Foundation has no code — only governance documents.
- Every component is independently deployable (Law 7). No component requires another to function.
- The Console is the primary interface to every component (Law 6).
- `zarishsphere` is the single source of truth and master navigation hub for all components.

---

## 3. GitHub organization inventory

**Organization:** `zarishsphere` — https://github.com/zsdotcom

All repositories follow ZUSS naming: `zs-{layer}-{module}[-{submodule}]`

### 3.1 Active V1 repositories

| Repository | Purpose | Status |
|---|---|---|
| `zarishsphere` | Master documentation · Ecosystem docs · Foundation governance | V1 — primary focus |
| `zs-index` | ZarishIndex project: data, scripts, docs | V1 — in design |
| `zs-standards` | ZarishStandards project: transformation, schemas, docs | V1 — in design |
| `zs-note` | ZarishNote project: transformation, schemas, docs | V1 — in design |

### 3.2 Platform repositories

| Repository | Purpose |
|---|---|
| `zs-platform` | ZarishSphere Platform core — platform-of-platforms infrastructure |
| `zs-g2a-engine` | Guideline-to-Action engine core — standard transformation automation |
| `zs-fhir-server` | Go-native FHIR R5 server (lightweight, less than 150 MB) |
| `zs-infra` | Infrastructure as Code — Cloudflare, GitHub Actions, configuration |

### 3.3 Ecosystem repositories

| Repository | Purpose |
|---|---|
| `zs-console` | ZarishSphere Console — browser-based management center |
| `zs-marketplace` | ZarishSphere Marketplace — component discovery and deployment |
| `zs-builder` | ZarishSphere Builder — GUI-based no-code creation tool |
| `zs-forms` | ZarishSphere Forms — dynamic form engine |
| `zs-sdk` | ZarishSphere SDK — development kit for platform builders |
| `zs-cli` | ZarishSphere CLI — command-line interface (secondary) |
| `zs-api` | ZarishSphere API gateway and endpoint definitions |
| `zs-services` | ZarishSphere backend services (identity, audit, sync, etc.) |

### 3.4 Module repositories

| Repository | Purpose |
|---|---|
| `zs-modules-health` | Health domain module |
| `zs-modules-education` | Education domain module |
| `zs-modules-logistics` | Logistics domain module |
| `zs-modules-protection` | Protection domain module |
| `zs-modules-nutrition` | Nutrition domain module |
| `zs-modules-wash` | WASH domain module |
| `zs-modules-environment` | Environment domain module |
| `zs-modules-human-rights` | Human rights domain module |
| `zs-modules-governance` | Governance and public administration module |
| `zs-modules-trade` | Trade and commerce module |

> Additional domain modules follow `zs-modules-{domain}` naming as the domain registry expands toward 100+ domains.

### 3.5 Content repositories

| Repository | Purpose |
|---|---|
| `zs-content-forms` | Domain-agnostic form definitions (FHIR Questionnaire) |
| `zs-content-protocols` | Clinical and operational protocol definitions |
| `zs-content-templates` | Deployment templates and distribution packages |
| `zs-content-reports` | Report templates and dashboard definitions |

### 3.6 Repository creation sequence

Create repositories in this order. Do not create a repository before its documentation is complete in `zs-docs`.

```
 1. zs-docs              ← create first, all other repos wait for this
 2. zs-zarish-index      ← only after 004-zs-index/ is complete in zs-docs
 3. zs-zarish-standards  ← only after 005-zs-standards/ is complete in zs-docs
 4. zs-platform          ← only after 003-zs-platform/ is complete in zs-docs
 5. zs-g2a-engine        ← only after 003-zs-platform/004-g2a-engine.md is complete
 6. zs-infra             ← only after 006-zs-infrastructure/ is complete in zs-docs
 7. zs-console           ← only after 010-zs-ecosystem/001-console-spec.md is complete
 8. zs-marketplace       ← only after 010-zs-ecosystem/002-marketplace-spec.md is complete
 9. zs-builder           ← only after 010-zs-ecosystem/003-builder-spec.md is complete
10. zs-fhir-server       ← only after 007-zs-tech-stack/002-go-fhir-server.md is complete
11. All other components ← after corresponding docs in 010-zs-ecosystem/ exist in zs-docs
12. Domain modules       ← after platform is operational
```

---

## 4. zs-docs folder architecture

`zs-docs` is the single source of truth for the ZarishSphere Foundation and Platform. It is also the master navigation index for ZARISH-INDEX and ZARISH-STANDARDS.

### 4.1 Top-level structure

```
zs-docs/
├── README.md
├── AGENTS.md
├── llms.txt
├── TODO.md
├── 001-zs-meta/               ← identity, constitution, ZUSS, architecture, glossary
├── 002-zs-foundation/         ← governance, charter, licensing, contributors
├── 003-zs-platform/           ← platform architecture, modules, G2A, FHIR, API
├── 004-zs-index/              ← ZARISH-INDEX cross-reference
├── 005-zs-standards/          ← ZARISH-STANDARDS cross-reference
├── 006-zs-infrastructure/     ← Cloudflare, GitHub, domains, email, CI/CD
├── 007-zs-tech-stack/         ← technology decisions, tools, versions
├── 008-zs-adrs/               ← architecture decision records
├── 009-zs-operations/         ← SOPs and procedures
└── 010-zs-ecosystem/          ← ecosystem component specifications
```

### 4.2 001-zs-meta — identity and standards

Purpose: The foundational identity documents and ZUSS writing standard. These documents define what ZarishSphere is, how it is governed, and how everything is written.

```
001-zs-meta/
├── 001-zarishsphere-constitution.md       # 12 laws across 4 tiers — supreme governing document
├── 002-zarishsphere-profile.md            # Foundation identity, vision, full ecosystem description
├── 003-zarishsphere-founder-profile.md    # Founder context for AI agents
├── 004-zarishsphere-writing-rules.md      # ZUSS — all naming, formatting, structure rules
├── 005-zarishsphere-ecosystem-architecture.md  # This document — the master map
├── 006-zarishsphere-glossary.md           # All ZarishSphere terms defined
├── 007-zarishsphere-agent-ecosystem-strategy.md  # AI agent, MCP, and skills strategy
└── 008-zarishsphere-monorepo-blueprint.md # Monorepo architecture blueprint (vision)
```

**Rule:** No document in any other folder may contradict `001-zs-meta/`. All other documents are downstream of meta.

> **Cross-reference note:** → **008-zarishsphere-monorepo-blueprint.md** records the monorepo layout as an alternative/vision and re-scopes part of the folder architecture above. The multi-repo layout documented in this file remains the current, authoritative architecture; the monorepo blueprint does not supersede it.

### 4.3 002-zs-foundation — governance

Purpose: The legal, ethical, and operational governance of the ZarishSphere Foundation as an institution.

```
002-zs-foundation/
├── 001-foundation-charter.md            # Foundation mission, scope, obligations
├── 002-governance-model.md              # How decisions are made, who has authority
├── 003-licensing-policy.md              # Apache 2.0 + CC BY 4.0 application rules
├── 004-contributor-guidelines.md        # How anyone contributes to any project
└── 005-country-adoption-model.md        # Country-level adoption and onboarding
```

### 4.4 003-zs-platform — ZarishSphere platform

Purpose: The ZarishSphere Platform architecture — all modules, services, APIs, and deployment planes.

```
003-zs-platform/
├── 001-platform-overview.md             # What the platform is, five planes, G2A
├── 002-module-architecture.md           # Module sovereignty rules, how modules relate
├── 003-deployment-planes.md             # Plane 0 (air-gapped) through Plane 4 (global SaaS)
├── 004-g2a-engine.md                    # Guideline-to-Action engine specification
├── 005-fhir-architecture.md             # FHIR R5 integration — profiles, endpoints, IDs
├── 006-api-design.md                    # API contracts, REST patterns, versioning
├── 007-data-model.md                    # Core data entities, ZS-UID patterns
├── 008-domain-registry.md               # All 40 domains with scope and module status
├── 009-repository-catalog.md            # Repository inventory and ownership
└── 010-free-tier-resource-map.md        # Free-tier capacity map for all services
```

### 4.5 004-zs-index — INDEX project cross-reference

Purpose: Master-level overview of ZARISH-INDEX as seen from the platform level. Not the project's own documentation (that lives in `zs-zarish-index/docs/`), but the integration specification and cross-reference.

```
004-zs-index/
├── 001-zarish-index-overview.md         # What ZARISH-INDEX is — platform-level view
├── 002-domain-taxonomy-40.md            # The 40-domain classification system
├── 003-metadata-schema.md               # ZI-[DOMAIN_CODE]-[NNNNN] schema definition
├── 004-harvesting-policy.md             # License compliance, what is harvested, what is not
└── 005-zarish-index-to-platform.md      # How INDEX feeds STANDARDS and Platform (integration spec)
```

**Cross-link rule:** Every document in this folder must include a cross-reference to the corresponding document in `zs-zarish-index/docs/` (cross-project).

### 4.6 005-zs-standards — STANDARDS project cross-reference

Purpose: Master-level overview of ZARISH-STANDARDS as seen from the platform. Integration specification only.

```
005-zs-standards/
├── 001-zarish-standards-overview.md     # What ZARISH-STANDARDS is — platform-level view
├── 002-transformation-model.md          # How raw INDEX metadata becomes structured standards
├── 003-standards-schema.md              # Standard entity structure, fields, identifiers
└── 004-standards-to-platform-pipeline.md # How STANDARDS feeds Platform modules (integration spec)
```

### 4.7 006-zs-infrastructure — everything that runs the system

Purpose: All infrastructure — Cloudflare, GitHub, domains, email, CI/CD. This is Infrastructure as Code documentation: every setting, every configuration, every value is written here before being configured in a GUI or file.

```
006-zs-infrastructure/
├── 001-infrastructure-overview.md       # Full infrastructure stack — all components
├── 002-github-org-architecture.md       # All repos, branch policies, team permissions
├── 003-cloudflare-architecture.md       # DNS zones, Pages, Workers, R2, configuration
├── 004-domain-architecture.md           # zarishsphere.com + all subdomains + routing rules
├── 005-email-architecture.md            # All email addresses, routing, Cloudflare Email Routing
├── 006-ci-cd-architecture.md            # GitHub Actions — workflow naming, triggers, structure
├── 007-security-architecture.md         # Security model and controls
├── 008-security-policies.md             # Security policies and procedures
├── 009-compliance-controls.md           # Compliance and regulatory controls
└── 010-threat-models.md                 # Threat models per deployment plane
```

### 4.8 007-zs-tech-stack — technology decisions

Purpose: Every technology choice, version-pinned. This is the Technology as Code layer — any new tooling decision is recorded here before it is used anywhere.

```
007-zs-tech-stack/
├── 001-tech-stack-master.md             # Complete versioned inventory of all tools
├── 002-go-fhir-server.md                # Go FHIR R5 server — choice, config, constraints
├── 003-frontend-stack.md                # Frontend: frameworks, hosting, build tools
├── 004-data-pipeline.md                 # ZARISH-INDEX data collection and processing stack
├── 005-no-code-tools.md                 # All GUI tools: Cloudflare Dashboard, GitHub GUI, etc.
└── 006-oss-tool-catalog.md              # Open-source tool catalog and licenses
```

**Rule:** `latest` tag is forbidden in every file in this folder. Every tool has a pinned version.

### 4.9 008-zs-adrs — architecture decision records

Purpose: One file per architectural decision, in ZUSS ADR format (6 sections, per ZUSS §8.1). Every decision that affects the ecosystem architecture is recorded here before implementation.

```
008-zs-adrs/
├── 001-adr-go-as-primary-language.md
├── 002-adr-cloudflare-as-edge-platform.md
├── 003-adr-github-as-government.md
├── 004-adr-no-hapi-fhir.md
├── 005-adr-fhir-r5-over-r4.md
├── 006-adr-zero-cost-toolchain.md
├── 007-adr-markdown-first-documentation.md
├── 008-adr-apache-cc-dual-license.md
├── 009-adr-no-vendor-lock-in.md
└── 010-adr-gui-first-ux.md
```

**Rule:** Every file follows ZUSS §8.1 — six required sections: Decision, Context, Alternatives Considered, Reason for Decision, Consequences, Status.

### 4.10 009-zs-operations — procedures

Purpose: Standard Operating Procedures (SOPs) for how ZarishSphere operates. GUI-first steps for every process.

```
009-zs-operations/
├── 001-sop-new-document-creation.md     # How to create any new ZUSS-compliant document
├── 002-sop-github-workflow.md           # How to create repos, branches, PRs via GitHub GUI
├── 003-sop-contribution-process.md      # How any contributor submits work
├── 004-sop-zuss-compliance-audit.md     # How to run a compliance audit (the process used above)
└── 005-sop-deployment-checklist.md      # Pre-deployment verification steps
```

### 4.11 010-zs-ecosystem — ecosystem component specifications

Purpose: Specifications for every ecosystem component — Console, Marketplace, Builder, Apps, Forms, SDK, CLI, API, Services, Modules, Distributions, Engine, System, and the content, Home, and FHIR Hub products. Each component is documented here before its first commit.

```
010-zs-ecosystem/
├── 001-console-spec.md                  # ZarishSphere Console specification
├── 002-marketplace-spec.md              # ZarishSphere Marketplace specification
├── 003-builder-spec.md                  # ZarishSphere Builder specification
├── 004-apps-spec.md                     # ZarishSphere Apps specification
├── 005-forms-spec.md                    # ZarishSphere Forms specification
├── 006-sdk-spec.md                      # ZarishSphere SDK specification
├── 007-cli-spec.md                      # ZarishSphere CLI specification
├── 008-api-spec.md                      # ZarishSphere API specification
├── 009-services-spec.md                 # ZarishSphere Services specification
├── 010-modules-spec.md                  # ZarishSphere Modules specification
├── 011-distributions-spec.md            # ZarishSphere Distributions specification
├── 012-engine-spec.md                   # ZarishSphere Engine specification
├── 013-system-spec.md                   # ZarishSphere System specification
├── 014-content-forms-spec.md            # Content forms product specification
├── 015-content-protocols-spec.md        # Content protocols product specification
├── 016-content-templates-spec.md        # Content templates product specification
├── 017-content-reports-spec.md          # Content reports product specification
├── 018-home-spec.md                     # Home product specification
└── 019-fhir-hub-spec.md                 # FHIR Hub product specification
```

**Rule:** Every component listed in the constitution §11 (Ecosystem scope) must have a corresponding specification in this folder before it receives its first code commit.

### 4.12 Complete file count

| Folder | Files |
|---|---|
| Root (`README.md`, `AGENTS.md`, `llms.txt`, `TODO.md`) | 4 |
| `001-zs-meta/` | 8 |
| `002-zs-foundation/` | 5 |
| `003-zs-platform/` | 10 |
| `004-zs-index/` | 5 |
| `005-zs-standards/` | 4 |
| `006-zs-infrastructure/` | 10 |
| `007-zs-tech-stack/` | 6 |
| `008-zs-adrs/` | 23 |
| `009-zs-operations/` | 15 |
| `010-zs-ecosystem/` | 19 |
| **Total** | **109 numbered documents** |

> **Note:** This count covers the numbered documents in `zs-docs` as of the current revision. The monorepo blueprint (→ **008-zarishsphere-monorepo-blueprint.md**) re-scopes this layout as a monorepo alternative; this multi-repo count remains the plan for the current architecture.

---

## 5. zs-zarish-index docs structure

The ZARISH-INDEX project documentation lives inside the `zs-zarish-index` repository under a `docs/` folder (cross-project). This is the project's own operational documentation — distinct from the cross-reference documents in `zs-docs/004-zs-index/`.

```
zs-zarish-index/
├── README.md
├── TODO.md
└── docs/
    ├── 001-project-charter.md           # Mission, scope, vision, non-negotiables
    ├── 002-40-domain-taxonomy.md        # Full taxonomy: all 40 domains, subdomains, scope
    ├── 003-entry-schema.md              # Schema for every index entry (ZI-[DOMAIN]-[NNNNN])
    ├── 004-harvesting-workflow.md       # How sources are found, verified, ingested
    ├── 005-licensing-compliance.md      # What can be harvested, what cannot, why
    ├── 006-quality-standards.md         # Accuracy, completeness, and freshness rules
    ├── 007-source-registry.md           # Registry of all indexed source organizations
    └── 008-api-reference.md             # How to query the index (future API specification)
```

**File count:** 10 (including README and TODO)

---

## 6. zs-zarish-standards docs structure

The ZARISH-STANDARDS project documentation lives inside the `zs-zarish-standards` repository (cross-project).

```
zs-zarish-standards/
├── README.md
├── TODO.md
└── docs/
    ├── 001-project-charter.md           # Mission, scope, what STANDARDS is not
    ├── 002-transformation-model.md      # How INDEX metadata becomes structured standards
    ├── 003-standards-schema.md          # Standard entity fields, relationships, IDs
    ├── 004-domain-coverage.md           # Which of the 40 domains have standard definitions
    ├── 005-integration-guide.md         # How Platform modules consume STANDARDS
    └── 006-contribution-guide.md        # How domain experts contribute standard definitions
```

**File count:** 8 (including README and TODO)

---

## 7. Subdomain architecture

**Primary domain:** `zarishsphere.com` (Cloudflare DNS, Cloudflare Pages)

All subdomains are configured through Cloudflare DNS. Cloudflare Free plan supports unlimited DNS records (including subdomains). The constraint is on Cloudflare Workers (100,000 requests/day free) and Pages (500 builds/month free).

| Subdomain | Purpose | Target | Status |
|---|---|---|---|
| `zarishsphere.com` | Ecosystem landing page | Cloudflare Pages | Planned |
| `console.zarishsphere.com` | ZarishSphere Console (management center) | Cloudflare Pages + Workers | Planned |
| `marketplace.zarishsphere.com` | ZarishSphere Marketplace | Cloudflare Pages | Planned |
| `builder.zarishsphere.com` | ZarishSphere Builder (no-code tool) | Cloudflare Pages + Workers | Planned |
| `docs.zarishsphere.com` | `zs-docs` published documentation | Cloudflare Pages | Planned |
| `index.zarishsphere.com` | ZARISH-INDEX public interface | Cloudflare Pages | Planned |
| `standards.zarishsphere.com` | ZARISH-STANDARDS public interface | Cloudflare Pages | Planned |
| `api.zarishsphere.com` | ZarishSphere API gateway | Cloudflare Workers | Planned |
| `fhir.zarishsphere.com` | FHIR R5 endpoint | Cloudflare Worker → Go server | Planned |
| `status.zarishsphere.com` | Ecosystem status page | Cloudflare Pages | Planned |
| `identity.zarishsphere.com` | Identity and authentication service | Cloudflare Workers | Planned |

> All subdomains above are free to configure. Cloudflare does not limit the number of subdomains on the free plan.

---

## 8. Email architecture

**Email routing:** Cloudflare Email Routing (free, unlimited aliases)

All `*@zarishsphere.com` addresses are routing aliases — they receive mail and forward to `zarishsphere@gmail.com`. No mail server is operated.

| Address | Purpose |
|---|---|
| `hello@zarishsphere.com` | General contact |
| `founder@zarishsphere.com` | Founder direct contact |
| `contribute@zarishsphere.com` | Contribution inquiries |
| `index@zarishsphere.com` | ZARISH-INDEX submissions and inquiries |
| `standards@zarishsphere.com` | ZARISH-STANDARDS inquiries |
| `security@zarishsphere.com` | Security disclosure |
| `legal@zarishsphere.com` | Licensing and legal inquiries |

> All aliases forward to `zarishsphere@gmail.com`. Configure each in Cloudflare Dashboard → Email Routing → Routing Rules.

---

## 9. As-code layer inventory

ZarishSphere is built on the principle that every concern — infrastructure, governance, standards, documentation — is managed as code (committed, versioned, auditable). This table maps each concern to its "as-code" layer.

| Concern | As-code pattern | Location in `zs-docs` | Implementation tool |
|---|---|---|---|
| Governance decisions | Governance as Code | `001-zs-meta/001-zarishsphere-constitution.md` | Git commit · PR review |
| Architecture decisions | Architecture as Code | `008-zs-adrs/` | ADR files · GitHub |
| Standard operating procedures | Process as Code | `009-zs-operations/` | Markdown · GitHub |
| Infrastructure configuration | Infrastructure as Code | `006-zs-infrastructure/` | Cloudflare · GitHub Actions |
| Technology choices | Technology as Code | `007-zs-tech-stack/` | Markdown · version-pinned configs |
| Standards indexing | Index as Code | `004-zs-index/` | ZARISH-INDEX pipeline |
| Standards transformation | Standards as Code | `005-zs-standards/` | G2A engine |
| Forms and assessments | Forms as Code | `010-zs-ecosystem/005-forms-spec.md` | FHIR Questionnaire JSON |
| Console management | Console as Code | `010-zs-ecosystem/001-console-spec.md` | Browser-based GUI |
| Marketplace | Marketplace as Code | `010-zs-ecosystem/002-marketplace-spec.md` | Browser-based GUI |
| Builder | Builder as Code | `010-zs-ecosystem/003-builder-spec.md` | Browser-based GUI |
| Domain modules | Modules as Code | `zs-modules-{domain}` repos | Independent deployable units |
| Deployment distributions | Distribution as Code | `010-zs-ecosystem/011-distributions-spec.md` | YAML · Docker Compose |
| Identity and access | Identity as Code | `010-zs-ecosystem/013-system-spec.md` | ZS-UID · IAM |
| AI agent context | Agent Context as Code | `AGENTS.md` · `llms.txt` · Skills | MCP · SKILL.md |

---

## 10. Navigation rules

### 10.1 How to find any document

Every document in the ZarishSphere ecosystem is findable by following this path:

```
1. Start at zs-docs/README.md or zs-docs/llms.txt
2. Identify which folder applies to your query (001-zs-meta through 010-zs-ecosystem)
3. Open the folder — files are numbered and sortable
4. If the document references another component: follow the cross-link
5. If the document references an ADR: follow the link to 008-zs-adrs/
6. For ecosystem component specs: look in 010-zs-ecosystem/
7. AI agents: read AGENTS.md first for bootstrap context
```

### 10.2 Cross-reference format

Per ZUSS §11: all cross-document references use this format:

```markdown
→ **004-zs-index/004-harvesting-policy.md** — Strategic direction for the ZarishIndex autonomous research project
```

When referencing a document in a different repository:

```markdown
→ **zs-zarish-index/docs/001-project-charter.md** — (cross-project: see `zarishsphere/zs-zarish-index` repo) — ZARISH-INDEX mission, scope, and vision
```

### 10.3 How AI agents navigate this ecosystem

An AI agent reading `zs-docs` gains full context for the ecosystem by reading in this sequence:

```
1. zs-docs/001-zs-meta/001-zarishsphere-constitution.md   ← laws and principles
2. zs-docs/001-zs-meta/005-zarishsphere-ecosystem-architecture.md  ← this document (the map)
3. zs-docs/001-zs-meta/004-zarishsphere-writing-rules.md   ← naming and formatting rules
4. zs-docs/001-zs-meta/006-zarishsphere-glossary.md        ← all terms defined
5. Relevant folder based on task                           ← targeted context
```

Any agent that reads documents 1–4 in sequence has sufficient context to work in any folder, create any document type, and navigate to any project.

---

## 11. Initialization sequence

This is the order of operations for building the `zs-docs` repository from zero. Do not skip steps. Do not create later-numbered documents before earlier-numbered documents in the same folder are complete.

### 11.1 Phase A — foundation (do now)

```
Step 1: Create GitHub repository zs-docs under zarishsphere org
        → GitHub GUI: New repository → name: zs-docs → public → no template → no README

Step 2: Create folder structure
        → Create all 10 folders via GitHub GUI (add a .gitkeep file to each)

Step 3: Commit 001-zs-meta/ documents
        → 001-zarishsphere-constitution.md        (12-law revised version)
        → 002-zarishsphere-profile.md             (foundation identity and mandate)
        → 003-zarishsphere-founder-profile.md     (founder context profile)
        → 004-zarishsphere-writing-rules.md       (ZUSS standard)
        → 005-zarishsphere-ecosystem-architecture.md  (this document)
        → 006-zarishsphere-glossary.md            (canonical terms)
        → 007-zarishsphere-agent-ecosystem-strategy.md  (agent-native strategy)
        → 008-zarishsphere-monorepo-blueprint.md  (monorepo vision blueprint)

Step 4: Commit README.md and TODO.md to root
```

### 11.2 Phase B — foundation governance (next)

```
Step 5: Author 002-zs-foundation/ documents (5 files)
Step 6: Author 003-zs-platform/ documents (10 files)
Step 7: Author 006-zs-infrastructure/ documents (10 files) — Cloudflare and GitHub config
```

### 11.3 Phase C — projects (after B)

```
Step 8: Author 004-zs-index/ (5 files) → then create zs-zarish-index repo
Step 9: Author 005-zs-standards/ (4 files) → then create zs-zarish-standards repo
Step 10: Author 008-zs-adrs/ (23 files) — one per architectural decision
```

### 11.4 Phase D — operations and stack (after C)

```
Step 11: Author 007-zs-tech-stack/ (6 files)
Step 12: Author 009-zs-operations/ (15 files) — SOPs for all processes
```

### 11.5 Phase E — ecosystem components (after D)

```
Step 13: Author 010-zs-ecosystem/ (19 files) — specifications for all ecosystem components
Step 14: Create zs-console repo
Step 15: Create zs-marketplace repo
Step 16: Create zs-builder repo
Step 17: Create remaining component repos as specifications are completed
```

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
