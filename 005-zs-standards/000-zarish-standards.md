---
id: "ZS-INDEX-005"
title: "zarishsphere-standards"
domain: "005-zs-standards"
doc-type: "index"
entity-type: "landing-page"
description: >-
  Landing page for the ZarishStandards part of the ZarishSphere documentation. Indexes the seventeen documents covering the ZARISH-STANDARDS registry, the transformation model, the standards schema, the pipeline to platform, FHIR R5 conventions, API conventions, form schemas, i18n, and terminology governance.
version: "1.0.0"
status: "stable"
tags:
  - "index"
  - "zarish-standards"
  - "fhir"
  - "api"
isolation_tier: "global"
canonical: true
audience:
  - "contributors"
  - "ai-agents"
last_updated: "2026-08-01"
---
# 000-zarish-standards.md
## ZARISH-STANDARDS — the standards registry

ZARISH-STANDARDS is the curated implementation subset of ZARISH-INDEX: the 40-domain standards registry with machine-readable schemas, FHIR R5 conventions, API design conventions, form specifications, and terminology governance that the platform enforces.

## Document index

| # | File | Description | Type | Status |
|---|---|---|---|---|
| 001 | [001-zarish-standards-overview.md](001-zarish-standards-overview.md) | Strategic direction for ZARISH-STANDARDS: curated subset of ZARISH-INDEX, three-type labelling, 40-domain standards hierarchy, display rules, ADR-based maintenance | Direction | Stable |
| 002 | [002-transformation-model.md](002-transformation-model.md) | Data-mapping and transformation model converting 22-field ZARISH-INDEX entries into 28-field ZS-* standard definitions with five validation gates | Specification | Stable |
| 003 | [003-standards-schema.md](003-standards-schema.md) | Machine-readable JSON Schema (Draft 2020-12), YAML, and TypeScript schemas for ZS-* records with 15 mandatory fields and enum registries | Specification | Stable |
| 004 | [004-standards-to-platform-pipeline.md](004-standards-to-platform-pipeline.md) | Real-time standards enforcement pipeline from registry to G2A Engine: release artifacts, per-plane distribution, three-level compliance checking, sync and caching | Specification | Stable |
| 005 | [005-fhir-r5-conventions.md](005-fhir-r5-conventions.md) | FHIR R5 resource-level conventions: UUID v7 resource IDs, tenant isolation, mandatory meta, AuditEvent generation, pagination, OperationOutcome format | Specification | Stable |
| 006 | [006-fhir-profiling-policy.md](006-fhir-profiling-policy.md) | When to create FHIR profiles, FSH and SUSHI authoring rules, mandatory extensions, naming, terminology binding strengths, CI validation | Policy | Stable |
| 007 | [007-fhir-search-standards.md](007-fhir-search-standards.md) | Required and optional FHIR search parameters, modifiers, chaining, multi-tenant enforcement, pagination defaults, performance targets | Specification | Stable |
| 008 | [008-fhir-audit-policy.md](008-fhir-audit-policy.md) | Mandatory AuditEvent triggers and protected resources, async audit writing (PostgreSQL, NATS, R2), per-plane retention, access control | Policy | Stable |
| 009 | [009-fhir-r4-bridge-policy.md](009-fhir-r4-bridge-policy.md) | R4 to R5 bidirectional bridge policy: approved and prohibited use cases, translation rules, lossy-translation register, test coverage | Policy | Stable |
| 010 | [010-api-rest-design-guide.md](010-api-rest-design-guide.md) | REST API design conventions: URL patterns, headers, pagination, URL-path versioning, HTTP status mapping, per-plane rate limits | Guide | Stable |
| 011 | [011-api-openapi-conventions.md](011-api-openapi-conventions.md) | OpenAPI 3.2 conventions: spec location, required info block, FHIR endpoint patterns, error responses, PascalCase schema naming | Guide | Stable |
| 012 | [012-api-asyncapi-conventions.md](012-api-asyncapi-conventions.md) | AsyncAPI 3.1 event conventions over NATS 2.14.4 JetStream: subject naming, event envelope, DLQ routing, retention config | Guide | Stable |
| 013 | [013-form-schema-specification.md](013-form-schema-specification.md) | ZS Form Schema v1: root form structure, 14 field types with FHIR R5 fhirPath mapping, terminology-coded options, display conditions | Specification | Stable |
| 014 | [014-form-validation-rules.md](014-form-validation-rules.md) | Automated CI validation rules (V-01 to V-13) for forms: JSON schema, FHIR mapping completeness, terminology existence, i18n checks | Specification | Stable |
| 015 | [015-i18n-key-conventions.md](015-i18n-key-conventions.md) | i18n key naming, locale files, translation workflow, and runtime fallback chain (bn-BD → bn → en → key name) | Guide | Stable |
| 016 | [016-terminology-governance.md](016-terminology-governance.md) | Terminology governance: approved code systems by domain, coding requirements per resource, update schedule, caching, deprecation policy | Policy | Stable |
| 017 | [017-terminology-code-systems.md](017-terminology-code-systems.md) | Per-system reference for ICD-11, SNOMED CT, LOINC 2.82, CIEL, RxNorm, and CVX: source, licence, access, update frequency, FHIR coding patterns | Reference | Stable |

## See also

→ **README.md** — Space landing page and how the ten domains fit together
→ **004-zs-index/000-zarish-index.md** — The global standards index this registry curates
→ **003-zs-platform/004-g2a-engine.md** — The pipeline that consumes these standards
