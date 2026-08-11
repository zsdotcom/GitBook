# 003-standards-schema.md
## ZARISH-STANDARDS — Standards Schema
### JSON Schema · YAML Schema · Validation Rules · Enum Registries · V1

**Document type:** Specification — Canonical
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative. All ZS-* standard definitions must conform to this schema.

---

## Table of Contents

1. [Schema Design Goals](#1-schema-design-goals)
2. [JSON Schema for ZS-* Standard Definitions](#2-json-schema-for-zs--standard-definitions)
3. [Schema for Standard Type Classifications](#3-schema-for-standard-type-classifications)
4. [Mandatory Field Validation Rules](#4-mandatory-field-validation-rules)
5. [Enum Value Registries](#5-enum-value-registries)
6. [Schema Versioning Approach](#6-schema-versioning-approach)
7. [Example Validation Outputs](#7-example-validation-outputs)

---

## 1. Schema Design Goals

The ZARISH-STANDARDS schema is designed to serve two primary consumers: the automated transformation pipeline and the G2A Engine runtime. Every design decision balances machine-readability with human debuggability.

### 1.1 Design principles

| Principle | Meaning in practice |
|---|---|
| **Machine-first** | Primary validation is automated. Schemas are published as JSON Schema (Draft 2020-12) and YAML Schema for direct consumption by CI/CD and the G2A Engine. |
| **Human-readable** | Field names are descriptive. Error messages include plain-language explanations alongside machine codes. |
| **Strict validation** | All mandatory fields must be present and valid at the point of entry. No deferred validation. |
| **Forward-compatible** | Schema versioning follows semantic versioning. New fields are additive — removal of fields requires a major version bump. |
| **Self-describing** | Each ZS-* record carries its own schema version reference, enabling the platform to validate records against the correct schema version. |

### 1.2 Supported schema formats

| Format | Purpose | Location |
|---|---|---|
| JSON Schema (Draft 2020-12) | Primary — used by transformation pipeline and G2A Engine | `schemas/zarish-standards/standard.json` |
| YAML Schema | YAML-based standards registry files | `schemas/zarish-standards/standard.yaml` |
| TypeScript types | Type definitions for Console and Builder UI components | `schemas/zarish-standards/standard.d.ts` |

---

## 2. JSON Schema for ZS-* Standard Definitions

The canonical schema for a ZS-* standard definition is defined in JSON Schema Draft 2020-12.

### 2.1 Root schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schemas.zarishsphere.com/zarish-standards/standard.v1.json",
  "title": "ZARISH-STANDARDS Standard Definition",
  "description": "Schema for a ZS-* standard definition in the ZARISH-STANDARDS registry.",
  "type": "object",
  "required": [
    "zs_id",
    "zarish_id",
    "name_full",
    "name_short",
    "standard_type",
    "governance_scope",
    "domain",
    "sub_domain",
    "lifecycle_status",
    "issuer",
    "issuer_type",
    "year_published",
    "mandate_level",
    "enforcement_rules",
    "deployment_planes"
  ],
  "properties": {
    "zs_id": {
      "type": "string",
      "pattern": "^ZS-[A-Z]{2,5}-[A-Z0-9]+-[0-9]{4}$",
      "description": "Unique ZARISH-STANDARDS identifier"
    },
    "zarish_id": {
      "type": "string",
      "pattern": "^[A-Z]{2,5}-[A-Z]+-[A-Z0-9-]+-[0-9]{4}$",
      "description": "Source ZARISH-INDEX identifier"
    },
    "name_full": {
      "type": "string",
      "minLength": 1,
      "description": "Complete official standard name"
    },
    "name_short": {
      "type": "string",
      "minLength": 1,
      "description": "Short display name or acronym"
    },
    "standard_type": {
      "type": "string",
      "enum": ["TYPE-A", "TYPE-B", "TYPE-C", "UNVERIFIED"],
      "description": "Three-type framework classification"
    },
    "governance_scope": {
      "type": "string",
      "enum": ["GLOBAL", "REGIONAL", "NATIONAL", "HUMANITARIAN"],
      "description": "Geographic scope of governance authority"
    },
    "domain": {
      "type": "string",
      "enum": [
        "Health and Medical",
        "Food Safety and Nutrition",
        "Animal Health and Veterinary",
        "Pharmaceuticals and Medical Devices",
        "Occupational Health and Safety",
        "Plant Health and Agriculture",
        "Metrology and Measurement",
        "Manufacturing and Industrial",
        "Electrical and Electronics",
        "Construction and Civil Engineering",
        "Chemical and Process",
        "Materials and Metallurgy",
        "Aerospace and Aviation",
        "Space and Satellite",
        "Human Rights",
        "Labour and Employment",
        "Humanitarian and Emergency Response",
        "Legal and Justice",
        "Governance and Public Administration",
        "Education and Training",
        "Culture and Cultural Heritage",
        "Sports and Recreation",
        "Finance, Banking and Accounting",
        "Trade and Commerce",
        "Supply Chain and Logistics",
        "Sustainability, ESG and Circular Economy",
        "Taxation and Public Finance",
        "Information and Communication Technology",
        "Cybersecurity and Data Protection",
        "Artificial Intelligence and Data Governance",
        "Energy and Power",
        "Transport and Mobility",
        "Water, Sanitation and Hygiene",
        "Urban Planning and Infrastructure",
        "Defence and Security",
        "Climate and Meteorology",
        "Marine and Maritime",
        "Biodiversity and Ecology",
        "Disaster Risk Reduction and Resilience",
        "Extractives and Mining"
      ],
      "description": "Canonical domain from the 40-domain taxonomy"
    },
    "sub_domain": {
      "type": "string",
      "minLength": 1,
      "description": "Sub-category within the canonical domain"
    },
    "lifecycle_status": {
      "type": "string",
      "enum": ["ACTIVE", "BETA", "DEPRECATED", "RETIRED", "PENDING"],
      "description": "Current lifecycle status in the platform"
    },
    "issuer": {
      "type": "string",
      "minLength": 1,
      "description": "Name of the issuing standards body"
    },
    "issuer_type": {
      "type": "string",
      "enum": [
        "UN Agency",
        "Treaty Body",
        "ISO",
        "IEC",
        "ITU",
        "Industry SDO",
        "Professional Body",
        "NGO",
        "Intergovernmental",
        "National Government",
        "Regional Body"
      ],
      "description": "Classification of the issuing body"
    },
    "year_published": {
      "type": "integer",
      "minimum": 1850,
      "maximum": 2030,
      "description": "Year of current edition"
    },
    "year_first": {
      "type": "integer",
      "minimum": 1850,
      "maximum": 2030,
      "description": "Year first published"
    },
    "mandate_level": {
      "type": "string",
      "enum": ["MANDATORY", "RECOMMENDED", "OPTIONAL", "REFERENCE"],
      "description": "Enforcement level in the ZarishSphere Platform"
    },
    "enforcement_rules": {
      "type": "object",
      "properties": {
        "validation_type": {
          "type": "string",
          "enum": ["strict", "warning", "informational"]
        },
        "block_on_failure": {
          "type": "boolean"
        },
        "audit_required": {
          "type": "boolean"
        },
        "custom_rules": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "rule_id": { "type": "string" },
              "description": { "type": "string" },
              "severity": { "type": "string", "enum": ["error", "warning", "info"] }
            },
            "required": ["rule_id", "severity"]
          }
        }
      },
      "required": ["validation_type", "block_on_failure", "audit_required"],
      "description": "Platform-level enforcement configuration"
    },
    "g2a_ruleset_ref": {
      "type": "string",
      "description": "Reference to G2A ruleset in the platform, if applicable"
    },
    "display_priority": {
      "type": "integer",
      "minimum": 1,
      "maximum": 100,
      "description": "UI display priority (higher = more prominent)"
    },
    "localisation_rules": {
      "type": "object",
      "properties": {
        "display_name": {
          "type": "string",
          "description": "Localised display name for the UI"
        },
        "preferred_terminology": {
          "type": "string",
          "description": "Preferred local terminology for field labels"
        },
        "language_overrides": {
          "type": "object",
          "description": "Language-specific field value mappings"
        }
      },
      "description": "Localisation and display rules for the ZarishSphere UI"
    },
    "replaces": {
      "type": "string",
      "pattern": "^ZS-[A-Z]{2,5}-[A-Z0-9]+-[0-9]{4}$",
      "description": "ZS-* ID of the prior version this standard replaces"
    },
    "replaced_by": {
      "type": "string",
      "pattern": "^ZS-[A-Z]{2,5}-[A-Z0-9]+-[0-9]{4}$",
      "description": "ZS-* ID of the successor standard"
    },
    "dependencies": {
      "type": "array",
      "items": {
        "type": "string",
        "pattern": "^ZS-[A-Z]{2,5}-[A-Z0-9]+-[0-9]{4}$"
      },
      "description": "Array of ZS-* IDs this standard depends on"
    },
    "domain_modules": {
      "type": "array",
      "items": {
        "type": "string",
        "pattern": "^ZS-MOD-[a-z]+$"
      },
      "description": "Domain modules that consume this standard"
    },
    "deployment_planes": {
      "type": "array",
      "items": {
        "type": "integer",
        "minimum": 0,
        "maximum": 4
      },
      "minItems": 1,
      "uniqueItems": true,
      "description": "Deployment planes this standard applies to (0-4)"
    },
    "official_url": {
      "type": "string",
      "format": "uri",
      "pattern": "^https://",
      "description": "Primary source URL from the issuing body"
    },
    "why_it_matters": {
      "type": "string",
      "minLength": 10,
      "description": "Plain-language significance explanation"
    },
    "data_source": {
      "type": "string",
      "minLength": 1,
      "description": "Provenance record"
    },
    "transformation_log": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "event_type": {
            "type": "string",
            "enum": ["CREATE", "UPDATE", "DEPRECATE", "RETIRE", "OVERRIDE"]
          },
          "zarish_version": { "type": "string" },
          "timestamp": { "type": "string", "format": "date-time" },
          "trigger": { "type": "string" },
          "gate_results": {
            "type": "object",
            "properties": {
              "gate_1": { "type": "string", "enum": ["PASS", "FAIL"] },
              "gate_2": { "type": "string", "enum": ["PASS", "FAIL"] },
              "gate_3": { "type": "string", "enum": ["PASS", "FAIL"] },
              "gate_4": { "type": "string", "enum": ["PASS", "FAIL"] },
              "gate_5": { "type": "string", "enum": ["PASS", "FAIL"] }
            }
          },
          "operator": { "type": "string" }
        },
        "required": ["event_type", "timestamp", "trigger", "gate_results", "operator"]
      },
      "description": "Audit trail of transformation events"
    },
    "notes": {
      "type": "string",
      "description": "Additional contextual information"
    }
  }
}
```

### 2.2 Schema validation rules summary

| Rule | Target | Check |
|---|---|---|
| S01 | `zs_id` | Must match regex `^ZS-[A-Z]{2,5}-[A-Z0-9]+-[0-9]{4}$` |
| S02 | `zarish_id` | Must match regex `^[A-Z]{2,5}-[A-Z]+-[A-Z0-9-]+-[0-9]{4}$` |
| S03 | `standard_type` | Must be one of TYPE-A, TYPE-B, TYPE-C, UNVERIFIED |
| S04 | `governance_scope` | Must be one of GLOBAL, REGIONAL, NATIONAL, HUMANITARIAN |
| S05 | `domain` | Must be one of the 40 canonical domain names |
| S06 | `lifecycle_status` | Must be one of ACTIVE, BETA, DEPRECATED, RETIRED, PENDING |
| S07 | `mandate_level` | Must be one of MANDATORY, RECOMMENDED, OPTIONAL, REFERENCE |
| S08 | `deployment_planes` | Must contain at least one value; all values in 0-4 range |
| S09 | `official_url` | Must be a valid HTTPS URL |
| S10 | `why_it_matters` | Must be at least 10 characters |
| S11 | `display_priority` | Must be an integer between 1 and 100 |
| S12 | No duplicate `zs_id` across the registry | Ensures identifier uniqueness |

---

## 3. Schema for Standard Type Classifications

Each standard type (TYPE-A, TYPE-B, TYPE-C) has a sub-schema that validates the type-specific fields.

### 3.1 TYPE-A classification sub-schema

TYPE-A standards define classifications, taxonomies, code systems, and ontologies. Additional required fields:

```json
{
  "type_specific": {
    "classification_type": {
      "type": "string",
      "enum": ["taxonomy", "code-system", "ontology", "classification", "terminology"],
      "description": "Type of classification system"
    },
    "code_count": {
      "type": "integer",
      "minimum": 0,
      "description": "Approximate number of codes or entries in the system"
    },
    "hierarchy_depth": {
      "type": "integer",
      "minimum": 1,
      "maximum": 10,
      "description": "Maximum depth of the classification hierarchy"
    },
    "has_linearization": {
      "type": "boolean",
      "description": "Whether the classification has a simplified linearization"
    },
    "api_available": {
      "type": "boolean",
      "description": "Whether a free machine-readable API is available"
    },
    "fhir_codesystem_url": {
      "type": "string",
      "format": "uri",
      "description": "FHIR CodeSystem URL if available"
    }
  }
}
```

### 3.2 TYPE-B exchange sub-schema

TYPE-B standards define data exchange formats, APIs, protocols, and messaging standards. Additional required fields:

```json
{
  "type_specific": {
    "exchange_format": {
      "type": "string",
      "enum": [
        "FHIR-R5", "FHIR-R4", "HL7-v2", "openEHR", "DICOM",
        "JSON-API", "GraphQL", "REST", "gRPC",
        "XML-Schema", "SOAP", "EDIFACT", "ISO-20022",
        "GeoJSON", "OGC-API",
        "OAuth-2.0", "OpenID-Connect", "OpenAPI-3.x",
        "IETF-RFC", "W3C-Standard"
      ],
      "description": "The data exchange format or protocol"
    },
    "version": {
      "type": "string",
      "description": "Specific version of the exchange standard"
    },
    "mime_types": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Supported MIME types"
    },
    "transport": {
      "type": "string",
      "enum": ["HTTP", "MQTT", "WebSocket", "gRPC", "SMTP", "FTP", "AMQP"],
      "description": "Primary transport protocol"
    },
    "auth_required": {
      "type": "boolean",
      "description": "Whether authentication is required for exchange"
    }
  }
}
```

### 3.3 TYPE-C management sub-schema

TYPE-C standards define operational processes, governance frameworks, and management systems. Additional required fields:

```json
{
  "type_specific": {
    "management_focus": {
      "type": "string",
      "enum": [
        "quality-management",
        "clinical-protocol",
        "governance-framework",
        "operational-guideline",
        "humanitarian-standard",
        "compliance-framework",
        "audit-standard",
        "reporting-framework",
        "environmental-management",
        "social-protection"
      ],
      "description": "Area of management or governance"
    },
    "certification_model": {
      "type": "string",
      "enum": ["third-party-certification", "self-declaration", "no-certification", "not-applicable"],
      "description": "Whether and how conformance is certified"
    },
    "audit_frequency": {
      "type": "string",
      "enum": ["annual", "biannual", "triennial", "not-specified", "not-applicable"],
      "description": "Recommended audit frequency"
    },
    "implementation_guide_url": {
      "type": "string",
      "format": "uri",
      "description": "URL to implementation guidance"
    }
  }
}
```

---

## 4. Mandatory Field Validation Rules

### 4.1 Mandatory fields

The following 15 fields are mandatory for every ZS-* record:

| # | Field | Validation | Error code |
|---|---|---|---|
| 1 | `zs_id` | Regex pattern match | E-M001 |
| 2 | `zarish_id` | Regex pattern match | E-M002 |
| 3 | `name_full` | Non-empty string | E-M003 |
| 4 | `name_short` | Non-empty string | E-M004 |
| 5 | `standard_type` | Enum value match | E-M005 |
| 6 | `governance_scope` | Enum value match | E-M006 |
| 7 | `domain` | Enum value match | E-M007 |
| 8 | `sub_domain` | Non-empty string | E-M008 |
| 9 | `lifecycle_status` | Enum value match | E-M009 |
| 10 | `issuer` | Non-empty string | E-M010 |
| 11 | `issuer_type` | Enum value match | E-M011 |
| 12 | `year_published` | Integer in range 1850-2030 | E-M012 |
| 13 | `mandate_level` | Enum value match | E-M013 |
| 14 | `enforcement_rules` | Object — must contain validation_type, block_on_failure, audit_required | E-M014 |
| 15 | `deployment_planes` | Non-empty array of integers 0-4 | E-M015 |

### 4.2 Conditional mandatory fields

These fields become mandatory based on the value of another field:

| Field | Condition | Error code |
|---|---|---|
| `g2a_ruleset_ref` | Required if `standard_type` is TYPE-B or TYPE-C | E-C001 |
| `replaces` | Required if this standard supersedes a prior version | E-C002 |
| `replaced_by` | Required if `lifecycle_status` is DEPRECATED or RETIRED | E-C003 |
| `dependencies` | Required if the standard references other standards for implementation | E-C004 |
| `domain_modules` | Required if any deployment plane is assigned | E-C005 |
| `type_specific` | Required for all standard types; schema varies by TYPE-A/B/C | E-C006 |

### 4.3 Validation error format

All validation errors follow this structure:

```json
{
  "error_code": "E-M001",
  "severity": "error",
  "field": "zs_id",
  "value": "ZS-HL-ICD11",
  "expected": "pattern: ^ZS-[A-Z]{2,5}-[A-Z0-9]+-[0-9]{4}$",
  "message": "The zs_id field must include a 4-digit year suffix (e.g., ZS-HL-ICD11-2025)",
  "record": "ZS-HL-ICD11-2025"
}
```

---

## 5. Enum Value Registries

### 5.1 standard_type

```
TYPE-A       Classification or taxonomy standard
TYPE-B       Exchange or protocol standard
TYPE-C       Management or governance standard
UNVERIFIED   Type not yet confirmed — requires curator review
```

### 5.2 governance_scope

```
GLOBAL         Issued by an international body for worldwide application
REGIONAL       Issued by a regional body for a defined group of countries
NATIONAL       Issued by a national body for a single country
HUMANITARIAN   Issued by the humanitarian standards community
```

### 5.3 lifecycle_status

```
ACTIVE      Currently enforced; G2A bindings active
BETA        Released for testing; not for production use
DEPRECATED  Being phased out; legacy data only
RETIRED     No longer available; removed from active pipeline
PENDING     Not yet released; awaiting curator review
```

### 5.4 mandate_level

```
MANDATORY   Field values must conform; validation enforced; blocks on failure
RECOMMENDED Guidance displayed; non-conformance logged but not blocked
OPTIONAL    Available but not preferred; no validation enforcement
REFERENCE   Informational only; no validation or enforcement
```

### 5.5 issuer_type

```
UN Agency           WHO, ILO, ICAO, IMO, IAEA, UNESCO, FAO, UNHCR, etc.
Treaty Body         OHCHR treaty committees, UNFCCC Secretariat, etc.
ISO                 ISO and all technical committees
IEC                 IEC and all technical committees
ITU                 ITU-T, ITU-R, ITU-D
Industry SDO        IEEE, IETF, W3C, ASTM, ASME, API, NFPA, SAE, OASIS, ECMA
Professional Body   ICOM, ICOMOS, ICCROM, WADA, IOC, FIFA, World Athletics
NGO                 Sphere Project, CHS Alliance, INEE, GRI, CALP
Intergovernmental   OECD, WTO, WCO, FSB, BCBS, BIS, FATF, IOSCO
National Government NIST, BSI, DIN, BSTI, ANSI
Regional Body       CEN/CENELEC, ETSI, ARSO, SARSO, ASEAN Standards
```

### 5.6 enforcement_rules.validation_type

```
strict          Hard validation — records that fail enforcement cannot be saved
warning         Soft validation — warning displayed but record can be saved
informational   Validation runs silently — logged for analytics
```

### 5.7 transformation_log.event_type

```
CREATE      New ZS-* record created from transformation
UPDATE      Existing ZS-* record updated from re-transformation
DEPRECATE   Record lifecycle changed to DEPRECATED
RETIRE      Record lifecycle changed to RETIRED
OVERRIDE    Validation gate overridden by curator
```

---

## 6. Schema Versioning Approach

### 6.1 Schema version format

The schema version follows semantic versioning: `v[MAJOR].[MINOR].[PATCH]`

| Component | Increment when |
|---|---|
| MAJOR | A field is removed, renamed, or made mandatory (breaking change) |
| MINOR | A field is added, enum values are extended, or optional fields added |
| PATCH | Description or documentation changes only |

### 6.2 Schema evolution rules

- **Backward-compatible changes** (MINOR bump): Adding new fields, extending enum registries. Older consumers continue to work with existing data.
- **Breaking changes** (MAJOR bump): Removing fields, changing field types, making optional fields mandatory. All consumers must upgrade.
- **No destructive changes without deprecation:** Fields that are being removed must first be deprecated for at least one minor version cycle before being removed in a major version.

### 6.3 Schema file naming

```
standard.v1.json      Current major version
standard.v1.1.json    Current minor version (if extended)
standard.v2.json      Next major version (if breaking)
```

The schema reference is stored inline in each ZS-* record as a `$schema` property referencing the canonical URL:

```
https://schemas.zarishsphere.com/zarish-standards/standard.v1.json
```

### 6.4 Schema distribution

Schemas are distributed via:
- The `zs-zarish-standards` repository in `/schemas/zarish-standards/`
- A dedicated GitHub Pages site: `https://schemas.zarishsphere.com/zarish-standards/`
- The G2A Engine's embedded schema registry for offline (Plane 0) validation

