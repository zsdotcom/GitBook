# 010-modules-spec.md
## ZarishSphere Modules specification
### Domain packages

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Module sovereignty](#2-module-sovereignty)
3. [Available modules](#3-available-modules)
4. [Module contents](#4-module-contents)
5. [Cross-references](#5-cross-references)

---

## 1. Purpose

ZarishSphere Modules are independently deployable domain packages. Each module corresponds to one domain and contains all forms, workflows, data models, and services needed for that domain. No module requires another module to function.

## 2. Module sovereignty

| Rule | Meaning |
|---|---|
| Independent deployment | Each module deploys alone |
| Independent data | Each module has its own database schema |
| Independent lifecycle | Each module updates independently |
| Standard interfaces | Modules communicate through APIs only |
| No coupling | No module imports another module's internals |

## 3. Available modules

| Module code | Domain | Status |
|---|---|---|
| ZS-MOD-HEALTH | Health | Planned |
| ZS-MOD-EDUCATION | Education | Planned |
| ZS-MOD-LOGISTICS | Logistics | Planned |
| ZS-MOD-PROTECTION | Protection | Planned |
| ZS-MOD-WASH | WASH | Planned |
| ZS-MOD-NUTRITION | Nutrition | Planned |
| ZS-MOD-SHELTER | Shelter | Planned |
| ZS-MOD-CAMP | Camp management | Planned |
| ZS-MOD-FINANCE | Finance | Planned |
| ZS-MOD-HUMANITARIAN | Humanitarian | Planned |

Full 40-domain list in → **003-zs-platform/008-domain-registry.md**.

## 4. Module contents

Each module contains:
- **Forms** (FHIR Questionnaire definitions)
- **Workflows** (approval chains, data pipelines)
- **Data models** (domain-specific entities)
- **Services** (business logic)
- **APIs** (OpenAPI specs)
- **Dashboards** (reporting views)
- **Documentation** (module guide, SOPs)

## 5. Cross-references

→ **003-zs-platform/008-domain-registry.md** — Domain registry: the full 40-domain classification taxonomy behind the module list
→ **010-zs-ecosystem/004-apps-spec.md** — Apps specification: apps declare module dependencies in their manifests
→ **010-zs-ecosystem/011-distributions-spec.md** — Distributions specification: pre-packaged bundles of domain modules
→ **010-zs-ecosystem/012-engine-spec.md** — Engine specification: the module runtime that executes domain modules
→ **010-zs-ecosystem/005-forms-spec.md** — Forms specification: the FHIR Questionnaire forms shipped inside each module
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 7: module sovereignty

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
