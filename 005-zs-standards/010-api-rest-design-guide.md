# 010-api-rest-design-guide.md
## REST API design guide
### URL patterns, versioning, headers, error responses — V1

**Document type:** Standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative for all REST API design in the ZarishSphere Platform

---

## Table of contents

1. [URL patterns](#1-url-patterns)
2. [Request headers](#2-request-headers)
3. [Response headers](#3-response-headers)
4. [Pagination](#4-pagination)
5. [API versioning](#5-api-versioning)
6. [HTTP status code conventions](#6-http-status-code-conventions)
7. [Rate limiting](#7-rate-limiting)

---

## 1. URL patterns

### 1.1 FHIR resource endpoints

FHIR resource endpoints follow the standard FHIR RESTful API pattern:

```
GET    /fhir/R5/{ResourceType}                      # Search
POST   /fhir/R5/{ResourceType}                      # Create
GET    /fhir/R5/{ResourceType}/{id}                  # Read
PUT    /fhir/R5/{ResourceType}/{id}                  # Update
DELETE /fhir/R5/{ResourceType}/{id}                  # Soft delete
GET    /fhir/R5/{ResourceType}/{id}/_history         # History
```

### 1.2 FHIR operations

```
GET    /fhir/R5/metadata                             # CapabilityStatement
POST   /fhir/R5/{ResourceType}/$validate             # Validate resource
GET    /fhir/R5/ValueSet/$expand?url={url}           # Expand ValueSet
GET    /fhir/R5/{ResourceType}/$doc                  # Document reference
POST   /fhir/R5/$export                              # Bulk data export
```

### 1.3 Platform service endpoints

Non-FHIR platform services use the following URL structure:

```
GET    /health                                       # Health check (no auth)
GET    /metrics                                      # Prometheus metrics (internal)
GET    /api/v1/{service}/{resource}                  # Platform service API
POST   /api/v1/{service}/{resource}                  # Platform service create
GET    /api/v1/{service}/{resource}/{id}             # Platform service read
```

### 1.4 URL conventions

| Rule | Example |
|---|---|
| Lowercase throughout | `/fhir/r5/patient` (but FHIR uses uppercase resource types) |
| Hyphens for multi-word resources | `/api/v1/patient-registry/` |
| No trailing slash | `/fhir/R5/Patient`, not `/fhir/R5/Patient/` |
| Plural resource names | `/fhir/R5/Patients` is non-standard — use FHIR resource type names |
| Version in URL path | `/fhir/R5/` for FHIR, `/api/v1/` for platform APIs |

---

## 2. Request headers

### 2.1 FHIR API request headers

| Header | Required | Description |
|---|---|---|
| `Authorization: Bearer {jwt}` | Yes (protected resources) | SMART App Launch 2.2.0 JWT bearer token |
| `Content-Type: application/fhir+json` | Yes (write operations) | FHIR JSON content type |
| `Accept: application/fhir+json` | Yes | Request FHIR JSON response format |
| `X-Tenant-ID` | Conditional | Overrides JWT tenant claim (requires elevated scope) |
| `X-Request-ID` | No | UUID for request tracing across services |

### 2.2 Platform API request headers

| Header | Required | Description |
|---|---|---|
| `Authorization: Bearer {jwt}` | Yes | Platform JWT bearer token |
| `Content-Type: application/json` | Yes | JSON content type |
| `Accept: application/json` | Yes | JSON response format |
| `X-Request-ID` | No | UUID for request tracing |

---

## 3. Response headers

### 3.1 Standard response headers

| Header | When present | Description |
|---|---|---|
| `Content-Type` | Always | Media type of the response body |
| `Location` | `201 Created` | URL of the newly created resource |
| `ETag: W/"version-id"` | Read/update responses | Version ID for optimistic locking |
| `Last-Modified` | Read/search responses | RFC 7231 formatted timestamp |
| `X-Request-ID` | When provided in request | Echo the request tracing ID |
| `X-Tenant-ID` | Always | The effective tenant for the request |
| `RateLimit-Remaining` | Always | Remaining requests in current window |
| `RateLimit-Reset` | Always | Unix timestamp of rate limit reset |

---

## 4. Pagination

### 4.1 FHIR-style pagination

All paginated list endpoints follow the FHIR Bundle pattern:

```
GET /fhir/R5/Patient?_count=20&_offset=0

Response:
{
  "resourceType": "Bundle",
  "type": "searchset",
  "total": 47,
  "link": [
    { "relation": "self", "url": "/fhir/R5/Patient?_count=20&_offset=0" },
    { "relation": "next", "url": "/fhir/R5/Patient?_count=20&_offset=20" },
    { "relation": "prev", "url": "/fhir/R5/Patient?_count=20&_offset=0" }
  ],
  "entry": [...]
}
```

### 4.2 Platform API pagination

Non-FHIR platform APIs use offset-based pagination:

```
GET /api/v1/registry/patients?limit=20&offset=0

Response:
{
  "data": [...],
  "pagination": {
    "total": 47,
    "limit": 20,
    "offset": 0,
    "next_offset": 20,
    "prev_offset": null
  }
}
```

### 4.3 Pagination limits

| Parameter | FHIR API | Platform API |
|---|---|---|
| Default page size | 20 | 20 |
| Maximum page size | 200 | 100 |
| Pagination field | `_count`, `_offset` | `limit`, `offset` |
| Max total without narrowing | 10,000 | 10,000 |

> **Constraint:** No list endpoint may return all resources without pagination. Requests without explicit `_count`/`limit` parameters default to the configured page size.

---

## 5. API versioning

### 5.1 FHIR versioning

FHIR API version is embedded in the URL path:

| Version | URL prefix | Status |
|---|---|---|
| R5 | `/fhir/R5/` | Active — primary |
| R4 | `/fhir/R4/` | Bridge — partner compatibility |

Service version (implementation) is tracked via SemVer in the Docker image tag and Helm chart version. It does not appear in the URL.

### 5.2 Platform API versioning

| Version | URL prefix | Status |
|---|---|---|
| v1 | `/api/v1/` | Active — current |

### 5.3 Version compatibility

| Rule | Description |
|---|---|
| Backward-compatible changes | Additive (new fields, new optional parameters) — no version bump needed |
| Breaking changes | Field removal, type changes, required field addition — new major version |
| Deprecation notice | Breaking changes announced at least 1 minor release cycle before removal |
| Sunset policy | Deprecated endpoints remain available for 6 months after replacement is available |

---

## 6. HTTP status code conventions

### 6.1 Success codes

| Code | Usage |
|---|---|
| `200 OK` | Successful read, search, update |
| `201 Created` | Successful create (with `Location` header) |
| `202 Accepted` | Asynchronous operation accepted (export, batch) |
| `204 No Content` | Successful delete |

### 6.2 Error codes

| Code | Usage | Response body |
|---|---|---|
| `400 Bad Request` | Invalid input, missing required field | `OperationOutcome` |
| `401 Unauthorized` | Missing or invalid authentication | `OperationOutcome` |
| `403 Forbidden` | Insufficient scope, cross-tenant access | `OperationOutcome` |
| `404 Not Found` | Resource does not exist | `OperationOutcome` |
| `409 Conflict` | Version ID mismatch (optimistic locking) | `OperationOutcome` |
| `422 Unprocessable Entity` | FHIR validation failure | `OperationOutcome` |
| `429 Too Many Requests` | Rate limit exceeded | JSON with retry-after |
| `500 Internal Server Error` | Unexpected server error | `OperationOutcome` |

---

## 7. Rate limiting

### 7.1 Rate limit defaults

| Tier | Requests per minute | Applied at |
|---|---|---|
| Global SaaS (Plane 4) | 10,000 | API gateway |
| National cloud (Plane 3) | 5,000 | API gateway |
| District server (Plane 2) | 1,000 | Local gateway |
| Edge (Plane 1) | 500 | Local gateway |

### 7.2 Rate limit headers

Every API response includes these headers:

```
RateLimit-Remaining: 4998
RateLimit-Reset: 1711267200
RateLimit-Limit: 5000
```

### 7.3 Exceeding the limit

When the rate limit is exceeded, the API returns HTTP 429 with:

```json
{
  "resourceType": "OperationOutcome",
  "issue": [{
    "severity": "error",
    "code": "throttled",
    "diagnostics": "Rate limit exceeded. Retry after 30 seconds.",
    "details": {
      "text": "Retry-After: 30"
    }
  }]
}
```

---

## Cross-references

→ **005-zs-standards/001-zarish-standards-overview.md** — Standards framework and three-type classification
→ **005-zs-standards/005-fhir-r5-conventions.md** — FHIR searchset bundle and pagination rules (parallel definition)
→ **005-zs-standards/007-fhir-search-standards.md** — FHIR search and pagination rules (parallel definition)
→ **003-zs-platform/006-api-design.md** — Platform-level API design principles
→ **010-zs-ecosystem/008-api-spec.md** — Ecosystem API component specification

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