---

## 7. Example Validation Outputs

### 7.1 Valid record — ICD-11

```json
{
  "zs_id": "ZS-HL-ICD11-2026",
  "zarish_id": "HL-WHO-ICD11-2026",
  "name_full": "International Classification of Diseases 11th Revision (2026)",
  "name_short": "ICD-11",
  "standard_type": "TYPE-A",
  "governance_scope": "GLOBAL",
  "domain": "Health and Medical",
  "sub_domain": "Clinical Classification",
  "lifecycle_status": "ACTIVE",
  "issuer": "World Health Organization",
  "issuer_type": "UN Agency",
  "year_published": 2026,
  "year_first": 2018,
  "mandate_level": "MANDATORY",
  "enforcement_rules": {
    "validation_type": "strict",
    "block_on_failure": true,
    "audit_required": true
  },
  "g2a_ruleset_ref": "g2a:rs:hl-icd11-2026",
  "display_priority": 95,
  "deployment_planes": [0, 1, 2, 3, 4],
  "official_url": "https://icd.who.int/browse/2026-01",
  "why_it_matters": "ICD-11 is the global standard for diagnostic coding in health systems. Every clinical diagnosis in ZarishSphere must be coded using ICD-11 codes for interoperability with international health systems.",
  "data_source": "ZARISH-INDEX release v1.0.0; manual curator review 2026-06-01",
  "transformation_log": [
    {
      "event_type": "CREATE",
      "zarish_version": "1.0.0",
      "timestamp": "2026-06-10T08:00:00Z",
      "trigger": "batch-pipeline-release-v1.0.0",
      "gate_results": {
        "gate_1": "PASS",
        "gate_2": "PASS",
        "gate_3": "PASS",
        "gate_4": "PASS",
        "gate_5": "PASS"
      },
      "operator": "system"
    }
  ],
  "notes": "Replaces ICD-10 as mandatory classification from 2025. WHO has extended free API access for low-income countries."
}
```

