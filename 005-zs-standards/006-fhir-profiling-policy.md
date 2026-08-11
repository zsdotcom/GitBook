# 006-fhir-profiling-policy.md
## FHIR profiling policy
### When to create profiles, naming, authoring, validation — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all FHIR profile creation in the ZarishSphere Platform

---

## Table of contents

1. [Profile creation criteria](#1-profile-creation-criteria)
2. [Profile authoring tools](#2-profile-authoring-tools)
3. [Mandatory ZarishSphere extensions](#3-mandatory-zarishsphere-extensions)
4. [Profile naming convention](#4-profile-naming-convention)
5. [Terminology binding strength](#5-terminology-binding-strength)
6. [Profile validation in CI](#6-profile-validation-in-ci)

---

## 1. Profile creation criteria

### 1.1 When to create a profile

Create a ZarishSphere FHIR profile (a `StructureDefinition` resource) only when one or more of these conditions are met:

| Condition | Example |
|---|---|
| A FHIR resource needs domain-specific constraints | Adding required fields or restricting optional fields |
| A resource needs mandatory ZarishSphere extensions | Tenant ID, program code |
| A particular combination of terminology bindings is required for all uses | All diagnosis codes bound to ICD-11 with `required` strength |
| A resource needs country-specific or deployment-specific constraints | National patient identifier format |
| A workflow requires cross-resource consistency rules | Encounter must have a referenced Patient with matching jurisdiction |

### 1.2 When NOT to create a profile

Do NOT create a profile solely to document how a resource is used in a particular workflow. Use implementation guides instead.

| Use implementation guide | Use profile |
|---|---|
| Documenting recommended usage patterns | Enforcing mandatory constraints |
| Providing examples and narrative guidance | Defining cardinality rules and fixed values |
| Describing workflow sequences | Binding terminology with `required` strength |

---

## 2. Profile authoring tools

### 2.1 FHIR Shorthand and SUSHI

All ZarishSphere profiles are authored using FHIR Shorthand (FSH) and compiled with SUSHI.

```bash
# Install SUSHI
npm install -g fsh-sushi

# Compile profiles in the current directory
sushi .
```

### 2.2 Repository structure

Profiles are maintained in the `zs-data-fhir-profiles` repository:

```
zs-data-fhir-profiles/
├── input/
│   ├── fsh/
│   │   ├── profiles/       ← FSH source files per resource type
│   │   ├── extensions/     ← ZarishSphere extension definitions
│   │   └── valuesets/      ← ZarishSphere value set definitions
│   └── pages/              ← Implementation guide narrative
├── output/                  ← SUSHI-generated JSON StructureDefinitions
├── sushi-config.yaml        ← SUSHI configuration
└── .github/workflows/       ← CI validation pipeline
```

> **Constraint:** All profiles must be authored in FSH and compiled via SUSHI. Hand-authored JSON `StructureDefinition` resources are not permitted. FSH source ensures traceability and reviewability.

---

## 3. Mandatory ZarishSphere extensions

### 3.1 Extension definitions

All ZarishSphere-profiled resources MUST include the tenant extension. The program extension is required for multi-program deployments.

```fsh
Extension: ZSTenantExtension
Id: zs-tenant
Title: "ZarishSphere Tenant ID"
Description: "The organization or facility tenant identifier for multi-tenant isolation."
* value[x] only string

Extension: ZSProgramExtension
Id: zs-program
Title: "ZarishSphere Program"
Description: "The program context identifier (e.g., refugee-response, nutrition-surveillance)."
* value[x] only string
```

### 3.2 Extension usage in profiles

```fsh
Profile: ZSPatientProfile
Parent: Patient
Id: zs-patient
Title: "ZarishSphere Patient Profile"
Description: "Patient resource with ZarishSphere mandatory extensions."
* extension contains ZSTenantExtension named tenant 1..1
* extension contains ZSProgramExtension named program 0..1
```

---

## 4. Profile naming convention

### 4.1 Name patterns

| Pattern | Purpose | Example |
|---|---|---|
| `ZS{ResourceType}Profile` | Base ZarishSphere profile for a resource | `ZSPatientProfile` |
| `ZS{CC}{ResourceType}Profile` | Country-specific adaptation | `ZSBDPatientProfile` |
| `ZS{CC}{REGION}{ResourceType}Profile` | Region-specific adaptation | `ZSBDCXBPatientProfile` |

### 4.2 Profile ID and URL

```
Profile ID:   zs-{resource-type-abbreviation}
Canonical URL: https://zarishsphere.com/fhir/StructureDefinition/zs-{resource-type-abbreviation}
FHIR `url`:   https://zarishsphere.com/fhir/StructureDefinition/zs-{resource-type-abbreviation}
```

### 4.3 Example profiles

| Profile | ID | Usage |
|---|---|---|
| `ZSPatientProfile` | `zs-patient` | Base patient — all deployments |
| `ZSObservationProfile` | `zs-observation` | Base observation — all deployments |
| `ZSBDPatientProfile` | `zs-bd-patient` | Bangladesh-specific patient constraints |

---

## 5. Terminology binding strength

### 5.1 Binding strength table

Use these binding strengths for coded fields across all ZarishSphere profiles:

| Field type | Primary system | Binding strength |
|---|---|---|
| Diagnosis codes | ICD-11 or SNOMED CT | `required` |
| Observation codes | LOINC | `required` |
| Clinical findings | SNOMED CT | `required` |
| Medication codes | RxNorm | `preferred` |
| Vaccine codes | CVX | `required` |
| Units of measure | UCUM | `required` |
| Administrative gender | FHIR AdministrativeGender ValueSet | `required` |
| Resource status codes | FHIR ValueSet per resource | `required` |
| Encounter type / class | FHIR ValueSet | `preferred` |
| Country codes | ISO 3166-1 alpha-2 | `required` |
| Language codes | ISO 639-1 / BC-47 | `required` |

### 5.2 Binding level meanings

| Binding strength | Meaning |
|---|---|
| `required` | The coded value must be taken from the specified value set. No other codes are permitted. |
| `preferred` | The coded value should be taken from the specified value set if available. Other codes are permitted if the concept is not covered. |
| `example` | The specified value set is illustrative. Any code may be used. |

---

## 6. Profile validation in CI

### 6.1 CI validation pipeline

All profiles in `zs-data-fhir-profiles` are validated in CI using the HL7 FHIR Validator (HAPI-based).

```yaml
# .github/workflows/validate-fhir-profiles.yml
- name: Validate FHIR profiles
  run: |
    java -jar validator_cli.jar \
      -version 5.0.0 \
      -ig output/ \
      -recurse \
      resources/**/*.json
```

### 6.2 Validation checks

| Check | Description | Failure action |
|---|---|---|
| Profile conformance | Validates against FHIR R5 `StructureDefinition` schema | PR blocked |
| Extension resolution | All referenced extensions must exist in the IG | PR blocked |
| Value set resolution | All bound value sets must resolve to valid FHIR resources | PR blocked |
| Cross-reference integrity | Profile must not reference non-existent profiles | PR blocked |
| SUSHI compilation | FSH source must compile without errors | PR blocked |

### 6.3 Local validation

```bash
# Validate a single profile output
java -jar validator_cli.jar \
  -version 5.0.0 \
  output/StructureDefinition-zs-patient.json

# Expected output
# Summary: 0 error(s), 0 warning(s)
```

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/005-fhir-r5-conventions.md** — Resource-level conventions (IDs, tenant tags, audit events)
→ **005-zs-standards/007-fhir-search-standards.md** — Search parameter standards for profiled resources
→ **005-zs-standards/016-terminology-governance.md** — Terminology governance for binding decisions
→ **003-zs-platform/005-fhir-architecture.md** — FHIR engine architecture consuming these profiles

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
