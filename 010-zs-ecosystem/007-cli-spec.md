# 007-cli-spec.md
## ZarishSphere CLI specification
### Command-line interface

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Secondary status](#2-secondary-status)
3. [CLI commands](#3-cli-commands)
4. [Use cases](#4-use-cases)
5. [CLI architecture](#5-cli-architecture)
6. [Command reference](#6-command-reference)
7. [Configuration](#7-configuration)
8. [Output formats](#8-output-formats)
9. [Shell completion](#9-shell-completion)
10. [Exit codes](#10-exit-codes)
11. [Example workflows](#11-example-workflows)
12. [Rate limiting and retry](#12-rate-limiting-and-retry)
13. [Cross-references](#13-cross-references)

---

## 1. Purpose

The ZarishSphere CLI provides terminal-based access to ecosystem management functions. It is always secondary to the Console. The CLI exists for CI/CD automation, bulk operations, and headless environments where a GUI is unavailable.

## 2. Secondary status

Per Law 6 of the constitution, every capability must be operable through the Console without a terminal. The CLI may offer additional automation features but must never be the only way to perform a function.

All procedures document the GUI-first path before the CLI path (ZUSS §8.2).

## 3. CLI commands

| Command | Purpose |
|---|---|
| `zs module deploy` | Deploy a module |
| `zs module list` | List deployed modules |
| `zs app deploy` | Deploy an app |
| `zs form push` | Push form definitions |
| `zs data export` | Export data |
| `zs sync status` | Check sync status |
| `zs config get/set` | Manage configuration |
| `zs plane status` | Check deployment plane health |
| `zs login` | Authenticate with the platform |
| `zs logout` | Clear stored credentials |
| `zs version` | Show CLI version |

## 4. Use cases

- CI/CD pipelines (GitHub Actions)
- Bulk import/export operations
- Server-side configuration in headless environments
- Integration with existing DevOps workflows
- Automated deployment scripts
- Remote diagnostics and health checks

## 5. CLI architecture

The CLI is built using the Go ecosystem's standard toolchain:

- **Cobra** — command tree and flag parsing (github.com/spf13/cobra)
- **Viper** — configuration management (github.com/spf13/viper)
- **Go SDK** — all CLI commands delegate to the Go SDK's client library as defined in → **[010-zs-ecosystem/006-sdk-spec.md](010-zs-ecosystem/006-sdk-spec.md)**

Command tree structure:

```
zs                        # Root command
├── login                 # Authenticate
├── logout                # Clear credentials
├── module                # Module operations
│   ├── deploy            # Deploy a module
│   ├── list              # List deployed modules
│   ├── status            # Module deployment status
│   └── remove            # Remove a module
├── app                   # App operations
│   ├── deploy            # Deploy an app
│   ├── list              # List installed apps
│   └── status            # App deployment status
├── form                  # Form operations
│   ├── push              # Push form definitions
│   ├── validate          # Validate form schemas
│   └── list              # List available forms
├── data                  # Data operations
│   ├── export            # Export data
│   ├── import            # Import data
│   └── query             # Run ad-hoc queries
├── sync                  # Sync operations
│   ├── status            # Check sync status
│   ├── trigger           # Trigger sync
│   └── bundle            # Create offline sync bundle
├── config                # Configuration
│   ├── get               # Get config value
│   ├── set               # Set config value
│   └── view              # View full config
├── plane                 # Plane operations
│   ├── status            # Check plane health
│   └── info              # Show plane details
└── version               # Show version
```

## 6. Command reference

| Command | Syntax | Flags | Description |
|---|---|---|---|
| `zs login` | `zs login [--token \| --oauth]` | `--token`, `--oauth`, `--region` | Authenticate with API token or OAuth2 flow |
| `zs module deploy` | `zs module deploy <path> [flags]` | `--plane`, `--config`, `--dry-run` | Deploy a module from a local path or registry URL |
| `zs module list` | `zs module list [flags]` | `--domain`, `--status`, `--format` | List deployed modules with optional domain filter |
| `zs app deploy` | `zs app deploy <app-id> [flags]` | `--plane`, `--version`, `--config` | Deploy an app from the Marketplace |
| `zs form push` | `zs form push <file> [flags]` | `--module`, `--validate`, `--force` | Push a form definition file (FHIR Questionnaire JSON/YAML) |
| `zs data export` | `zs data export [flags]` | `--format`, `--module`, `--since`, `--out` | Export data in supported formats |
| `zs data import` | `zs data import <file> [flags]` | `--format`, `--module`, `--validate` | Import data from external files |
| `zs sync status` | `zs sync status [flags]` | `--plane`, `--json` | Show synchronization status for a plane |
| `zs sync trigger` | `zs sync trigger [flags]` | `--target`, `--full` | Trigger immediate synchronization |
| `zs config get` | `zs config get <key>` | — | Get a specific configuration value |
| `zs config set` | `zs config set <key> <value>` | `--global`, `--local` | Set a configuration value |
| `zs plane status` | `zs plane status [flags]` | `--watch`, `--json` | Check deployment plane health |
| `zs version` | `zs version` | — | Display CLI version and build info |

## 7. Configuration

The CLI loads configuration from the following sources (lowest to highest priority):

1. **Defaults** — compiled-in default values
2. **Config file** — `~/.zarishsphere/config.yaml` (auto-created on first `zs login`)
3. **Environment variables** — all config keys mapped to `ZS_*` env vars
4. **Command-line flags** — highest priority

### Config file format

```yaml
# ~/.zarishsphere/config.yaml
api:
  base_url: "https://api.zarishsphere.dev"
  timeout: 30s
  retry:
    max_retries: 3
    base_delay: 1s
auth:
  token: ""  # Set via zs login
  region: "us-east"
plane:
  default: 0
output:
  format: "table"    # table, json, yaml, raw
  color: "auto"      # auto, always, never
sync:
  auto_poll: true
  poll_interval: 60s
```

### Environment variable mapping

| Config key | Environment variable |
|---|---|
| `api.base_url` | `ZS_API_BASE_URL` |
| `api.timeout` | `ZS_API_TIMEOUT` |
| `auth.region` | `ZS_REGION` |
| `output.format` | `ZS_OUTPUT_FORMAT` |
| `plane.default` | `ZS_DEFAULT_PLANE` |

## 8. Output formats

All commands that return structured data support the following output formats:

| Format | Flag | Use case |
|---|---|---|
| Table | `--format table` | Human-readable terminal output (default) |
| JSON | `--format json` | Programmatic consumption, jq piping |
| YAML | `--format yaml` | Config file generation, diff-friendly |
| Raw | `--format raw` | Minimal output for simple scripts |

Example:

```bash
zs module list --format json | jq '.[] | {name: .name, status: .state}'
zs plane status --format yaml > plane-status.yaml
```

## 9. Shell completion

The CLI provides shell completion scripts for three shells:

| Shell | Command | File |
|---|---|---|
| Bash | `zs completion bash` | `~/.bash_completion.d/zs` |
| Zsh | `zs completion zsh` | `~/.zsh/completion/_zs` |
| Fish | `zs completion fish` | `~/.config/fish/completions/zs.fish` |

Installation (one-time):

```bash
# Bash
source <(zs completion bash)

# Zsh (with oh-my-zsh)
zs completion zsh > ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zs/_zs

# Fish
zs completion fish > ~/.config/fish/completions/zs.fish
```

## 10. Exit codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | General error (invalid input, operation failure) |
| 2 | Configuration error (missing or invalid config) |
| 3 | Authentication error (missing or expired credentials) |
| 4 | API error (server returned error response) |
| 5 | Network error (connection refused, timeout) |
| 6 | Permission denied (user lacks required role) |
| 7 | Not found (requested resource does not exist) |
| 8 | Conflict (resource state conflict) |
| 9 | Rate limited (API rate limit exceeded) |
| 10 | Plane offline (target plane unreachable) |

Scripts should check exit codes for reliable automation:

```bash
if zs module deploy ./my-module --plane 1; then
    echo "Deployment started successfully"
else
    case $? in
        3) echo "Please run: zs login" ;;
        4) echo "API error — check server logs" ;;
        6) echo "Permission denied — contact admin" ;;
    esac
fi
```

## 11. Example workflows

### Workflow 1: Deploy a module via CLI

```bash
# 1. Authenticate
zs login --token "$ZS_API_TOKEN"

# 2. Verify target plane is healthy
zs plane status --plane 1

# 3. Deploy module with config override
zs module deploy ./modules/immunization \
    --plane 1 \
    --config ./configs/edge-clinic.yaml \
    --dry-run

# 4. Confirm dry-run succeeded, then deploy
zs module deploy ./modules/immunization \
    --plane 1 \
    --config ./configs/edge-clinic.yaml

# 5. Verify deployment
zs module list --domain health --format table
```

### Workflow 2: Export data via CLI

```bash
# 1. List available modules
zs module list --domain nutrition

# 2. Export last 30 days in FHIR format
zs data export \
    --module nutrition-screening \
    --format fhir \
    --since "2026-05-11" \
    --out ./exports/nutrition-data.ndjson

# 3. Verify export
ls -lh ./exports/
```

### Workflow 3: Run diagnostics

```bash
# 1. Check overall health
zs plane status --watch

# 2. Check sync status
zs sync status --json

# 3. View current config
zs config view

# 4. Test API connectivity
zs module list --format json --timeout 5s
```

## 12. Rate limiting and retry

The CLI implements client-side rate limiting to avoid overwhelming the API:

- **Default rate limit** — 60 requests per minute per client
- **Burst allowance** — up to 10 concurrent requests
- **Retry policy** — automatic retry on HTTP 429 (rate limited), 502, 503, 504
- **Retry backoff** — exponential: 1s, 2s, 4s, 8s (max 3 retries by default)
- **Jitter** — ±500ms random jitter to avoid thundering herd
- **Configurable** — `api.retry.max_retries` and `api.retry.base_delay` in config file

When the CLI receives a 429 response, it logs a warning and waits for the `Retry-After` header duration before retrying. If all retries are exhausted, it exits with code 9.

## 13. Cross-references

→ **010-zs-ecosystem/001-console-spec.md** — Console specification: the GUI-first primary interface the CLI never replaces
→ **010-zs-ecosystem/006-sdk-spec.md** — SDK specification: the Go SDK client library all CLI commands delegate to
→ **010-zs-ecosystem/008-api-spec.md** — API specification: the endpoints the CLI calls
→ **010-zs-ecosystem/011-distributions-spec.md** — Distributions specification: the `zs distribution` commands for download, verify, deploy, and rollback
→ **010-zs-ecosystem/009-services-spec.md** — Services specification: the plane status and sync health endpoints the CLI reports on
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 6: no-code first — every procedure documents the GUI path first
→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS §8.2: SOP step-by-step procedures are numbered and GUI-first

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
