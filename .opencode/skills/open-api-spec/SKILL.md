---
title: OpenAPI
description: Add an OpenAPI spec to a page and render interactive API blocks.
updatedAt: 2026-07-01
icon: brackets-curly
---

# OpenAPI

OpenAPI example spec from `https://petstore3.swagger.io/api/v3/openapi.json`, rendered as a `POST /pet` block.

## Purpose
Add an OpenAPI spec to a page and render interactive API blocks.

## Inputs
- An OpenAPI spec in `JSON` or `YAML`
- A spec provided as a file or a URL
- Supported versions:
  - Swagger 2.0
  - OpenAPI 3.0
  - OpenAPI 3.1

## Usage
1. Add the OpenAPI spec to a page.
2. Choose a file or URL source.
3. Render the spec as interactive API blocks.
4. Use the built-in **Test it** experience to try endpoints on the page.

## Outputs
- Interactive API blocks
- Endpoint details, parameters, schemas, and authentication schemes
- Testable requests powered by Scalar
- Support for OpenAPI 3.1-only features like `webhooks`

## Compatibility
- Swagger 2.0
- OpenAPI 3.0
- OpenAPI 3.1

## Notes
- URL-based specs require CORS GET access from the docs site.
- Allow the exact docs origin.
- Public endpoints can use:
  - `Access-Control-Allow-Origin: *`

## Example
- Example spec: `https://petstore3.swagger.io/api/v3/openapi.json`
- Example endpoint: `POST /pet`