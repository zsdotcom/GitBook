# 017-terminology-code-systems.md
## Supported terminology code systems
### ICD-11, SNOMED CT, LOINC, CIEL, RxNorm, CVX — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all code system usage in the ZarishSphere Platform

---

## Table of contents

1. [Code system overview](#1-code-system-overview)
2. [ICD-11](#2-icd-11)
3. [SNOMED CT](#3-snomed-ct)
4. [LOINC](#4-loinc)
5. [CIEL (OpenMRS)](#5-ciel-openmrs)
6. [RxNorm](#6-rxnorm)
7. [CVX](#7-cvx)

---

## 1. Code system overview

| System | Issuer | Type | Primary domain | Update frequency | Access |
|---|---|---|---|---|---|
| ICD-11 | WHO | Classification | Diagnoses | Annual (January) | Free REST API |
| SNOMED CT | SNOMED International | Ontology | Clinical findings | Monthly | Free for DPG implementers |
| LOINC | Regenstrief Institute | Terminology | Lab observations | Bi-annual (Feb/Aug) | Free CSV download |
| CIEL | OpenMRS | Terminology | OpenMRS concepts | Irregular | Open access |
| RxNorm | NLM / NIH | Terminology | Medications | Continuous | Free REST API |
| CVX | CDC | Code set | Vaccines | As needed | Free download |

---

## 2. ICD-11

### 2.1 Source and license

| Field | Detail |
|---|---|
| Issuer | World Health Organization (WHO) |
| Current version | ICD-11 2026-01 |
| License | Free for all purposes — WHO does not charge for ICD-11 use |
| Access method | WHO REST API: `https://id.who.int/icd/release/11/{year}/mms` |
| Repository | `zs-data-icd11` |

### 2.2 When to use ICD-11

Use ICD-11 codes in:

| FHIR resource | Field | Example |
|---|---|---|
| `Condition` | `code` | Diagnosis and problem list |
| `Observation` | `code` | When observation represents a condition (screening result) |
| `DiagnosticReport` | `conclusionCode` | Final diagnostic conclusion |
| `DocumentReference` | `type` | Document category |

### 2.3 Code format

ICD-11 uses alphanumeric codes with dot separators:

```
BA00     — Pulmonary tuberculosis
BA00.0   — Pulmonary tuberculosis, confirmed
BA00.Z   — Pulmonary tuberculosis, unspecified
5A10     — Diabetes mellitus
6A70     — Depressive episode
```

### 2.4 FHIR coding pattern

```json
{
  "code": {
    "coding": [
      {
        "system": "http://id.who.int/icd/release/11/2026-01/mms",
        "code": "BA00.0",
        "display": "Pulmonary tuberculosis, confirmed"
      }
    ],
    "text": "Pulmonary tuberculosis, confirmed"
  }
}
```

> **Constraint:** The `system` URI MUST include the ICD-11 version year: `http://id.who.int/icd/release/11/{YEAR}/mms`. Omitting the year creates ambiguity when the system is updated.

### 2.5 Cross-mapping

ZarishSphere provides ICD-11 ↔ ICD-10 cross-maps in `zs-data-concept-maps` for integration with legacy systems:

```
ICD-11 BA00.0 → ICD-10 A15.0 (Tuberculosis of lung, confirmed by microscopy)
ICD-11 5A10   → ICD-10 E11   (Type 2 diabetes mellitus)
```

---

## 3. SNOMED CT

### 3.1 Source and license

| Field | Detail |
|---|---|
| Issuer | SNOMED International |
| Current version | August 2026 International Edition |
| License | Free for low-income countries and DPG implementers (SNOMED International MSG) |
| Access method | Member license / DPG implementer license via SNOMED International |
| Repository | `zs-data-snomed` |

### 3.2 When to use SNOMED CT

Use SNOMED CT codes in:

| FHIR resource | Field | Use case |
|---|---|---|
| `Condition` | `code` | Clinical findings alongside ICD-11 |
| `Procedure` | `code` | Clinical procedures |
| `AllergyIntolerance` | `code` | Allergen substance codes |
| `Observation` | `code` | Clinical observation codes |
| CDS Hooks | `context` | Clinical decision support triggers |

### 3.3 FHIR coding pattern

```json
{
  "code": {
    "coding": [
      {
        "system": "http://snomed.info/sct",
        "code": "271737000",
        "display": "Anaemia (disorder)"
      },
      {
        "system": "http://id.who.int/icd/release/11/2026-01/mms",
        "code": "3A00",
        "display": "Iron deficiency anaemia"
      }
    ],
    "text": "Anaemia due to iron deficiency"
  }
}
```

### 3.4 Common codes

| Clinical concept | SNOMED code |
|---|---|
| Malnutrition | 248325000 |
| Severe acute malnutrition | 238131007 |
| Moderate acute malnutrition | 302872004 |
| Antenatal care | 424525001 |
| Postnatal care | 133906008 |
| Immunization (procedure) | 127785005 |
| Blood pressure taking | 75367002 |
| Weighing patient | 39857003 |

---

## 4. LOINC

### 4.1 Source and license

| Field | Detail |
|---|---|
| Issuer | Regenstrief Institute |
| Current version | LOINC 2.82 |
| License | Free — Creative Commons CC0 (public domain equivalent) |
| Access method | CSV download from https://loinc.org / FHIR CodeSystem |
| Repository | `zs-data-loinc` |

### 4.2 When to use LOINC

Use LOINC codes in:

| FHIR resource | Field | Use case |
|---|---|---|
| `Observation` | `code` | ALL clinical observations and measurements |
| `DiagnosticReport` | `code` | Lab panels and report types |
| `Questionnaire` | `item.code` | Questionnaire item identification |

### 4.3 FHIR coding pattern

```json
{
  "code": {
    "coding": [
      {
        "system": "http://loinc.org",
        "code": "8480-6",
        "display": "Systolic blood pressure"
      }
    ],
    "text": "Systolic blood pressure"
  },
  "valueQuantity": {
    "value": 120,
    "unit": "mmHg",
    "system": "http://unitsofmeasure.org",
    "code": "mm[Hg]"
  }
}
```

### 4.4 Mandatory LOINC codes

| Observation | LOINC code | Unit (UCUM) |
|---|---|---|
| Body weight | 29463-7 | kg |
| Height / Body height | 8302-2 | cm |
| Mid-upper arm circumference (MUAC) | 56072-2 | cm |
| Systolic blood pressure | 8480-6 | mm[Hg] |
| Diastolic blood pressure | 8462-4 | mm[Hg] |
| Heart rate | 8867-4 | /min |
| Respiratory rate | 9279-1 | /min |
| Body temperature | 8310-5 | Cel |
| Oxygen saturation (SpO2) | 59408-5 | % |
| Blood glucose | 15074-8 | mmol/L |
| Hemoglobin | 718-7 | g/dL |

All unit codes above follow UCUM version 2.2.

### 4.5 Panel codes

| Panel | Panel LOINC | Component LOINCs |
|---|---|---|
| Complete blood count (CBC) | 58410-2 | Hemoglobin 718-7, Hematocrit 20570-8, WBC 6690-2 |
| Blood glucose panel | 1558-6 | Glucose 1558-6 |

---

## 5. CIEL (OpenMRS)

### 5.1 Source and license

| Field | Detail |
|---|---|
| Issuer | OpenMRS Community |
| Current version | CIEL 2024 release |
| License | Open access — Creative Commons |
| Access method | CSV download / OpenMRS concept dictionary API |
| Repository | `zs-data-ciel` |

### 5.2 When to use CIEL

Use CIEL codes when integrating with OpenMRS instances or when concept coverage is needed that is not yet available in standard terminology systems.

| FHIR resource | Field | Use case |
|---|---|---|
| `Condition` | `code` | OpenMRS-integrated concepts |
| `Observation` | `code` | OpenMRS observations |
| `MedicationRequest` | `medication` | OpenMRS drug concepts |

### 5.3 FHIR coding pattern

```json
{
  "code": {
    "coding": [
      {
        "system": "https://openmrs.org/concepts/",
        "code": "5089",
        "display": "Weight (kg)"
      }
    ],
    "text": "Weight"
  }
}
```

---

## 6. RxNorm

### 6.1 Source and license

| Field | Detail |
|---|---|
| Issuer | National Library of Medicine (NLM), NIH |
| Current version | Monthly release |
| License | Free — U.S. Government public domain |
| Access method | REST API: `https://rxnav.nlm.nih.gov/REST/` |
| Repository | `zs-data-rxnorm` |

### 6.2 When to use RxNorm

Use RxNorm codes in:

| FHIR resource | Field | Use case |
|---|---|---|
| `MedicationRequest` | `medication` | Drug identification |
| `Medication` | `code` | Medication catalog |
| `MedicationDispense` | `medication` | Dispensed medication |

### 6.3 FHIR coding pattern

```json
{
  "medicationCodeableConcept": {
    "coding": [
      {
        "system": "http://www.nlm.nih.gov/research/umls/rxnorm",
        "code": "1049502",
        "display": "amoxicillin 500 MG oral capsule"
      }
    ],
    "text": "Amoxicillin 500mg capsule"
  }
}
```

---

## 7. CVX

### 7.1 Source and license

| Field | Detail |
|---|---|
| Issuer | Centers for Disease Control and Prevention (CDC) |
| Current version | CVX 2026-04-16 code set release (latest CDC publication) |
| License | Free — U.S. Government public domain |
| Access method | CSV download from CDC website / HL7 code system |
| Repository | `zs-data-cvx` |

### 7.2 When to use CVX

Use CVX codes in:

| FHIR resource | Field | Use case |
|---|---|---|
| `Immunization` | `vaccineCode` | Vaccine administered |
| `ImmunizationRecommendation` | `vaccineCode` | Recommended vaccine |

### 7.3 FHIR coding pattern

```json
{
  "vaccineCode": {
    "coding": [
      {
        "system": "http://hl7.org/fhir/sid/cvx",
        "code": "08",
        "display": "Hep B, adolescent or pediatric"
      }
    ],
    "text": "Hepatitis B vaccine"
  }
}
```

### 7.4 Common CVX codes

| Vaccine | CVX code |
|---|---|
| Hepatitis B, pediatric | 08 |
| OPV (oral polio) | 01 |
| IPV (inactivated polio) | 10 |
| BCG (TB vaccine) | 19 |
| DTP (diphtheria-tetanus-pertussis) | 20 |
| Measles | 05 |
| MR (measles-rubella) | 94 |
| Tetanus toxoid | 35 |

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/016-terminology-governance.md** — Governance policy: approved systems, update schedules, deprecation
→ **003-zs-platform/007-data-model.md** — Platform data model and identifier patterns
→ **007-zs-tech-stack/004-data-pipeline.md** — Terminology data pipeline for loading code systems

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
