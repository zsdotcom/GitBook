# 005-no-code-tools.md
## No-code and low-code tooling specification
### Offline-capable declarative extensibility via YAML/JSON, workflow DSL, and ZSM module packaging

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Architecture overview](#1-architecture-overview)
2. [Declarative form definitions](#2-declarative-form-definitions)
3. [Workflow DSL](#3-workflow-dsl)
4. [Module packaging format (ZSM)](#4-module-packaging-format-zsm)
5. [Domain expert workflow](#5-domain-expert-workflow)
6. [Builder integration](#6-builder-integration)
7. [Marketplace packaging](#7-marketplace-packaging)
8. [Plane 0 offline operation](#8-plane-0-offline-operation)
9. [Cross-references](#9-cross-references)

---

## 1. Architecture overview

The no-code tooling layer enables domain experts — clinicians, logistics coordinators, education administrators — to create, modify, and deploy ecosystem assets without writing code. All assets are defined in declarative YAML or JSON formats.

### 1.1 Design principles

| Principle | Description |
|---|---|
| Declarative only | No custom scripting language. YAML and JSON are the only authoring formats. |
| Offline capable | All no-code tools work on Plane 0 with local filesystem access. |
| Schema validated | Every definition is validated against a JSON Schema before acceptance. |
| FHIR compatible | Forms and workflows map to FHIR Questionnaire and PlanDefinition. |
| GUI + file | Assets can be created in the Builder GUI or written as plain files. |

### 1.2 No-code asset types

| Asset type | Description | Format | Rendered by |
|---|---|---|---|
| Form definition | Data collection form | YAML / JSON | Forms engine |
| Workflow definition | Step-by-step process | YAML DSL | Workflow engine |
| Module manifest | Package descriptor (ZSM) | YAML | Module loader |
| Dashboard definition | Report layout | YAML | Dashboard renderer |
| Validation rule | Field-level constraint | YAML | Forms engine |
| Export template | Output transformation | YAML / Go template | Export engine |

### 1.3 Layer diagram

```
┌──────────────────────────────────────────────────────┐
│                    Domain Expert                      │
│           (creates forms, workflows, modules)         │
├──────────────────────────────────────────────────────┤
│                        │                              │
│              ┌─────────┴─────────┐                    │
│              │   Builder GUI     │    Plain text      │
│              │  (drag-and-drop)  │    editor          │
│              └─────────┬─────────┘    (YAML/JSON)     │
│                        │                              │
├────────────────────────┼──────────────────────────────┤
│              ┌─────────┴─────────┐                    │
│              │  YAML / JSON      │                    │
│              │  Definitions      │                    │
│              └─────────┬─────────┘                    │
│                        │                              │
│              ┌─────────┴─────────┐                    │
│              │  Schema Validator │                    │
│              └─────────┬─────────┘                    │
│                        │                              │
├────────────────────────┼──────────────────────────────┤
│              ┌─────────┴─────────┐                    │
│              │  Runtime Engine   │                    │
│              │  (Go + React)     │                    │
│              └───────────────────┘                    │
│                                                        │
│  Forms Engine    Workflow Engine    Module Loader      │
│  (React 19.2.8)  (Go)              (Go)                │
└──────────────────────────────────────────────────────┘
```

---

## 2. Declarative form definitions

### 2.1 Form definition format

Forms are defined in YAML following the ZARISH-STANDARDS form schema. Each form maps to a FHIR Questionnaire resource at runtime.

```yaml
# example-form.yml
zs-form:
  id: "zs-form-ncd-intake-v1"
  title: "NCD Intake Assessment"
  description: "Initial intake form for non-communicable disease patients"
  domain: health
  version: "1.0.0"
  
  metadata:
    author: "ZarishSphere Foundation"
    fhir_profile: "https://zarishsphere.com/fhir/StructureDefinition/zs-questionnaire"
    category: "clinical-intake"
    target_plane: [0, 1, 2, 3, 4]
  
  pages:
    - id: demographics
      title: "Patient demographics"
      fields:
        - id: full-name
          type: text
          label: "Full name"
          required: true
          fhir_path: "Patient.name[0].text"
          validation:
            min_length: 2
            max_length: 200
        
        - id: date-of-birth
          type: date
          label: "Date of birth"
          required: true
          fhir_path: "Patient.birthDate"
          validation:
            max_past_years: 120
        
        - id: sex
          type: select
          label: "Sex at birth"
          required: true
          fhir_path: "Patient.gender"
          options:
            - value: "male"
              label: "Male"
            - value: "female"
              label: "Female"
            - value: "other"
              label: "Other"
        
        - id: phone
          type: text
          label: "Phone number"
          required: false
          fhir_path: "Patient.telecom[0].value"
          validation:
            pattern: "^\\+?[0-9]{7,15}$"
    
    - id: vitals
      title: "Vital signs"
      fields:
        - id: systolic-bp
          type: number
          label: "Systolic blood pressure (mmHg)"
          required: true
          fhir_path: "Observation.valueQuantity.value"
          unit: "mmHg"
          fhir_code: "8480-6"
          validation:
            min: 50
            max: 300
        
        - id: diastolic-bp
          type: number
          label: "Diastolic blood pressure (mmHg)"
          required: true
          fhir_path: "Observation.valueQuantity.value"
          unit: "mmHg"
          fhir_code: "8462-4"
          validation:
            min: 30
            max: 200
        
        - id: blood-glucose
          type: number
          label: "Blood glucose (mmol/L)"
          required: false
          fhir_path: "Observation.valueQuantity.value"
          unit: "mmol/L"
          fhir_code: "2339-0"
          validation:
            min: 1
            max: 40
```

### 2.2 Supported field types

| Field type | HTML element | FHIR mapping | Validation support |
|---|---|---|---|
| text | `<input type="text">` | string, markdown | min/max length, regex pattern |
| number | `<input type="number">` | integer, decimal | min, max, step, decimal places |
| date | `<input type="date">` | date | min, max, max_past_years |
| datetime | `<input type="datetime-local">` | dateTime | min, max |
| select | `<select>` | CodeableConcept | required, multi-select |
| radio | `<input type="radio">` | CodeableConcept | required |
| checkbox | `<input type="checkbox">` | boolean, Coding | required |
| textarea | `<textarea>` | text (multi-line) | min/max length, rows |
| reference | Autocomplete search | Reference | resource type filter |
| group | `<fieldset>` | BackboneElement | nested fields |
| table | Dynamic row editor | List of BackboneElement | min/max rows |
| file | `<input type="file">` | Attachment | max size, allowed types |

### 2.3 Form validation schema

Validation rules are defined in the form YAML and enforced both client-side and server-side:

```yaml
validation:
  - field: systolic-bp
    rules:
      - type: required
        message: "Systolic BP is required"
      - type: range
        min: 50
        max: 300
        message: "Systolic BP must be between 50 and 300 mmHg"
      - type: conditional
        if: "age >= 60"
        then:
          - type: range
            min: 90
            max: 180
            message: "Systolic BP should be 90-180 for patients 60+"
```

---

## 3. Workflow DSL

### 3.1 Workflow definition format

Workflows are defined in YAML and executed by the Go-based workflow engine. A workflow consists of steps with conditions, transitions, and actions.

```yaml
# example-workflow.yml
zs-workflow:
  id: "zs-workflow-ncd-screening-v1"
  title: "NCD screening protocol"
  description: "Standard screening workflow for NCD patients"
  version: "1.0.0"
  
  triggers:
    - event: "form.submitted"
      filter:
        form_id: "zs-form-ncd-intake-v1"
  
  steps:
    - id: triage
      type: decision
      title: "BP triage"
      description: "Check blood pressure for immediate action"
      input:
        - source: "form.systolic-bp"
          as: "sbp"
        - source: "form.diastolic-bp"
          as: "dbp"
      conditions:
        - if: "sbp >= 180 || dbp >= 120"
          then: "step.emergency-referral"
        - if: "sbp >= 140 || dbp >= 90"
          then: "step.schedule-followup"
        - else: "step.routine-counseling"
    
    - id: emergency-referral
      type: action
      title: "Emergency referral"
      actions:
        - type: create-resource
          resource_type: ServiceRequest
          template:
            status: "active"
            intent: "plan"
            priority: "stat"
            code:
              coding:
                - system: "http://snomed.info/sct"
                  code: "306098008"
                  display: "Referral to hospital"
        - type: notify
          channel: "alert"
          message: "Emergency referral needed: SBP {{sbp}}, DBP {{dbp}}"
      transitions:
        next: "step.complete"
    
    - id: schedule-followup
      type: form
      title: "Schedule follow-up"
      form_id: "zs-form-followup-schedule-v1"
      transitions:
        next: "step.complete"
    
    - id: routine-counseling
      type: form
      title: "Lifestyle counseling"
      form_id: "zs-form-counseling-v1"
      transitions:
        next: "step.complete"
    
    - id: complete
      type: terminal
      title: "Workflow complete"
      actions:
        - type: update-resource
          resource_type: CarePlan
          status: "completed"
```

### 3.2 Workflow step types

| Step type | Purpose | Behavior |
|---|---|---|
| decision | Conditional branching | Evaluates conditions, routes to next step |
| form | Display and collect form data | Opens form, waits for submission |
| action | Execute automated action | Creates/updates resources, sends notifications |
| delay | Wait for a time period | Timer-based pause (minutes, hours, days) |
| webhook | Call external service | HTTP request to configured endpoint |
| subworkflow | Run another workflow | Chained workflow execution |
| terminal | End the workflow | Final step, cleanup actions |

### 3.3 Workflow expression language

Conditions use a simple expression syntax evaluated by the Go workflow engine:

| Expression | Example | Evaluates to |
|---|---|---|
| Field comparison | `sbp >= 140` | boolean |
| String equality | `gender == "male"` | boolean |
| Logical AND | `sbp >= 140 && dbp >= 90` | boolean |
| Logical OR | `age > 60 || bmi > 30` | boolean |
| In list | `risk_level in ["high", "very-high"]` | boolean |
| Date arithmetic | `days_since(last_visit) > 90` | boolean |

---

## 4. Module packaging format (ZSM)

### 4.1 ZSM structure

A ZarishSphere Module (ZSM) is a directory with a YAML manifest. Modules are distributed as `.zsm` archives (gzipped tar).

```
ncd-screening-module/
├── manifest.yml           # Required: module metadata
├── README.md              # Required: module documentation
├── forms/                 # Optional: form definitions
│   ├── ncd-intake.yml
│   └── followup.yml
├── workflows/             # Optional: workflow definitions
│   └── ncd-screening.yml
├── resources/             # Optional: static resources
│   └── images/
├── dashboards/            # Optional: dashboard templates
│   └── ncd-report.tmpl
├── config/                # Optional: default configuration
│   └── defaults.yml
└── i18n/                  # Optional: translations
    ├── en.yml
    └── bn.yml
```

### 4.2 Module manifest

```yaml
# manifest.yml
zs-module:
  id: "ZS-MOD-health-ncd"
  name: "NCD Screening Module"
  version: "1.0.0"
  type: "domain"
  domain: "health"
  
  description: "Non-communicable disease screening and management module"
  
  author:
    name: "ZarishSphere Foundation"
    email: "foundation@zarishsphere.com"
  
  license: "Apache 2.0"
  
  targets:
    planes: [0, 1, 2, 3, 4]
  
  dependencies:
    engine: ">=1.0.0"
    modules: []
  
  capabilities:
    forms: ["ncd-intake", "followup", "counseling"]
    workflows: ["ncd-screening"]
    resources:
      fhir_profiles:
        - "zs-patient"
        - "zs-observation"
        - "zs-condition"
  
  size_estimate: "2.4 MB"
  
  permissions:
    - "fhir:Patient:read,write"
    - "fhir:Observation:read,write"
    - "fhir:Condition:read,write"
    - "fhir:ServiceRequest:write"
```

### 4.3 Module lifecycle

```
1. Author creates module files (Builder GUI or text editor)
2. Package as .zsm archive
3. Distribute via Marketplace or USB/file transfer
4. Install: zs-cli module install ncd-screening.zsm
5. Validate: schema check + dependency resolution
6. Activate: engine loads forms, workflows, resources
7. Monitor: module usage tracked in analytics
8. Update: new version installed over existing (data preserved)
9. Remove: deactivate, archive configuration, preserve patient data
```

---

## 5. Domain expert workflow

### 5.1 How a domain expert creates without code

A domain expert (e.g., a nurse, logistics coordinator, or education administrator) can contribute to the ecosystem without writing any code.

**Path A: Builder GUI (recommended)**

```
1. Open Console → Builder app
2. Select asset type: Form, Workflow, or Module
3. Use visual editor:
   - Forms: drag-and-drop fields onto canvas
   - Workflows: connect step blocks in sequence
   - Modules: fill in manifest metadata, select components
4. Preview: test the form or workflow immediately
5. Save: saves as YAML definition in local filesystem
6. Export: package as .zsm or share YAML directly
7. Deploy: install into local or remote ZarishSphere instance
```

**Path B: Plain text editor**

```
1. Open any text editor (VS Code, Notepad, vim)
2. Write YAML following the published schema
3. Validate: use zs-cli validate form.yml
4. Deploy: zs-cli module install ./my-form.yml
5. The forms/workflows appear in Console immediately
```

### 5.2 Schema validation

Every definition is validated against a published JSON Schema before acceptance:

```bash
# Validate a form definition
zs-cli validate form.yml --schema=zs-form-schema-v1.json

# Validate a workflow
zs-cli validate workflow.yml --schema=zs-workflow-schema-v1.json

# Validate a module manifest
zs-cli validate manifest.yml --schema=zs-module-schema-v1.json
```

### 5.3 Domain expert skill requirements

| Skill level | Can do | Tools needed |
|---|---|---|
| Non-technical | Use pre-built forms and workflows | Console (browser) |
| Basic computer | Modify existing forms, add fields | Builder GUI |
| Domain specialist | Create new forms and simple workflows | Builder GUI |
| Technical enthusiast | Create workflows with conditions, write YAML | Builder GUI or text editor |
| Developer | Create modules, custom workflows, contribute to SDK | Full toolchain |

---

## 6. Builder integration

### 6.1 Builder application

The Builder is the GUI-based no-code creation tool for building forms, workflows, modules, and apps. It is a React 19.2.8 application served by the Console.

| Feature | Status | Description |
|---|---|---|
| Form designer | V1 | Drag-and-drop field layout, property editor, preview |
| Workflow designer | V1 | Visual step editor, conditional branching, simulation |
| Module packager | V1 | Select components, configure manifest, export .zsm |
| YAML editor | V1 | Syntax-highlighted text editor for advanced users |
| Schema validator | V1 | Real-time validation against published schemas |
| Preview mode | V1 | Test form/workflow before deployment |
| Template library | V1.1 | Pre-built form and workflow templates |

### 6.2 Builder architecture

```
Builder UI (React 19.2.8)
    │
    ├── Form Designer
    │   ├── Field palette (drag sources)
    │   ├── Canvas (drop target, layout)
    │   ├── Property panel (field configuration)
    │   └── Preview pane (live form render)
    │
    ├── Workflow Designer
    │   ├── Step palette
    │   ├── Canvas (node graph, connections)
    │   ├── Condition editor
    │   └── Simulation runner
    │
    └── Module Packager
        ├── Manifest editor
        ├── Component selector
        └── Export (.zsm archive)
```

### 6.3 Builder API

The Builder communicates with the backend via a small Go API:

```go
// Builder API endpoints
GET    /api/v1/schemas                  // Available form/workflow schemas
POST   /api/v1/validate                 // Validate a YAML definition
POST   /api/v1/preview/render           // Render a form preview
POST   /api/v1/preview/execute          // Run a workflow simulation
POST   /api/v1/package                  // Package as .zsm archive
GET    /api/v1/templates/:type          // List templates by type
```

→ **010-zs-ecosystem/003-builder-spec.md** — Complete Builder specification

---

## 7. Marketplace packaging

### 7.1 Marketplace listing format

Modules published to the Marketplace include a listing descriptor:

```yaml
# marketplace-listing.yml
zs-marketplace-listing:
  module_id: "ZS-MOD-health-ncd"
  name: "NCD Screening Module"
  version: "1.0.0"
  
  listing:
    category: "Health"
    tags: ["ncd", "hypertension", "diabetes", "screening"]
    description: "Complete NCD screening and management module for primary care"
    features:
      - "Patient intake form with vitals"
      - "BP-based triage workflow"
      - "Follow-up scheduling"
      - "Lifestyle counseling forms"
      - "Monthly report dashboard"
    
    screenshots:
      - "screenshots/intake-form.png"
      - "screenshots/dashboard.png"
    
    compatibility:
      min_engine_version: "1.0.0"
      planes: [0, 1, 2, 3, 4]
      languages: ["en", "bn"]
    
    stats:
      downloads: 0
      rating: 0
      active_installs: 0
    
    publisher:
      name: "ZarishSphere Foundation"
      verified: true
    
    license: "Apache 2.0"
    repository: "https://github.com/zsdotcom/zs-module-health-ncd"
```

### 7.2 Distribution channels

| Channel | Method | Plane support |
|---|---|---|
| Marketplace website | Download .zsm from browser | All planes (download first) |
| CLI install | `zs-cli marketplace install ZS-MOD-health-ncd` | Plane 2+ (needs internet) |
| USB transfer | Copy .zsm file to target machine | Plane 0, 1 |
| Git clone | Clone module repository | Plane 2+ |
| Pre-loaded distribution | Module included in distribution bundle | All planes |

---

## 8. Plane 0 offline operation

### 8.1 How no-code tools work offline

All no-code tooling is designed to function without network access:

| Capability | Plane 0 behavior |
|---|---|
| Build forms | Local file system read/write, YAML editor in browser |
| Build workflows | Local file system read/write |
| Package modules | Local directory → .zsm archive |
| Validate definitions | Local schema files (bundled with engine) |
| Preview forms | In-browser render using local React app |
| Export .zsm | Save to local filesystem |
| Install modules | Load from local .zsm file |
| Run forms/workflows | All in-memory, no network calls |

### 8.2 File system conventions

```
~/.zarishsphere/
├── forms/              # User-created form definitions
├── workflows/          # User-created workflow definitions
├── modules/            # Installed ZSM modules
├── marketplace/        # Downloaded .zsm archives
├── schemas/            # Validation schemas (bundled)
└── exports/            # Exported assets
```

### 8.3 Bundle distribution

For Plane 0 deployments, all no-code assets can be distributed via USB:

```
1. On an internet-connected machine:
   - Download modules from Marketplace
   - Download form templates
   
2. Copy to USB drive:
   zs-cli export-bundle --output=/media/usb/zs-bundle/
   
3. On air-gapped machine:
   zs-cli import-bundle --input=/media/usb/zs-bundle/
   
4. All forms, workflows, and modules are available offline.
```

### 8.4 Schema mirroring

Validation schemas are bundled with the engine binary:

```
/usr/local/share/zarishsphere/schemas/
├── zs-form-schema-v1.json
├── zs-workflow-schema-v1.json
├── zs-module-schema-v1.json
└── zs-marketplace-listing-schema-v1.json
```

---

## 9. Cross-references

→ **007-zs-tech-stack/001-tech-stack-master.md** — Master production tech stack mapping
→ **007-zs-tech-stack/003-frontend-stack.md** — Frontend framework and UI architecture (forms engine, Builder UI)
→ **010-zs-ecosystem/003-builder-spec.md** — Builder application specification
→ **010-zs-ecosystem/005-forms-spec.md** — Forms engine component specification
→ **010-zs-ecosystem/002-marketplace-spec.md** — Marketplace packaging and distribution specification
→ **005-zs-standards/013-form-schema-specification.md** — ZARISH-STANDARDS form schema
→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS documentation and formatting rules

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
