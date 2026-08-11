# 015-i18n-key-conventions.md
## Internationalization key conventions
### Key naming, locale files, translation workflow — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all i18n key management in the ZarishSphere Platform

---

## Table of contents

1. [Supported locales](#1-supported-locales)
2. [Key naming convention](#2-key-naming-convention)
3. [Translation file format](#3-translation-file-format)
4. [Adding a new translation](#4-adding-a-new-translation)
5. [Translation fallback strategy](#5-translation-fallback-strategy)

---

## 1. Supported locales

### 1.1 Current locale coverage

| Code | Language | Script | Status |
|---|---|---|---|
| `en` | English | Latin | Required — primary |
| `bn` | Bengali / Bangla | Bengali | Required |
| `my` | Burmese | Burmese | In progress |
| `ur` | Urdu | Arabic | Planned |
| `hi` | Hindi | Devanagari | Planned |
| `th` | Thai | Thai | Planned |
| `ar` | Arabic | Arabic | Planned |
| `fr` | French | Latin | Planned |

### 1.2 Minimum requirements

All forms MUST have `en` translations before merging. The second required locale is determined by the deployment context (e.g., `bn` for Bangladesh deployments). Other locales can be added progressively.

> **Constraint:** A form with missing translations for a required locale cannot be merged. The CI pipeline enforces this by checking that every i18n key in the form exists in all required locale files.

---

## 2. Key naming convention

### 2.1 Key hierarchy

```
{namespace}.{key}
```

### 2.2 Namespaces

| Namespace | Purpose | Example |
|---|---|---|
| `forms.{domain}` | Domain-specific form keys | `forms.maternity.anc_contact_number_label` |
| `forms.common` | Keys shared across all forms | `forms.common.date_of_birth_label` |
| `forms.options` | Shared option values | `forms.options.yes` |
| `units.{unit}` | Unit names and abbreviations | `units.kg` |
| `nav.{section}` | Navigation labels | `nav.dashboard` |
| `errors.{code}` | Error messages | `errors.required_field` |
| `alerts.{type}` | Alert and notification messages | `alerts.critical_low_muac` |

### 2.3 Key naming rules

| Rule | Example |
|---|---|
| Lowercase with underscores | `field_weight_label`, not `fieldWeightLabel` |
| Descriptive, hierarchical | `maternity.anc_contact_number_label` |
| Suffix indicates usage | `_label`, `_hint`, `_placeholder`, `_title`, `_description` |
| No inline values | Keys must reference translatable text, never contain actual display text |

### 2.4 Key examples

```
forms.maternity.anc_contact_number_label
forms.maternity.anc_contact_number_hint
forms.common.date_of_birth_label
forms.common.date_of_birth_hint
forms.options.yes
forms.options.no
forms.options.unknown
units.kg
units.cm
errors.required_field
errors.invalid_format
alerts.critical_low_muac
```

---

## 3. Translation file format

### 3.1 JSON structure

Translation files use a nested JSON structure matching the key hierarchy:

```json
// translations/en.json
{
  "forms": {
    "maternity": {
      "title": "Antenatal Care — First Contact",
      "section_1_title": "Maternal Information",
      "edd_label": "Expected Date of Delivery",
      "edd_hint": "Enter the date from ultrasound or LMP calculation",
      "gravida_label": "Number of pregnancies (Gravida)",
      "gravida_hint": "Include current pregnancy"
    },
    "common": {
      "date_of_birth_label": "Date of Birth",
      "date_of_birth_hint": "DD/MM/YYYY",
      "sex_label": "Sex",
      "weight_label": "Weight",
      "height_label": "Height"
    },
    "options": {
      "yes": "Yes",
      "no": "No",
      "unknown": "Unknown",
      "male": "Male",
      "female": "Female",
      "other": "Other"
    }
  },
  "units": {
    "kg": "kg",
    "g": "g",
    "cm": "cm",
    "mmhg": "mmHg",
    "celsius": "°C",
    "weeks": "weeks",
    "percent": "%"
  }
}
```

### 3.2 File location

Translation files are stored per repository:

```
{repo}/
├── translations/
│   ├── en.json          ← English (primary, always required)
│   ├── bn.json          ← Bengali
│   ├── my.json          ← Burmese
│   └── ...              ← Other locales
└── forms/
    └── ...
```

### 3.3 Key conventions in translation files

| Rule | Description |
|---|---|
| All keys lowercase | `forms.common.date_of_birth_label` |
| Underscores for hierarchy | Dots in the key = nesting in JSON |
| Values are display strings | Plain text, no HTML, no markup |
| Pluralisation | Use separate keys for singular/plural forms per locale |

---

## 4. Adding a new translation

### 4.1 Workflow

Adding a new translation follows this workflow:

1. Add all keys to `en.json` first
2. Create or update the target locale file (e.g., `bn.json`)
3. Ensure all keys in `en.json` have corresponding entries in the target file
4. Open a PR with both files
5. CI validates that all keys match between `en.json` and required locale files
6. Merge after CI passes

### 4.2 New locale addition

To add a new locale:

1. Create a new file: `translations/{locale-code}.json`
2. Copy the complete structure from `en.json`
3. Replace all values with translated text
4. Submit as a PR
5. CI validates structural parity with `en.json`

### 4.3 CI validation

```bash
# Validate translation parity
python3 tests/validate_translations.py

# Checks:
# - All keys in en.json exist in bn.json
# - No orphaned keys in bn.json that don't exist in en.json
# - All values in en.json are non-empty strings
```

---

## 5. Translation fallback strategy

### 5.1 Runtime fallback

When the platform renders a form or UI component, it resolves i18n keys in this order:

1. **Exact locale match** (e.g., `bn-BD`) — highest priority
2. **Base locale match** (e.g., `bn`) — if regional variant is missing
3. **English fallback** (`en`) — always present; the guaranteed fallback
4. **Key name display** — if even English is missing, display the key name as a diagnostic aid

### 5.2 Example resolution

```
Requested locale: bn-BD (Bangladesh Bengali)

1. Look for key in translations/bn-BD.json
2. Not found → look in translations/bn.json
3. Not found → look in translations/en.json
4. Not found → display "forms.common.date_of_birth_label" (key name)
```

> **Constraint:** The English locale file MUST contain every key used across all forms. If a key exists in only a non-English locale but not in English, it is a validation error and the form is blocked from merge.

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/013-form-schema-specification.md** — Form schema consuming i18n keys
→ **005-zs-standards/014-form-validation-rules.md** — CI validation of i18n key parity (Rules V-05, V-06, V-12)
→ **010-zs-ecosystem/005-forms-spec.md** — Forms engine component specification

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
