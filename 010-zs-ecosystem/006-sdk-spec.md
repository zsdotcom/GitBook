# 006-sdk-spec.md
## ZarishSphere SDK specification
### Software development kits

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Supported languages](#2-supported-languages)
3. [SDK contents](#3-sdk-contents)
4. [Optionality constraint](#4-optionality-constraint)
5. [SDK architecture](#5-sdk-architecture)
6. [Package structure](#6-package-structure)
7. [Installation guide](#7-installation-guide)
8. [Error handling patterns](#8-error-handling-patterns)
9. [Example walkthrough](#9-example-walkthrough)
10. [Testing with the SDK](#10-testing-with-the-sdk)
11. [Versioning policy](#11-versioning-policy)
12. [Cross-references](#12-cross-references)

---

## 1. Purpose

The ZarishSphere SDK provides software development kits for developers who want to build custom integrations, modules, or applications on the platform. The SDK is always optional — no ecosystem function requires SDK usage.

## 2. Supported languages

| Language | Use case |
|---|---|
| Go | Backend services, module development, CLI tools |
| JavaScript | Frontend integration, browser extensions |
| Python | Data analysis, ML integration, automation scripts |

## 3. SDK contents

| Component | Description |
|---|---|
| API client libraries | Wrappers for all ecosystem APIs |
| Module development templates | Scaffold new modules |
| Authentication helpers | OAuth, JWT, API token handling |
| Form helpers | Generate and parse FHIR Questionnaire resources |
| Data export helpers | Export in FHIR, CSV, JSON, Parquet |
| CLI tools (Go SDK) | Dev utility commands |
| Integration examples | Tutorials and reference implementations |

## 4. Optionality constraint

No ecosystem function requires SDK usage. Everything achievable through the SDK is also achievable through the Console. The SDK exists for developer convenience, not as a gate to functionality.

## 5. SDK architecture

The SDK follows a layered architecture pattern consistent across all three language implementations:

- **Transport layer** — HTTP client with configurable timeout, retry, and TLS settings. Wraps all REST and GraphQL endpoints defined in → **[010-zs-ecosystem/008-api-spec.md](010-zs-ecosystem/008-api-spec.md)**.
- **Authentication layer** — Pluggable auth providers: OAuth2 client credentials, JWT bearer tokens, API key headers. Token refresh and caching are handled automatically.
- **Client layer** — Language-specific client objects (e.g., `zs.NewClient()`, `new ZarishSphereClient()`) that expose typed methods for every API operation.
- **Domain models** — Shared data structures (FHIR Resources, ModuleConfig, FormDefinition) serialized to/from JSON. Go uses typed structs, JavaScript uses TypeScript interfaces, Python uses dataclasses.
- **High-level helpers** — Convenience functions for common tasks: form rendering, data export, sync triggers.

Authentication flow diagram (conceptual):

```
User App → SDK Client → Auth Provider → Token Cache → API Request with Bearer Token → ZarishSphere API
```

Token refresh occurs transparently when the cached token expires. In Plane 0 environments, the SDK falls back to local API token authentication without OAuth.

## 6. Package structure

Each language SDK follows the package conventions of its ecosystem:

### Go module layout

```
github.com/zarishsphere/zs-sdk-go/
├── zs.go                 # Main client constructor
├── client/               # HTTP client with retry middleware
├── auth/                 # OAuth2, JWT, API token providers
├── forms/                # FHIR Questionnaire helpers
├── modules/              # Module deployment client
├── export/               # Data export client
├── cmd/                  # CLI development tools
│   └── zs-dev/           # Dev utility binary
├── go.mod
└── README.md
```

### JavaScript package layout

```
@zarishsphere/sdk/
├── src/
│   ├── index.ts          # Main entry point
│   ├── client.ts         # API client with axios/ky
│   ├── auth.ts           # OAuth2 and token management
│   ├── forms.ts          # Form generation helpers
│   ├── modules.ts        # Module API wrapper
│   └── export.ts         # Export API wrapper
├── package.json
├── tsconfig.json
└── README.md
```

### Python package layout

```
zarishsphere-sdk/
├── zarishsphere/
│   ├── __init__.py       # Package init with Client class
│   ├── client.py         # httpx-based API client
│   ├── auth.py           # OAuth2 session management
│   ├── forms.py          # Form data helpers
│   ├── modules.py        # Module operations
│   └── export.py         # Export operations
├── pyproject.toml
├── setup.py
└── README.md
```

## 7. Installation guide

### Go

```bash
go get github.com/zarishsphere/zs-sdk-go@v1.0.0
```

Import path: `import "github.com/zarishsphere/zs-sdk-go"`

Minimum Go version: 1.22

### JavaScript

```bash
npm install @zarishsphere/sdk@1.0.0
# or
yarn add @zarishsphere/sdk@1.0.0
# or
pnpm add @zarishsphere/sdk@1.0.0
```

Requires Node.js 20+. ES module only (`"type": "module"`).

### Python

```bash
pip install zarishsphere-sdk==1.0.0
# or
poetry add zarishsphere-sdk@1.0.0
```

Requires Python 3.11+. Published to PyPI under Apache 2.0 license.

## 8. Error handling patterns

Each language SDK uses idiomatic error handling:

### Go

The Go SDK returns errors as the last return value. All API errors are wrapped with context:

```go
client, err := zs.NewClient(zs.WithRegion("us-east"))
if err != nil {
    return fmt.Errorf("failed to create client: %w", err)
}

module, err := client.Modules.Get(ctx, "module-id")
if err != nil {
    var apiErr *zs.APIError
    if errors.As(err, &apiErr) {
        // apiErr.Code, apiErr.Status, apiErr.Details
    }
    return err
}
```

### JavaScript

The JavaScript SDK uses typed exceptions:

```javascript
import { ZarishSphereClient, AuthenticationError, ValidationError } from '@zarishsphere/sdk';

const client = new ZarishSphereClient({ region: 'us-east' });

try {
  const module = await client.modules.get('module-id');
} catch (err) {
  if (err instanceof AuthenticationError) {
    // Token expired, refresh flow
  } else if (err instanceof ValidationError) {
    // Invalid request parameters
  }
}
```

### Python

The Python SDK raises typed exceptions:

```python
from zarishsphere import Client
from zarishsphere.exceptions import AuthenticationError, APIError

client = Client(region="us-east")

try:
    module = client.modules.get("module-id")
except AuthenticationError:
    # Handle token expiry
except APIError as e:
    # e.status_code, e.details
    pass
```

All SDK clients implement automatic retry with exponential backoff for transient failures (HTTP 429, 502, 503, 504). Retry policy is configurable: `WithRetry(maxRetries=3, baseDelay=1.0)` in Go, `{ retry: { maxRetries: 3 } }` in JS, `retry=Retry(total=3)` in Python.

## 9. Example walkthrough

The following walkthrough demonstrates creating a custom nutrition screening module using the Go SDK:

### Step 1: Initialize the client

```go
client, err := zs.NewClient(
    zs.WithBaseURL("https://api.zarishsphere.dev"),
    zs.WithAPIKey(os.Getenv("ZS_API_KEY")),
)
```

### Step 2: Define the module metadata

```go
mod := &zs.ModuleDefinition{
    Name:        "nutrition-screening",
    DisplayName: "Nutrition Screening",
    Domain:      "nutrition",
    Version:     "1.0.0",
    Forms:       []string{"muac-assessment", "dietary-diversity"},
}
```

### Step 3: Register the module

```go
result, err := client.Modules.Create(ctx, mod)
```

### Step 4: Create a form definition

```go
form := &zs.FormDefinition{
    ModuleID: result.ID,
    Name:     "muac-assessment",
    Schema:   loadMUACQuestionnaire(), // FHIR Questionnaire JSON
}
_, err = client.Forms.Create(ctx, form)
```

### Step 5: Deploy the module

```go
deployment, err := client.Modules.Deploy(ctx, result.ID, zs.DeployToPlane(1))
```

### Step 6: Verify deployment status

```go
status, err := client.Modules.Status(ctx, deployment.ID)
fmt.Printf("Module %s deployed to plane %d: %s\n", status.Name, status.Plane, status.State)
```

### Step 7: Export data from the module

```go
export, err := client.Export.Create(ctx, &zs.ExportRequest{
    ModuleID:  result.ID,
    Format:    zs.FormatFHIR,
    DateFrom:  time.Now().Add(-30 * 24 * time.Hour),
})
```

### Step 8: Submit the module for Marketplace listing

```go
_, err = client.Marketplace.Publish(ctx, result.ID)
```

This walkthrough works identically in JavaScript and Python with equivalent method names.

## 10. Testing with the SDK

Each language SDK ships with mock services and testing utilities:

### Go testing patterns

```go
import "github.com/zarishsphere/zs-sdk-go/mock"

func TestModuleDeploy(t *testing.T) {
    server := mock.NewServer(t)
    defer server.Close()

    client := zs.NewClient(zs.WithBaseURL(server.URL()))
    
    mod, err := client.Modules.Create(ctx, &zs.ModuleDefinition{
        Name: "test-module",
    })
    assert.NoError(t, err)
    assert.Equal(t, "test-module", mod.Name)
}
```

### JavaScript testing patterns

```javascript
import { MockServer } from '@zarishsphere/sdk/testing';

describe('ModuleClient', () => {
  let server;
  let client;

  beforeEach(() => {
    server = new MockServer();
    client = new ZarishSphereClient({ baseURL: server.url() });
  });

  it('creates a module', async () => {
    server.mockModuleCreate({ name: 'test-module' });
    const mod = await client.modules.create({ name: 'test-module' });
    expect(mod.name).toBe('test-module');
  });
});
```

### Python testing patterns

```python
from zarishsphere.testing import MockServer
from zarishsphere import Client

def test_module_create():
    server = MockServer()
    client = Client(base_url=server.url())
    
    mod = client.modules.create({"name": "test-module"})
    assert mod["name"] == "test-module"
```

Mock services simulate all API endpoints without requiring a live ZarishSphere deployment. They are suitable for unit tests, integration test scaffolding, and CI pipelines.

## 11. Versioning policy

- **Semantic Versioning 2.0** — all SDK packages follow `MAJOR.MINOR.PATCH` versioning as defined in → **[002-zs-foundation/003-licensing-policy.md](002-zs-foundation/003-licensing-policy.md)**.
- **Major version** — breaking changes to public API surface. Client code requires migration.
- **Minor version** — additive features and deprecation warnings. Backward compatible.
- **Patch version** — bug fixes, security patches, documentation improvements.
- **Compatibility guarantee** — within a major version, all three language SDKs maintain equivalent functionality. Minor versions are released simultaneously across all three languages within 72 hours.
- **Deprecation policy** — deprecated methods emit warnings for two minor versions before removal. Removal only occurs at a major version boundary.
- **Pre-release tags** — `alpha`, `beta`, `rc` suffixes per semver spec. Pre-release versions are used for early access and testing.
- **No unversioned references** — all versions are pinned explicitly. The Marketplace and Console reference SDK versions by exact semver string. The `latest` tag is never used in any version reference.

## 12. Cross-references

→ **010-zs-ecosystem/008-api-spec.md** — API specification: the REST and GraphQL endpoints wrapped by the SDK transport layer
→ **010-zs-ecosystem/007-cli-spec.md** — CLI specification: all CLI commands delegate to the Go SDK client library
→ **010-zs-ecosystem/005-forms-spec.md** — Forms specification: the FHIR Questionnaire helpers in the SDK
→ **010-zs-ecosystem/010-modules-spec.md** — Modules specification: the module deployment operations the SDK automates
→ **002-zs-foundation/003-licensing-policy.md** — Licensing policy: semantic versioning and Apache 2.0 publishing rules
→ **003-zs-platform/006-api-design.md** — API design: the contracts, versioning, and authentication conventions the SDK follows

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