**Validation result:**

```json
{
  "valid": true,
  "record_id": "ZS-HL-ICD11-2026",
  "schema_version": "v1.0.0",
  "timestamp": "2026-06-10T08:00:00Z",
  "errors": [],
  "warnings": []
}
```

### 7.2 Invalid record — missing required field

```json
{
  "zs_id": "ZS-HL-ICD11-2026",
  "zarish_id": "HL-WHO-ICD11-2026",
  "standard_type": "TYPE-A",
  "governance_scope": "GLOBAL",
  "domain": "Health and Medical",
  "name_short": "ICD-11"
}
```

**Validation result:**

```json
{
  "valid": false,
  "record_id": "ZS-HL-ICD11-2026",
  "schema_version": "v1.0.0",
  "timestamp": "2026-06-10T08:01:00Z",
  "errors": [
    {
      "error_code": "E-M003",
      "severity": "error",
      "field": "name_full",
      "value": null,
      "expected": "non-empty string",
      "message": "The name_full field is required and must contain the complete official standard name",
      "record": "ZS-HL-ICD11-2026"
    },
    {
      "error_code": "E-M009",
      "severity": "error",
      "field": "lifecycle_status",
      "value": null,
      "expected": "ACTIVE | BETA | DEPRECATED | RETIRED | PENDING",
      "message": "The lifecycle_status field is required",
      "record": "ZS-HL-ICD11-2026"
    },
    {
      "error_code": "E-M013",
      "severity": "error",
      "field": "mandate_level",
      "value": null,
      "expected": "MANDATORY | RECOMMENDED | OPTIONAL | REFERENCE",
      "message": "The mandate_level field is required",
      "record": "ZS-HL-ICD11-2026"
    },
    {
      "error_code": "E-M014",
      "severity": "error",
      "field": "enforcement_rules",
      "value": null,
      "expected": "object with validation_type, block_on_failure, audit_required",
      "message": "The enforcement_rules field is required for all ZS-* records",
      "record": "ZS-HL-ICD11-2026"
    },
    {
      "error_code": "E-M015",
      "severity": "error",
      "field": "deployment_planes",
      "value": null,
      "expected": "non-empty array of integers 0-4",
      "message": "At least one deployment plane must be specified",
      "record": "ZS-HL-ICD11-2026"
    }
  ],
  "warnings": []
}
```

