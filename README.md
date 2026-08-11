---
id: "ZS-README"
title: "zarishsphere-documentation"
domain: "zs-meta"
doc-type: "readme"
entity-type: "landing-page"
description: >-
  Landing page for the ZarishSphere documentation space. Indexes the ten ZUSS domains covering the foundation's constitution and governance, the platform architecture, ZarishIndex, ZarishStandards, infrastructure, technology stack, architecture decision records, operations runbooks, and the product ecosystem.
version: "1.0.0"
status: "stable"
tags:
  - "readme"
  - "landing"
  - "documentation"
isolation_tier: "global"
canonical: true
audience:
  - "contributors"
  - "deployers"
  - "ai-agents"
last_updated: "2026-08-01"
---
# ZarishSphere Documentation

ZarishSphere is an open-source initiative building universal Digital Public Infrastructure (DPI) — a GUI-first, offline-capable platform of platforms, governed entirely in the open on GitHub. This space is the documentation home for the whole ecosystem: the foundation's governance, the platform architecture, the standards registry (ZARISH-STANDARDS), the global standards index (ZARISH-INDEX), the technology stack, the architecture decision records, and the operational runbooks that keep the project running.

## Start here

- **New to ZarishSphere?** Read the [Foundation](#foundation) section for the charter and governance model.
- **Evaluating the platform?** Read [Platform Overview](003-zs-platform/001-platform-overview.md).
- **Contributing code or docs?** Read [Contributor Guidelines](002-zs-foundation/004-contributor-guidelines.md) and the [Documentation SOP](009-zs-operations/001-sop-new-document-creation.md).
- **Looking for a specific standard?** See [ZARISH-STANDARDS](005-zs-standards/001-zarish-standards-overview.md) or [ZARISH-INDEX](004-zs-index/001-zarish-index-overview.md).

## How this documentation is organized

This space mirrors the ten ZUSS domains maintained in the project's source repository:

| # | Domain | Covers |
|---|---|---|
| 001 | zs-meta | Constitution, profiles, writing rules, glossary, agent strategy |
| 002 | zs-foundation | Charter, governance, licensing, contributor guidelines, country adoption model |
| 003 | zs-platform | Platform overview, modules, deployment planes, FHIR and G2A architecture |
| 004 | zs-index | ZARISH-INDEX — the global standards index |
| 005 | zs-standards | ZARISH-STANDARDS — the 40-domain standards registry |
| 006 | zs-infrastructure | GitHub org, Cloudflare, domains, security, compliance |
| 007 | zs-tech-stack | The pinned, version-exact technology stack |
| 008 | zs-adrs | Architecture Decision Records |
| 009 | zs-operations | SOPs — deployment, incident response, onboarding, backup and DR |
| 010 | zs-ecosystem | Console, marketplace, builder, SDK, CLI, API, engine specifications |

The ten domains are grouped into ten parts in the [Summary](SUMMARY.md):

1. **Welcome and Foundation** — the constitution and the foundation's charter, governance, licensing, and contributor guidelines
2. **Platform Architecture** — the deployable, Go-native platform and its five deployment planes
3. **ZarishIndex** — the global standards index project
4. **ZarishStandards and FHIR** — the curated standards registry and FHIR R5 conventions
5. **Infrastructure** — GitHub, Cloudflare, domains, security, and compliance
6. **Technology Stack** — the pinned, version-exact tooling
7. **Architecture Decision Records** — the decisions behind the architecture
8. **Operations** — the SOPs that keep the project running
9. **Product Ecosystem** — the nineteen product specifications
10. **Standards and References** — the writing rules, glossary, and agent strategy

Every page in this space traces back to a source document maintained outside GitBook and is kept in sync through [Git Sync](https://gitbook.com/docs/getting-started/git-sync). This `README.md` and `SUMMARY.md` are the source of truth and are edited only through the repository, never through the GitBook web editor, to avoid sync conflicts.

## License

Content in this space is dual-licensed under Apache 2.0 and Creative Commons, per the [Licensing Policy](002-zs-foundation/003-licensing-policy.md). See that document for the exact terms that apply to code versus documentation.
