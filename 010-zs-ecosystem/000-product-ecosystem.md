---
id: "ZS-INDEX-010"
title: "zarishsphere-product-ecosystem"
domain: "010-zs-ecosystem"
doc-type: "index"
entity-type: "landing-page"
description: >-
  Landing page for the Product Ecosystem part of the ZarishSphere documentation. Indexes the nineteen specifications covering the Console, Marketplace, Builder, apps, forms, SDK, CLI, API, services, modules, distributions, the core engine, the System base layer, content repositories, the home site, and the FHIR hub.
version: "1.0.0"
status: "stable"
tags:
  - "index"
  - "ecosystem"
  - "specifications"
isolation_tier: "global"
canonical: true
audience:
  - "contributors"
  - "deployers"
  - "ai-agents"
last_updated: "2026-08-01"
---
# 000-product-ecosystem.md
## Product ecosystem — nineteen specifications

The published product specifications for every component of the ZarishSphere ecosystem — 13 ecosystem components published as 19 product specifications. Nineteen documents define the Console, Marketplace, Builder, apps, forms, SDK, CLI, API, services, modules, distributions, the core engine, the System base layer, the content repositories, the home site, and the FHIR hub.

## Document index

| # | File | Description | Type | Status |
|---|---|---|---|---|
| 001 | [001-console-spec.md](001-console-spec.md) | The browser-based Console: the no-code management center for all ecosystem functions | Specification | Stable |
| 002 | [002-marketplace-spec.md](002-marketplace-spec.md) | The Marketplace: discovery and deployment hub for open-source, free ecosystem components with one-click install | Specification | Stable |
| 003 | [003-builder-spec.md](003-builder-spec.md) | The Builder: a GUI no-code tool for forms, workflows, modules, and apps exporting YAML, JSON, Markdown, FHIR Questionnaire, and XLSForm | Specification | Stable |
| 004 | [004-apps-spec.md](004-apps-spec.md) | Pre-built domain apps: catalog, manifest.yaml format, lifecycle, security boundary, offline behaviour, customization via Builder | Specification | Stable |
| 005 | [005-forms-spec.md](005-forms-spec.md) | The Forms dynamic form engine generating browser-based FHIR Questionnaire forms from ZARISH-STANDARDS definitions, fully offline-capable | Specification | Stable |
| 006 | [006-sdk-spec.md](006-sdk-spec.md) | Optional Go, JavaScript, and Python SDKs: architecture, package layout, install, error handling, testing, semver versioning policy | Specification | Stable |
| 007 | [007-cli-spec.md](007-cli-spec.md) | The CLI (Cobra, Viper, Go SDK): always secondary to the Console, for CI/CD and headless automation | Specification | Stable |
| 008 | [008-api-spec.md](008-api-spec.md) | The REST, GraphQL, and webhook API with OpenAPI 3.2 specs, free tiers, and auth methods | Specification | Stable |
| 009 | [009-services-spec.md](009-services-spec.md) | Backend microservices: identity, audit, sync, notification, export, integration; their Plane 0 fallbacks, registry, health endpoints | Specification | Stable |
| 010 | [010-modules-spec.md](010-modules-spec.md) | Independently deployable domain modules: sovereignty rules, module table, module contents | Specification | Stable |
| 011 | [011-distributions-spec.md](011-distributions-spec.md) | Pre-packaged, pre-tested deployment bundles: 5 ready and 6 planned distributions, archive and manifest format, update channels, signing | Specification | Stable |
| 012 | [012-engine-spec.md](012-engine-spec.md) | The core runtime Engine: G2A, module, form, workflow, sync, and report sub-components; Plane 0 standalone mode | Specification | Stable |
| 013 | [013-system-spec.md](013-system-spec.md) | The System base layer: IAM, encryption, config, monitoring, backup, audit; identity tiers; Plane 0 minimal mode | Specification | Stable |
| 014 | [014-content-forms-spec.md](014-content-forms-spec.md) | The zs-content-forms repository of domain-agnostic FHIR R5 Questionnaire forms | Specification | Stable |
| 015 | [015-content-protocols-spec.md](015-content-protocols-spec.md) | The zs-content-protocols repository of machine-readable protocols feeding the G2A Engine | Specification | Stable |
| 016 | [016-content-templates-spec.md](016-content-templates-spec.md) | The zs-content-templates repository of plane-per-template deployment bundles with strict version pinning | Specification | Stable |
| 017 | [017-content-reports-spec.md](017-content-reports-spec.md) | The zs-content-reports repository of YAML report templates and dashboard configs | Specification | Stable |
| 018 | [018-home-spec.md](018-home-spec.md) | The zs-home static Cloudflare Pages landing site at zarishsphere.com | Specification | Stable |
| 019 | [019-fhir-hub-spec.md](019-fhir-hub-spec.md) | The zs-fhir-hub stateless FHIR R5 proxy and integration gateway with R4-to-R5, DHIS2, CSV, and HL7 v2.9.1 adapters | Specification | Stable |

## See also

→ **README.md** — Space landing page and how the ten domains fit together
→ **003-zs-platform/000-platform-architecture.md** — The platform these components form
→ **005-zs-standards/013-form-schema-specification.md** — The form schema the Forms engine implements
