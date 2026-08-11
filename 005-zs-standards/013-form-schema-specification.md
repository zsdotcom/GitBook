# 013-form-schema-specification.md
## ZS form schema specification v1
### Complete form structure, field types, FHIRPath mapping — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all form schema definitions in the ZarishSphere Platform

---

## Table of contents

1. [Overview](#1-overview)
2. [Complete form structure](#2-complete-form-structure)
3. [Field types](#3-field-types)
4. [Coded field option format](#4-coded-field-option-format)
5. [Display conditions](#5-display-conditions)
6. [Form metadata fields](#6-form-metadata-fields)

---

## 1. Overview

### 1.1 Purpose

ZS Form Schema v1 is the standard for all domain forms in the ZarishSphere Platform. Every form in `zs-content-forms-*` repositories MUST validate against this schema.

### 1.2 Schema guarantees

The schema ensures that every form in the platform is:

| Guarantee | Enforcement |
|---|---|
| Every field has a FHIR R5 mapping | `fhirPath` property required on all data fields |
| Every coded field has a terminology code | LOINC, SNOMED, ICD-11, or other approved code system reference |
| Every label is an i18n key | No inline text — all labels use `{{i18n:...}}` pattern |
| Forms are machine-processable | Structured JSON consumed by the form engine at runtime |

### 1.3 Schema reference

```
Schema URI:    https://zarishsphere.com/schema/form/v1
Schema file:   schemas/zs-form-schema-v1.json
Form engine:   zs-pkg-ui-form-engine
```

---

## 2. Complete form structure

### 2.1 Root-level form definition

```json
{
  "$schema": "https://zarishsphere.com/schema/form/v1",
  "id": "zs-form-{domain}-{name}-v1",
  "title": "{{i18n:forms.{domain}.title}}",
  "description": "{{i18n:forms.{domain}.description}}",
  "version": "1.0.0",
  "fhirResource": "Observation",
  "status": "active",
  "tags": ["domain", "workflow", "context"],
  "programs": ["program-identifier"],
  "sections": [
    {
      "id": "section-{n}",
      "title": "{{i18n:forms.{domain}.section_{n}_title}}",
      "description": "{{i18n:forms.{domain}.section_{n}_description}}",
      "repeating": false,
      "fields": [
        {
          "id": "field-{nnn}",
          "type": "number",
          "label": "{{i18n:forms.{domain}.field_{nnn}_label}}",
          "hint": "{{i18n:forms.{domain}.field_{nnn}_hint}}",
          "placeholder": "{{i18n:forms.{domain}.field_{nnn}_placeholder}}",
          "fhirPath": "Observation.valueQuantity.value",
          "fhirResource": "Observation",
          "loincCode": "29463-7",
          "loincDisplay": "Body weight",
          "unit": "kg",
          "ucumUnit": "kg",
          "required": true,
          "readOnly": false,
          "hidden": false,
          "validation": {
            "min": 0,
            "max": 300,
            "decimalPlaces": 1
          },
          "displayCondition": null
        }
      ]
    }
  ],
  "logic": [],
  "calculatedFields": []
}
```

### 2.2 Root-level field requirements

| Field | Required | Description |
|---|---|---|
| `$schema` | Yes | Must be `https://zarishsphere.com/schema/form/v1` |
| `id` | Yes | Must follow `zs-form-{domain}-{name}-v1` pattern |
| `title` | Yes | Must be an i18n key (`{{i18n:...}}`) |
| `description` | No | i18n key for descriptive text |
| `version` | Yes | Semantic version string |
| `fhirResource` | Yes | Valid FHIR R5 resource type the form maps to |
| `status` | Yes | `active`, `draft`, `deprecated`, `retired` |
| `sections` | Yes | Array — at least 1 section required |

---

## 3. Field types

### 3.1 Supported types

| Type | Description | FHIR output | Validation |
|---|---|---|---|
| `text` | Short free-text input | `valueString` | `maxLength`, `pattern` |
| `textarea` | Multi-line text | `valueString` | `maxLength` |
| `number` | Numeric input | `valueQuantity` | `min`, `max`, `decimalPlaces` |
| `integer` | Whole number | `valueInteger` | `min`, `max` |
| `date` | Date picker | `valueDateTime` | `minDate`, `maxDate` |
| `datetime` | Date + time | `valueDateTime` | `minDate`, `maxDate` |
| `select` | Single choice (dropdown) | `valueCoding` | `options` (coded list) |
| `multiselect` | Multiple choices | `valueCodeableConcept[]` | `options` (coded list) |
| `boolean` | Yes/No toggle | `valueBoolean` | — |
| `scale` | Likert scale | `valueInteger` | `min`, `max`, `labels` |
| `signature` | Digital signature | `Attachment` | — |
| `photo` | Camera capture | `Attachment` | `maxSizeMB` |
| `gps` | GPS coordinates | `Extension(geolocation)` | — |
| `barcode` | Barcode scanner | `valueString` | `pattern` |

### 3.2 Required field properties per type

| Field type | Required additional properties |
|---|---|
| `text`, `textarea` | `fhirPath`, `validation.maxLength` |
| `number`, `integer` | `fhirPath`, `validation.min`, `validation.max` |
| `date`, `datetime` | `fhirPath` |
| `select`, `multiselect` | `fhirPath`, `options` array |
| `boolean` | `fhirPath` |
| `photo` | `validation.maxSizeMB` |

> **Constraint:** Every field that captures data MUST have a `fhirPath` property. Fields without a FHIR mapping are not permitted. The `fhirPath` must be a valid FHIR R5 path for the resource declared in `fhirResource`.

---

## 4. Coded field option format

### 4.1 Option structure

Options for coded fields (`select`, `multiselect`) MUST reference standard terminology:

```json
{
  "type": "select",
  "id": "field-001",
  "fhirPath": "Observation.valueCodeableConcept.coding[0]",
  "options": [
    {
      "value": "LA19263-5",
      "display": "{{i18n:forms.options.yes}}",
      "system": "http://loinc.org"
    },
    {
      "value": "LA32-8",
      "display": "{{i18n:forms.options.no}}",
      "system": "http://loinc.org"
    },
    {
      "value": "LA4489-6",
      "display": "{{i18n:forms.options.unknown}}",
      "system": "http://loinc.org"
    }
  ]
}
```

### 4.2 Option field requirements

| Field | Required | Description |
|---|---|---|
| `value` | Yes | Terminology code value |
| `display` | Yes | i18n key for the display label |
| `system` | Yes | Terminology system URI |

### 4.3 Terminology systems for options

| System | URI | Used for |
|---|---|---|
| LOINC answer list | `http://loinc.org` | Yes/No/Unknown, Likert scales |
| FHIR administrative gender | `http://hl7.org/fhir/administrative-gender` | Gender selection |
| ISO 3166-1 | `urn:iso:std:iso:3166` | Country selection |
| Custom ZarishSphere | `https://zarishsphere.com/CodeSystem/{name}` | Program-specific codes |

---

## 5. Display conditions

### 5.1 Conditional field visibility

Fields can be conditionally shown or hidden based on other field values:

```json
{
  "id": "field-005",
  "label": "{{i18n:forms.domain.detail_field}}",
  "displayCondition": {
    "fieldId": "field-004",
    "operator": "equals",
    "value": "trigger-value"
  }
}
```

### 5.2 Supported operators

| Operator | Behaviour | Data type |
|---|---|---|
| `equals` | Show when field value equals the specified value | text, select, boolean |
| `notEquals` | Show when field value does not equal the specified value | text, select, boolean |
| `greaterThan` | Show when numeric value exceeds threshold | number, integer |
| `lessThan` | Show when numeric value is below threshold | number, integer |
| `contains` | Show when text value contains substring | text, textarea |
| `notEmpty` | Show when field has any value | all types |

### 5.3 Logical groups

```json
{
  "displayCondition": {
    "logicalOperator": "AND",
    "conditions": [
      { "fieldId": "field-001", "operator": "equals", "value": "yes" },
      { "fieldId": "field-002", "operator": "greaterThan", "value": 18 }
    ]
  }
}
```

---

## 6. Form metadata fields

### 6.1 Form identity

| Field | Pattern | Example |
|---|---|---|
| `id` | `zs-form-{domain}-{name}-v1` | `zs-form-anc-intake-v1` |
| `version` | semver `X.Y.Z` | `1.0.0` |
| `status` | `active`, `draft`, `deprecated`, `retired` | `active` |

### 6.2 Form tags and programs

| Field | Description |
|---|---|
| `tags` | Array of free-form tags for categorization (`["maternity", "anc", "core"]`) |
| `programs` | Array of program identifiers the form belongs to (`["refugee-response"]`) |

### 6.3 Calculated fields and logic

Forms MAY include `calculatedFields` for auto-computed values and `logic` for cross-field validation rules:

```json
{
  "calculatedFields": [
    {
      "id": "field-calc-001",
      "label": "{{i18n:forms.domain.bmi}}",
      "formula": "field-003 / ((field-004/100) * (field-004/100))",
      "fhirPath": "Observation.valueQuantity.value"
    }
  ],
  "logic": [
    {
      "ruleId": "rule-001",
      "condition": {
        "fieldId": "field-010",
        "operator": "equals",
        "value": "yes"
      },
      "action": "require",
      "targetField": "field-011"
    }
  ]
}
```

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/014-form-validation-rules.md** — Validation rules applied to form submissions
→ **005-zs-standards/015-i18n-key-conventions.md** — i18n key pattern used by form labels
→ **005-zs-standards/006-fhir-profiling-policy.md** — FHIR resource mapping targets for forms
→ **010-zs-ecosystem/005-forms-spec.md** — Ecosystem forms component specification

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
