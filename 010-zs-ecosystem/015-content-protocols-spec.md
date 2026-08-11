# 015-content-protocols-spec.md
## zs-content-protocols specification
### Protocol definitions repository

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Repository structure](#2-repository-structure)
3. [Protocol definition format](#3-protocol-definition-format)
4. [Protocol lifecycle](#4-protocol-lifecycle)
5. [Relation to G2A Engine](#5-relation-to-g2a-engine)
6. [Plane 0 operation](#6-plane-0-operation)
7. [Cross-references](#7-cross-references)

---

## 1. Purpose

The `zs-content-protocols` repository stores, versions, and distributes machine-readable protocol definitions — clinical guidelines, operational SOPs, decision trees, and workflow templates. These protocols are consumed by the G2A Engine to transform standards into executable actions.

Key design principles:

- **Machine-readable first** — protocols are structured data, not prose
- **Standards-aligned** — each protocol traces to a ZARISH-STANDARDS entry
- **Executable** — protocols drive automated decision support and workflow orchestration
- **Versioned immutably** — every change creates a new protocol version

## 2. Repository structure

```
zs-content-protocols/
├── protocols/
│   ├── health/
│   │   ├── ncd-screening-v1.yaml
│   │   ├── malnutrition-treatment-v2.yaml
│   │   └── immunization-schedule-v1.yaml
│   ├── protection/
│   │   ├── child-protection-intake-v1.yaml
│   │   └── gbv-clinical-care-v2.yaml
│   ├── wash/
│   │   └── water-quality-testing-v1.yaml
│   └── cross-domain/
│       ├── referral-guidelines-v1.yaml
│       └── consent-procedure-v1.yaml
├── workflows/
│   ├── approval-chains/
│   │   ├── emergency-funding-approval-v1.yaml
│   │   └── supply-request-approval-v2.yaml
│   └── notification-rules/
│       ├── critical-lab-alert-v1.yaml
│       └── referral-trigger-v1.yaml
├── decision-trees/
│   ├── triage-protocol-v1.yaml
│   ├── tb-screening-algorithm-v2.yaml
│   └── malnutrition-classification-v1.yaml
├── schemas/
│   ├── protocol-schema.json
│   └── workflow-schema.json
├── examples/
│   └── ncd-screening-complete.yaml
└── tests/
    ├── validate-protocols.go
    └── test-fixtures/
```

### 2.1 Directory conventions

| Directory | Purpose |
|---|---|
| `protocols/{domain}/` | Protocol definitions organized by domain taxonomy |
| `workflows/` | Operational workflow and approval chain definitions |
| `decision-trees/` | Decision support algorithms and triage protocols |
| `schemas/` | JSON Schema for protocol YAML validation |
| `examples/` | Complete working examples |
| `tests/` | Automated validation and integration tests |

## 3. Protocol definition format

Protocols are defined in YAML using a structured format that aligns with the FHIR R5 PlanDefinition and ActivityDefinition resources.

### 3.1 Core protocol structure

```yaml
# ncd-screening-v1.yaml
protocol:
  id: zs-ncd-screening-v1
  name: NCD Screening Protocol
  version: 1.0.0
  status: active
  domain: health
  source:
    standard: WHO PEN
    zarish-index-id: ZS-HLT-0014
    zarish-standards-id: ZS-STD-HLT-0014

  triggers:
    - event: patient-registration
      condition: age >= 40
      action: initiate-screening

  steps:
    - id: step-1
      name: Blood Pressure Measurement
      description: Measure blood pressure using automated device
      actor: clinician
      inputs:
        - systolic: integer
        - diastolic: integer
      outputs:
        - bp-category: choice
      rules:
        - if: systolic >= 180 OR diastolic >= 120
          then: set bp-category = "hypertensive-crisis"
          action: urgent-referral
        - if: systolic >= 140 OR diastolic >= 90
          then: set bp-category = "hypertensive"
          action: initiate-treatment
        - if: systolic >= 130 OR diastolic >= 85
          then: set bp-category = "pre-hypertensive"
          action: lifestyle-counseling
        - else: set bp-category = "normal"

    - id: step-2
      name: Blood Glucose Test
      description: Random blood glucose measurement
      actor: clinician
      inputs:
        - glucose-mgdl: integer
      rules:
        - if: glucose-mgdl >= 200
          then: action: diabetes-referral

  output:
    - risk-score: calculated
    - recommendations: list
```

### 3.2 Workflow definition

```yaml
# emergency-funding-approval-v1.yaml
workflow:
  id: zs-emergency-funding-approval-v1
  name: Emergency Funding Approval
  version: 1.0.0

  states:
    - name: submitted
      initial: true
    - name: manager-review
    - name: finance-review
    - name: approved
    - name: rejected

  transitions:
    - from: submitted
      to: manager-review
      action: submit-request
    - from: manager-review
      to: finance-review
      condition: amount <= 50000
      action: approve-manager
    - from: manager-review
      to: rejected
      condition: amount > 50000
    - from: finance-review
      to: approved
      action: approve-finance
    - from: finance-review
      to: rejected
      action: reject-finance
```

### 3.3 Decision tree definition

```yaml
# triage-protocol-v1.yaml
decision-tree:
  id: zs-triage-protocol-v1
  name: Emergency Triage Protocol
  version: 1.0.0

  root:
    question: Is the patient breathing?
    answers:
      - value: "no"
        action: immediate-resuscitation
        next:
          question: Is breathing restored?
          answers:
            - value: "yes"
              outcome: urgent-care
            - value: "no"
              outcome: deceased
              notify: manager
      - value: "yes"
        next:
          question: Is the patient conscious?
          answers:
            - value: "no"
              outcome: urgent-care
            - value: "yes"
              next:
                question: Trauma present?
                answers:
                  - value: "yes"
                    outcome: trauma-bay
                  - value: "no"
                    outcome: general-queue
```

## 4. Protocol lifecycle

```
[Draft] → [Review] → [Published] → [Deprecated]
```

| Stage | Description | Criteria |
|---|---|---|
| Draft | Initial development | In progress, not ready for production |
| Review | Under domain expert review | PR submitted, awaiting approval |
| Published | Ready for production | Reviewed, tested, and merged |
| Deprecated | Superseded | Newer version available |

### 4.1 Versioning

Protocols follow semantic versioning tied to the source standard version. When a referenced standard is updated (e.g., WHO releases new PEN guidelines), a new protocol version is created.

> **Constraint:** The `latest` tag must never be used. Consumers must pin to a specific semantic version.

### 4.2 Standards traceability

Every published protocol must include:

- `source.standard` — name of the source standard or guideline
- `source.zarish-index-id` — corresponding ZARISH-INDEX entry
- `source.zarish-standards-id` — corresponding ZARISH-STANDARDS entry

This traceability chain ensures every executable protocol is grounded in an indexed standard.

## 5. Relation to G2A Engine

The G2A Engine (Guideline-to-Action) consumes protocols from `zs-content-protocols` to:

1. **Transform standards** — convert ZARISH-STANDARDS entries into executable protocol steps
2. **Drive decision support** — evaluate protocol rules against patient data
3. **Orchestrate workflows** — manage approval chains and state transitions
4. **Generate alerts** — trigger notifications based on protocol conditions

The pipeline is:

```
ZARISH-INDEX → ZARISH-STANDARDS → zs-content-protocols → G2A Engine → Console/Forms
```

## 6. Plane 0 operation

At Plane 0, protocols ship as pre-loaded bundles:

- All protocols for the deployed distribution are bundled at deployment time
- Decision trees execute entirely locally — no network calls
- Workflow state is tracked in the local engine state store
- Protocol updates arrive via sync bundles from higher planes
- AI-dependent protocol features are disabled

## 7. Cross-references

→ **010-zs-ecosystem/012-engine-spec.md** — Engine specification: the G2A Engine sub-component that executes these protocols
→ **003-zs-platform/004-g2a-engine.md** — G2A Engine: the platform-side architecture of the guideline-to-action pipeline
→ **010-zs-ecosystem/010-modules-spec.md** — Modules specification: protocols are packaged and shipped inside domain modules
→ **010-zs-ecosystem/016-content-templates-spec.md** — zs-content-templates specification: bundles protocols into distribution packages
→ **010-zs-ecosystem/017-content-reports-spec.md** — zs-content-reports specification: compliance reports over protocol execution
→ **010-zs-ecosystem/014-content-forms-spec.md** — zs-content-forms specification: sibling content repository with the same lifecycle model
→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework: the ZARISH-STANDARDS entries each protocol traces to
→ **004-zs-index/001-zarish-index-overview.md** — ZARISH-INDEX overview: the indexed standards cited in `source.zarish-index-id`

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
