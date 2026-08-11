# 004-g2a-engine.md
## Guideline-to-Action Engine
### Six-stage pipeline for standard-to-asset transformation

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [What G2A is](#1-what-g2a-is)
2. [The six-stage pipeline](#2-the-six-stage-pipeline)
3. [Inputs and outputs](#3-inputs-and-outputs)
4. [Validation and quality](#4-validation-and-quality)
5. [Deployment pipeline](#5-deployment-pipeline)
6. [Plane 0 operation](#6-plane-0-operation)
7. [Cross-references](#7-cross-references)

---

## 1. What G2A is

The Guideline-to-Action (G2A) Engine is the primary technical innovation of the ZarishSphere ecosystem. It is the mechanism through which Law 3 of the constitution (Guideline-as-Code) is operationalised.

**Plain language:** Give it any indexed standard; it produces a deployable digital tool.

**Technical:** An open-source automation system that ingests a ZARISH-INDEX entry (via ZI-UID citation), processes it through ZARISH-STANDARDS transformation rules, and outputs deployable digital assets — forms, workflows, indicators, decision rules, SOPs.

### 1.1 What G2A solves

Standards exist as PDF documents. Workers need executable digital tools. The gap between them is not a knowledge problem but a systems problem. G2A eliminates this gap by making every standard machine-readable and auto-executable.

### 1.2 Legal basis

G2A does not modify or create standards. It transforms them. The original standard's copyright remains with its publisher. The transformation (form, workflow, decision rule) is a derivative work under the standard publisher's license. ZARISH-STANDARDS transformation rules are CC BY 4.0.

## 2. The six-stage pipeline

```
STAGE 1 — INGEST
  Input:   ZI-UID citation, PDF, URL, API
  Action:  Extract raw text and document structure
  Tools:   pdfplumber, python-docx, Trafilatura, Vision LLM
  Output:  Raw extracted text + document structure

STAGE 2 — PARSE
  Input:   Raw extracted text
  Action:  Parse into structured representation
  Tools:   Llama 3.1 8B (local) or Gemini 2.5 Flash (cloud, non-patient only)
  Output:  Structured JSON of the guideline

STAGE 3 — EXTRACT
  Input:   Structured JSON
  Action:  Generate all deployable assets
  Outputs:
    - FHIR R5 Questionnaire (forms)
    - FHIR R5 PlanDefinition (decision rules)
    - FHIR R5 ActivityDefinition (care activities)
    - DHIS2 DataSet + DataElement (indicators)
    - JSON Schema (data validation)
    - Markdown SOPs (training materials)
    - OCHA 5W template (donor reporting)
    - Budget model (cost estimation)

STAGE 4 — VALIDATE
  Action:  Validate all generated assets
  Tools:   HL7 FHIR R5 validator, JSON Schema Draft-2020-12
  Results: VALID / NEEDS_REVIEW / FAILED

STAGE 5 — STAGE
  Action:  Commit all assets as a GitHub PR
  Routing:
    Non-clinical: AUTO-MERGE
    Clinical:     PR held for review

STAGE 6 — DEPLOY
  Action:  PR merge triggers deployment
  Tools:   GitHub Actions → Argo CD → NATS broadcast
  Result:  Zero manual steps
```

## 3. Inputs and outputs

### 3.1 Input types

| Type | Source | Format |
|---|---|---|
| ZARISH-INDEX citation | ZI-UID | Machine-readable citation |
| PDF document | Standards body | PDF |
| Web page | Standards body | HTML |
| API | WHO SMART, UNHCR, etc. | JSON/XML |

### 3.2 Output types

| Asset | Format | Purpose |
|---|---|---|
| Forms | FHIR Questionnaire JSON | Data collection |
| Decision rules | FHIR PlanDefinition | Clinical decision support |
| Activities | FHIR ActivityDefinition | Care plan generation |
| Indicators | DHIS2 DataSet | M&E reporting |
| SOPs | Markdown | Training and reference |
| Supply lists | JSON Schema | Procurement planning |
| Reports | OCHA 5W template | Donor reporting |
| Budgets | JSON | Cost estimation |

## 4. Validation and quality

| Check | Tool | Failure action |
|---|---|---|
| FHIR conformance | HL7 FHIR validator | Flag for manual review |
| JSON Schema | Draft-2020-12 validator | Reject output |
| Clinical consistency | Automated contradiction check | Flag for clinical review |
| Cross-reference | All ZI-UIDs resolve | Reject until fixed |

## 5. Deployment pipeline

G2A outputs are deployed through the standard platform deployment pipeline:

1. **PR created** — all generated assets in a new branch
2. **Validation passes** — automated checks pass
3. **PR merged** — to main branch
4. **GitHub Actions triggered** — build and publish
5. **Argo CD syncs** — deployments update across all planes
6. **NATS broadcast** — all connected nodes receive new forms

## 6. Plane 0 operation

At Plane 0, G2A runs in batch mode:
- Standards are pre-loaded as SQLite-embedded form definitions
- No AI models (stage 2 parsing uses rule-based extraction)
- Forms are pre-rendered as static HTML/JS bundles
- New forms loaded via USB sync package

## 7. Cross-references

→ **010-zs-ecosystem/012-engine-spec.md** — Engine component specification: G2A, module, form, workflow runtimes
→ **004-zs-index/001-zarish-index-overview.md** — What ZARISH-INDEX is and the ZI-UID citation input to Stage 1
→ **004-zs-index/005-zarish-index-to-platform.md** — How INDEX feeds STANDARDS and the Platform (integration specification)
→ **003-zs-platform/005-fhir-architecture.md** — FHIR R5 formats the EXTRACT stage generates (Questionnaire, PlanDefinition, ActivityDefinition)
→ **003-zs-platform/001-platform-overview.md** — Platform context and where the Engine runs
→ **008-zs-adrs/005-adr-fhir-r5-over-r4.md** — ADR-005: FHIR R5 over R4, the conformance baseline for generated assets

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