### 7.3 Invalid record — enum value mismatch

```json
{
  "zs_id": "ZS-HL-TEST-2026",
  "lifecycle_status": "ARCHIVED",
  "mandate_level": "STRICT"
}
```

**Validation result:**

```json
{
  "valid": false,
  "errors": [
    {
      "error_code": "E-M009",
      "severity": "error",
      "field": "lifecycle_status",
      "value": "ARCHIVED",
      "expected": "ACTIVE | BETA | DEPRECATED | RETIRED | PENDING",
      "message": "ARCHIVED is not a valid lifecycle_status. Valid values are: ACTIVE, BETA, DEPRECATED, RETIRED, PENDING",
      "record": "ZS-HL-TEST-2026"
    },
    {
      "error_code": "E-M013",
      "severity": "error",
      "field": "mandate_level",
      "value": "STRICT",
      "expected": "MANDATORY | RECOMMENDED | OPTIONAL | REFERENCE",
      "message": "STRICT is not a valid mandate_level. Valid values are: MANDATORY, RECOMMENDED, OPTIONAL, REFERENCE",
      "record": "ZS-HL-TEST-2026"
    }
  ]
}
```

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Strategic direction and three-type framework
→ **005-zs-standards/002-transformation-model.md** — Transformation pipeline that produces ZS-* records
→ **004-zs-index/003-metadata-schema.md** — Upstream ZARISH-INDEX schema
→ **005-zs-standards/004-standards-to-platform-pipeline.md** — Downstream pipeline consuming validated ZS-* records

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
