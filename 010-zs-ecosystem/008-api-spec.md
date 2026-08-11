# 008-api-spec.md
## ZarishSphere API specification
### REST, GraphQL, webhooks

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [API types](#2-api-types)
3. [Authentication](#3-authentication)
4. [Rate limiting](#4-rate-limiting)
5. [OpenAPI specifications](#5-openapi-specifications)
6. [Cross-references](#6-cross-references)

---

## 1. Purpose

The ZarishSphere API is the programmatic interface for all ecosystem components. It provides RESTful and GraphQL endpoints, webhook support, and OpenAPI 3.1 specifications. All APIs are free to use with no paid tiers.

## 2. API types

| Type | Endpoint | Purpose |
|---|---|---|
| REST | `api.zarishsphere.com/v1/` | Core CRUD operations |
| FHIR REST | `fhir.zarishsphere.com/R5/` | Health data operations |
| GraphQL | `api.zarishsphere.com/graphql` | Dashboards and complex queries |
| Webhooks | Configured per integration | Event-driven notifications |
| ZARISH-INDEX | `data.zarishsphere.com/v1/` | Standard index queries |

## 3. Authentication

| Method | Use case |
|---|---|
| GitHub OAuth | Console and developer access |
| API tokens | Automated and CI/CD access |
| SMART on FHIR OAuth | Third-party health app integration |
| Keycloak JWT | Service-to-service communication |

## 4. Rate limiting

All tiers are free:
- Anonymous: 10 req/min
- Authenticated: 100 req/min
- API token: 1,000 req/min
- Partner: Custom (by arrangement)

## 5. OpenAPI specifications

Every API endpoint is documented with an OpenAPI 3.1 specification at:
- `https://api.zarishsphere.com/openapi/v1.yaml`
- `https://fhir.zarishsphere.com/openapi/r5.yaml`

Specifications are version-controlled and published through CI/CD.

## 6. Cross-references

→ **005-zs-standards/010-api-rest-design-guide.md** — REST API design guide: URL patterns, versioning, headers, and error responses
→ **005-zs-standards/011-api-openapi-conventions.md** — OpenAPI specification conventions: spec structure and FHIR endpoint patterns
→ **003-zs-platform/006-api-design.md** — API design: contracts, versioning, and authentication conventions
→ **010-zs-ecosystem/009-services-spec.md** — Services specification: the backend microservices exposing these APIs
→ **010-zs-ecosystem/006-sdk-spec.md** — SDK specification: the client libraries wrapping these endpoints
→ **010-zs-ecosystem/007-cli-spec.md** — CLI specification: terminal access to the same APIs
→ **010-zs-ecosystem/019-fhir-hub-spec.md** — FHIR Integration Hub specification: the stateless R5 proxy in front of the FHIR REST endpoints
→ **005-zs-standards/005-fhir-r5-conventions.md** — FHIR R5 implementation conventions behind the FHIR REST endpoints

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
