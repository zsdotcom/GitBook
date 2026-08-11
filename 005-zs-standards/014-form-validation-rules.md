# 014-form-validation-rules.md
## Form validation rules
### Automated CI validation, schema checks, quality gates — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all form validation in the ZarishSphere Platform CI pipeline

---

## Table of contents

1. [Overview](#1-overview)
2. [Schema validation rules](#2-schema-validation-rules)
3. [Content validation rules](#3-content-validation-rules)
4. [Translation validation rules](#4-translation-validation-rules)
5. [PII and security rules](#5-pii-and-security-rules)
6. [Running validation locally](#6-running-validation-locally)
7. [Common errors and fixes](#7-common-errors-and-fixes)

---

## 1. Overview

### 1.1 Enforcement

Every form in `zs-content-forms-*` repositories is automatically validated by the CI pipeline before it can be merged. This document describes all validation rules and how to fix common failures.

### 1.2 Validator agent

The validation pipeline is enforced by `zs-agent-content-validator`, which runs on every pull request. The validator performs checks in four categories: schema, content, translation, and security.

---

## 2. Schema validation rules

### 2.1 Rule V-01: Valid JSON syntax

**Rule:** The form file must be valid JSON.

**CI check:**
```bash
python3 -m json.tool form.json
```

**Fix:** Use a JSON validator or VS Code's built-in JSON validation. Verify no trailing commas, no unquoted keys, and valid string escaping.

---

### 2.2 Rule V-02: Validates against ZS form schema v1

**Rule:** The form must validate against the ZS form schema (→ **[005-zs-standards/013-form-schema-specification.md]**).

**CI check:**
```bash
jsonschema -i form.json schemas/zs-form-schema-v1.json
```

**Required fields at root:**

| Field | Validation |
|---|---|
| `$schema` | Must be `https://zarishsphere.com/schema/form/v1` |
| `id` | Must match pattern `zs-form-{domain}-{name}-v1` |
| `title` | Must be i18n key pattern `{{i18n:...}}` |
| `version` | Semantic version string (semver) |
| `fhirResource` | Valid FHIR R5 resource type |
| `sections` | Array with at least 1 section |

---

### 2.3 Rule V-07: Form ID follows naming convention

**Rule:** Form ID must match `^zs-form-[a-z]+-[a-z0-9-]+-v[0-9]+$`

```
✅ zs-form-anc-intake-v1
✅ zs-form-phq9-screening-v1
✅ zs-form-nutrition-screening-v1
❌ zs_form_anc_01  (underscores)
❌ ZS-FORM-ANC-01  (uppercase)
❌ zs-form-anc     (missing name and version suffix)
```

---

### 2.4 Rule V-08: Version is semantic

**Rule:** `version` must match semver format: `^[0-9]+\.[0-9]+\.[0-9]+$`

```
✅ 1.0.0
✅ 2.3.1
❌ 1.0     (missing patch)
❌ v1.0.0  (no 'v' prefix)
```

---

## 3. Content validation rules

### 3.1 Rule V-03: Every clinical field has a FHIR mapping

**Rule:** Every field of type `text`, `number`, `date`, `select`, `multiselect`, `boolean`, or `textarea` MUST have a `fhirPath` property.

```json
// ✅ Valid
{
  "id": "field-001",
  "type": "number",
  "label": "{{i18n:forms.domain.field_label}}",
  "fhirPath": "Observation.valueQuantity.value",
  "loincCode": "29463-7"
}

// ❌ Invalid — missing fhirPath
{
  "id": "field-001",
  "type": "number",
  "label": "{{i18n:forms.domain.field_label}}"
}
```

### 3.2 Rule V-04: Coded fields must have a terminology code

**Rule:** Fields with type `select` or `multiselect`, and all measurement fields (`number`, `integer`), MUST have at least one of: `loincCode`, `snomedCode`, `icd11Code`, `cvxCode`.

```json
// ✅ Valid — number field with LOINC
{ "type": "number", "loincCode": "8310-5", "fhirPath": "Observation.valueQuantity.value" }

// ❌ Invalid — measurement field with no terminology code
{ "type": "number", "fhirPath": "Observation.valueQuantity.value" }
```

### 3.3 Rule V-10: fhirPath is valid R5 path

**Rule:** The `fhirPath` value MUST be a valid FHIR R5 path for the resource declared in `fhirResource`.

**CI check:** Validated against FHIR R5 resource definitions. The validator maintains a registry of valid FHIRPath expressions per resource type.

### 3.4 Rule V-11: Option codes resolve to valid concepts

**Rule:** For `select` and `multiselect` fields, every option's `value` + `system` combination MUST resolve to a known concept in the referenced terminology system. The CI validator checks against a local cache of LOINC answer lists and other approved terminology sources.

---

## 4. Translation validation rules

### 4.1 Rule V-05: All labels are i18n keys

**Rule:** `label`, `placeholder`, and `hint` fields MUST use the pattern `{{i18n:{namespace}.{key}}}`. Inline text is not permitted.

```json
// ✅ Valid
{ "label": "{{i18n:forms.domain.field_label}}" }

// ❌ Invalid — inline English text
{ "label": "Patient Weight (kg)" }
```

### 4.2 Rule V-06: Translation keys exist

**Rule:** Every i18n key used in the form MUST exist in all required translation files.

**CI check:** Extracts all `{{i18n:...}}` keys from the form, checks against translation files in the repository.

The minimum required locales are `en` (English) and the primary deployment locale (e.g., `bn` for Bangladesh context). Other locales are validated when present.

### 4.3 Rule V-12: No orphaned translation keys

**Rule:** Every translation key in the locale files that is not shared across forms MUST be referenced by at least one form in the repository. Unused keys are reported as warnings.

---

## 5. PII and security rules

### 5.1 Rule V-09: No PHI as default values

**Rule:** No field's `default` property may contain real patient data or personally identifiable information.

**CI check:** Regex and pattern scan for:
- Name-like patterns (capitalized words in default values)
- Date of birth patterns (`\d{4}-\d{2}-\d{2}` or `\d{2}/\d{2}/\d{4}`)
- National ID number patterns (configurable per deployment country)
- Phone number patterns
- Email address patterns

```json
// ✅ Valid — no default
{ "id": "field-001", "type": "text", "default": null }

// ❌ Invalid — contains potential PII
{ "id": "field-001", "type": "text", "default": "John Doe" }
```

### 5.2 Rule V-13: No hardcoded credentials

**Rule:** Forms must not contain URLs, API keys, tokens, or any credential-like patterns in default values, validation rules, or display conditions.

---

## 6. Running validation locally

### 6.1 Local validation commands

```bash
# Navigate to the forms repository
cd zs-content-forms-core

# Install validator dependencies
pip3 install jsonschema

# Validate a single form
python3 tests/validate_forms.py forms/vitals/vitals-entry-form.json

# Validate all forms in the repository
python3 tests/validate_forms.py

# Expected output:
# ✅ forms/vitals/vitals-entry-form.json — VALID
# ❌ forms/lab/lab-order.json — INVALID: field-003 missing fhirPath
```

### 6.2 CI pipeline integration

```yaml
# .github/workflows/validate-forms.yml
- name: Validate all forms
  run: |
    python3 tests/validate_forms.py --strict
    python3 tests/validate_translations.py
    python3 tests/validate_pii.py
```

---

## 7. Common errors and fixes

| Error | Cause | Fix |
|---|---|---|
| `fhirPath missing on field-XXX` | Field has no FHIR mapping | Add `"fhirPath": "Resource.path"` to the field |
| `i18n key not in bn.json` | Bengali translation missing | Add key to `translations/bn.json` |
| `Invalid form ID format` | Wrong naming convention | Use `zs-form-{domain}-{name}-v1` |
| `loincCode required for measurement` | Missing LOINC code | Look up code at loinc.org |
| `Inline label text` | Using literal text instead of i18n key | Replace with `{{i18n:forms.x.y}}` |
| `Default value contains PII` | Default field has real data pattern | Remove or nullify default value |
| `Invalid FHIRPath` | FHIRPath not valid for declared resource | Check FHIR R5 resource definition |

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/013-form-schema-specification.md** — ZS Form Schema v1 definition
→ **005-zs-standards/015-i18n-key-conventions.md** — Translation key conventions
→ **005-zs-standards/016-terminology-governance.md** — Terminology code governance
→ **005-zs-standards/004-standards-to-platform-pipeline.md** — Enforcement pipeline applying validation at runtime

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
