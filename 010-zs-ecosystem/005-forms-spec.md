# 005-forms-spec.md
## ZarishSphere Forms specification
### Dynamic form engine

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [How forms work](#2-how-forms-work)
3. [Form definition format](#3-form-definition-format)
4. [Offline operation](#4-offline-operation)
5. [Customization](#5-customization)
6. [Cross-references](#6-cross-references)

---

## 1. Purpose

ZarishSphere Forms is a dynamic form engine that generates browser-based forms directly from ZARISH-STANDARDS definitions. It is the primary data collection interface for the entire ecosystem.

The Forms engine depends on the form schema specification and the form validation rules maintained in ZARISH-STANDARDS. Every form definition must conform to the ZS form schema specification and pass the validation rules before it is accepted for rendering.

## 2. How forms work

1. A standard is indexed in ZARISH-INDEX (e.g., WHO nutrition protocol)
2. ZARISH-STANDARDS transforms it into a form definition
3. The Form Engine renders it as a browser-based form
4. Collected data populates the appropriate module
5. Forms can be customized via the Builder without code
6. Forms work offline on all planes

## 3. Form definition format

Forms are defined as FHIR Questionnaire resources:

```json
{
  "resourceType": "Questionnaire",
  "id": "zs-ncd-intake-v1",
  "status": "active",
  "item": [
    {
      "linkId": "blood-pressure",
      "text": "Blood Pressure",
      "type": "group",
      "item": [
        { "linkId": "systolic", "text": "Systolic", "type": "integer" },
        { "linkId": "diastolic", "text": "Diastolic", "type": "integer" }
      ]
    }
  ]
}
```

This is the canonical form definition format. XLSForm (KoboToolbox) import/export is supported for interoperability.

## 4. Offline operation

Forms work fully offline:
- Form definitions are cached locally on first load
- Submitted data is queued in local storage
- Sync occurs automatically when connectivity is available
- No data loss on connectivity interruption

## 5. Customization

Forms can be customized through the Builder:
- Add or remove fields
- Change field types
- Modify validation rules
- Update display logic
- Translate labels
- Change styling

All customizations are saved as new form versions. Original definitions remain available.

## 6. Cross-references

→ **005-zs-standards/013-form-schema-specification.md** — ZS form schema specification: the authoritative form structure, field types, and FHIRPath mapping every form must conform to
→ **005-zs-standards/014-form-validation-rules.md** — Form validation rules: automated CI validation and quality gates for form definitions
→ **010-zs-ecosystem/014-content-forms-spec.md** — zs-content-forms specification: the repository of FHIR R5 Questionnaire definitions the Forms engine consumes
→ **010-zs-ecosystem/012-engine-spec.md** — Engine specification: the Form engine sub-component that renders definitions
→ **010-zs-ecosystem/003-builder-spec.md** — Builder specification: the no-code customization tool for forms
→ **010-zs-ecosystem/004-apps-spec.md** — Apps specification: pre-built apps that ship form definitions
→ **003-zs-platform/003-deployment-planes.md** — Deployment plane model: offline form operation on all planes

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
