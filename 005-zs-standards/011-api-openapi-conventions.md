# 011-api-openapi-conventions.md
## OpenAPI specification conventions
### OpenAPI 3.2 spec structure, FHIR endpoint patterns, error responses — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all OpenAPI specifications in the ZarishSphere Platform

---

## Table of contents

1. [File location and naming](#1-file-location-and-naming)
2. [Required info block](#2-required-info-block)
3. [FHIR endpoint pattern](#3-fhir-endpoint-pattern)
4. [Standard error responses](#4-standard-error-responses)
5. [Schema organization](#5-schema-organization)
6. [Code generation requirements](#6-code-generation-requirements)

---

## 1. File location and naming

### 1.1 Required location

Every `zs-svc-*` and `zs-core-*` repository MUST include an OpenAPI specification file at:

```
{repo}/docs/openapi.yaml
```

### 1.2 File format

| Format | Requirement | Notes |
|---|---|---|
| OpenAPI version | 3.2.0 | MUST use 3.2.0 (JSON Schema 2020-12 support) |
| File format | YAML | Primary specification format |
| Validation | Auto-validated in CI | Every PR validates against openapi-spec-validator |

---

## 2. Required info block

### 2.1 Info section structure

```yaml
openapi: "3.2.0"
info:
  title: "ZarishSphere {Service Name} API"
  description: |
    {Service description — one paragraph minimum}

    ## Authentication
    All endpoints require a valid SMART App Launch 2.2.0 JWT Bearer token.
    See https://docs.zarishsphere.com/auth for details.

  version: "1.0.0"
  contact:
    name: "ZarishSphere Platform"
    email: "platform@zarishsphere.com"
    url: "https://zarishsphere.com"
  license:
    name: "Apache 2.0"
    url: "https://www.apache.org/licenses/LICENSE-2.0"

servers:
  - url: "https://api.zarishsphere.com"
    description: "Production"
  - url: "http://localhost:{port}"
    description: "Local development"
    variables:
      port:
        default: "8001"

security:
  - bearerAuth: []

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

### 2.2 Required fields

| Field | Requirement |
|---|---|
| `openapi` | Must be `"3.2.0"` |
| `info.title` | Must follow pattern `ZarishSphere {Service Name} API` |
| `info.description` | Must include authentication note referencing docs URL |
| `info.version` | Must match the service semantic version |
| `info.contact.email` | Must be `platform@zarishsphere.com` |
| `info.license` | Apache 2.0 — URL must point to apache.org |
| `servers` | Must include production and local development URLs |
| `security` | Must include bearerAuth |

---

## 3. FHIR endpoint pattern

### 3.1 FHIR resource endpoint

```yaml
paths:
  /fhir/R5/Patient:
    post:
      summary: Create a FHIR Patient resource
      operationId: createPatient
      tags: [Patient]
      requestBody:
        required: true
        content:
          application/fhir+json:
            schema:
              $ref: "#/components/schemas/Patient"
      responses:
        "201":
          description: Patient created successfully
          headers:
            Location:
              schema:
                type: string
                format: uri
              description: URL of the created resource
          content:
            application/fhir+json:
              schema:
                $ref: "#/components/schemas/Patient"
        "400":
          $ref: "#/components/responses/BadRequest"
        "401":
          $ref: "#/components/responses/Unauthorized"
        "403":
          $ref: "#/components/responses/Forbidden"
        "422":
          $ref: "#/components/responses/UnprocessableEntity"
```

### 3.2 FHIR-only endpoint rules

| Rule | Description |
|---|---|
| Content type | All FHIR endpoints use `application/fhir+json` |
| Operation ID | Follows `{action}{ResourceType}` pattern |
| Tags | Use the resource type as the tag name |
| Response references | Standard error responses MUST be `$ref` references |
| Location header | MUST be specified on all `201 Created` responses |

---

## 4. Standard error responses

### 4.1 Error response definitions

```yaml
components:
  responses:
    BadRequest:
      description: Bad request — invalid input or missing required field
      content:
        application/fhir+json:
          schema:
            $ref: "#/components/schemas/OperationOutcome"
    Unauthorized:
      description: Unauthorized — missing or invalid JWT
      content:
        application/fhir+json:
          schema:
            $ref: "#/components/schemas/OperationOutcome"
    Forbidden:
      description: Forbidden — insufficient SMART scope or cross-tenant access
      content:
        application/fhir+json:
          schema:
            $ref: "#/components/schemas/OperationOutcome"
    NotFound:
      description: Resource not found
      content:
        application/fhir+json:
          schema:
            $ref: "#/components/schemas/OperationOutcome"
    UnprocessableEntity:
      description: FHIR validation failure — resource did not pass profile validation
      content:
        application/fhir+json:
          schema:
            $ref: "#/components/schemas/OperationOutcome"
```

### 4.2 Error response rules

| Rule | Description |
|---|---|
| All error responses | MUST reference `#/components/responses/{ErrorName}` |
| FHIR errors | MUST return `application/fhir+json` with `OperationOutcome` |
| Non-FHIR errors | MUST return `application/json` with standard error object |
| Error documentation | Each error response MUST have a non-empty description |

---

## 5. Schema organization

### 5.1 Component schema structure

```yaml
components:
  schemas:
    # Resource schemas — one per FHIR resource type
    Patient:
      type: object
      description: "ZarishSphere Patient resource"
      # ...

    # Common data types
    Address:
      type: object
      # ...

    # Request/response wrappers
    PatientSearchResponse:
      type: object
      properties:
        resourceType:
          type: string
          enum: [Bundle]
        type:
          type: string
          enum: [searchset]
        total:
          type: integer
        entry:
          type: array
          items:
            $ref: "#/components/schemas/Patient"
```

### 5.2 Schema naming conventions

| Pattern | Example |
|---|---|
| FHIR resource types | `Patient`, `Observation`, `Encounter` |
| Wrapper types | `{ResourceType}SearchResponse`, `{ResourceType}BatchRequest` |
| Common types | `Address`, `HumanName`, `ContactPoint`, `Coding`, `CodeableConcept` |
| Enum types | `{ResourceType}Status`, `{ValueSet}Code` |

> **Constraint:** All schema names MUST use PascalCase. Inline anonymous schemas are not permitted — every schema must have a named definition in `components.schemas`.

---

## 6. Code generation requirements

### 6.1 Client SDK generation

OpenAPI specifications MUST be structured to support client code generation:

| Requirement | Reason |
|---|---|
| All schemas in `components.schemas` | Code generators require named schemas |
| `operationId` on every endpoint | Generates method names |
| Consistent error schemas | Generates uniform error handling |
| No `oneOf`/`anyOf` in request bodies | Some generators do not support complex polymorphism |
| `nullable: true` instead of optional `null` type | JSON Schema 2020-12 compatibility |

### 6.2 CI validation

```yaml
# .github/workflows/validate-openapi.yml
- name: Validate OpenAPI spec
  run: |
    npx @stoplight/spectral lint docs/openapi.yaml
    npx openapi-spec-validator docs/openapi.yaml
```

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/010-api-rest-design-guide.md** — REST API design conventions (headers, versioning, errors)
→ **005-zs-standards/012-api-asyncapi-conventions.md** — AsyncAPI conventions for event-driven APIs
→ **003-zs-platform/006-api-design.md** — Platform-level API design principles
→ **010-zs-ecosystem/008-api-spec.md** — Ecosystem API component specification

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
