# 003-builder-spec.md
## ZarishSphere Builder specification
### No-code creation tool

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Builder capabilities](#2-builder-capabilities)
3. [Output format](#3-output-format)
4. [GitHub integration](#4-github-integration)
5. [Cross-references](#5-cross-references)

---

## 1. Purpose

The Builder is a GUI-based creation tool that lets anyone build forms, workflows, modules, and apps without writing code. It is the mechanism through which deployers customize their ZarishSphere deployment to their specific needs — no programming knowledge required.

## 2. Builder capabilities

| Capability | Description |
|---|---|
| Drag-and-drop form builder | Create forms from a palette of field types (text, number, date, select, etc.) |
| Visual workflow designer | Design approval flows, data pipelines, notification rules |
| Module composer | Combine existing components into new configurations |
| App assembler | Create custom domain applications from available modules |
| Template creator | Save configurations as reusable templates for the Marketplace |

### 2.1 Form builder specifics

- Field types: text, number, date, datetime, select, multi-select, radio, checkbox, file, image, location, barcode
- Validation rules: required, min/max, pattern, conditional required
- Display logic: show/hide fields based on other field values
- Scoring: auto-calculate scores from field values
- Multi-language: labels in all configured languages

### 2.2 Workflow builder specifics

- Node types: start, form, decision, approval, notification, integration, end
- Transitions: conditional branching, parallel paths, timeouts
- Approvals: single, multi-stage, parallel
- Notifications: email, SMS, in-app

## 3. Output format

Everything built in the Builder can be exported as:

| Format | Purpose |
|---|---|
| YAML | Configuration files |
| JSON | Data exchange |
| Markdown | Documentation |
| FHIR Questionnaire JSON | Health form standards |
| XLSForm | KoboToolbox compatibility |

## 4. GitHub integration

The Builder can commit outputs directly to a GitHub repository. This ensures:
- All changes are version-controlled (Law 1)
- Changes are auditable
- Rollback is always possible
- Collaboration is enabled through PR review

The GitHub integration is optional. Builder outputs can be downloaded and used offline.

## 5. Cross-references

→ **010-zs-ecosystem/005-forms-spec.md** — Forms specification: the drag-and-drop form builder's output is consumed by the Forms engine
→ **010-zs-ecosystem/004-apps-spec.md** — Apps specification: apps are customized through the Builder's three-way merge process
→ **010-zs-ecosystem/001-console-spec.md** — Console specification: the Builder is launched from the Console
→ **010-zs-ecosystem/002-marketplace-spec.md** — Marketplace specification: templates created in the Builder are published to the Marketplace
→ **006-zs-infrastructure/002-github-org-architecture.md** — GitHub organization architecture: the commit targets for Builder outputs
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 1 (GitHub is the government) and Law 6 (no-code first)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
