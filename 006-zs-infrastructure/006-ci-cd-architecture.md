# 006-ci-cd-architecture.md
## CI/CD orchestration layout via GitHub Actions
### Pipeline stages, workflow definitions, deployment automation

**Document type:** Architecture
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Draft

---

## Table of contents

1. [Purpose](#1-purpose)
2. [CI/CD design principles](#2-cicd-design-principles)
3. [Workflow naming convention](#3-workflow-naming-convention)
4. [Pipeline stages](#4-pipeline-stages)
5. [Standard workflows per repository](#5-standard-workflows-per-repository)
6. [Environment strategy](#6-environment-strategy)
7. [Secrets management](#7-secrets-management)
8. [Deployment targets](#8-deployment-targets)
9. [Plane-aware branching strategy](#9-plane-aware-branching-strategy)
10. [Workflow triggers reference](#10-workflow-triggers-reference)
11. [Cross-references](#11-cross-references)

---

## 1. Purpose

This document defines the complete CI/CD architecture for the ZarishSphere ecosystem. GitHub Actions is the sole CI/CD engine. Every repository, every workflow, every deployment target is specified here.

The CI/CD pipeline is the automation layer that connects all five infrastructure layers:
- **Layer 1** (GitHub) — source code and workflow definitions
- **Layer 2** (Cloudflare) — deployment targets
- **Layer 3** (Domains) — subdomain routing for deployed sites
- **Layer 4** (Email) — deployment notification triggers
- **Layer 5** (CI/CD) — this document, the automation engine

---

## 2. CI/CD design principles

### 2.1 Zero-cost

All CI/CD runs on GitHub Actions free tier (2,000 build minutes/month for public repositories). No runner other than GitHub-hosted runners is used.

| Resource | Free tier limit | ZarishSphere usage estimate |
|---|---|---|
| Build minutes | 2,000 min/month (public) | ~500 min/month (post-launch) |
| Concurrent jobs | 20 | Typically 1-3 |
| Storage (artifacts) | 500 MB | Under 100 MB |
| Workflow runs | Unlimited | — |

### 2.2 Documentation-first

Every workflow file is documented in this document before it is created. The workflow YAML files are the implementation; this document is the specification.

### 2.3 Minimal workflow footprint

Workflows are designed to complete in under 5 minutes. Long-running processes (standards harvesting, dataset generation) are separate scheduled workflows with their own runners.

### 2.4 Validation before deployment

No deployment occurs unless all validation checks pass. The pipeline enforces:

1. ZUSS compliance (documentation repos)
2. Build verification (code repos)
3. Cross-reference integrity (documentation repos)
4. Secret availability check (deployment workflows)

---

## 3. Workflow naming convention

Per ZUSS §2.3, all workflow files follow the three-segment naming pattern:

```
[id]--[trigger]--[process].yml
```

| Segment | Rules | Example |
|---|---|---|
| `[id]` | 3-digit zero-padded integer, unique per repo | `101` |
| `[trigger]` | What fires the workflow (kebab-case) | `on-push`, `on-pr`, `on-schedule` |
| `[process]` | What the workflow does (kebab-case) | `validate-markdown`, `publish-pages` |
| Separator | Double hyphen `--` between segments | `--` |

**Valid examples:**

```
101--on-push--validate-markdown.yml
102--on-pull-request--lint-yaml.yml
201--on-push--build-artifacts.yml
301--on-release--publish-pages.yml
401--on-schedule--nightly-harvest.yml
```

---

## 4. Pipeline stages

Every deployment follows a four-stage pipeline:

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Validate │ →  │  Build   │ →  │ Deploy   │ →  │ Notify   │
│  (lint)  │    │ (compile)│    │ (release)│    │ (status) │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

### 4.1 Stage 1: Validate

| Check | Tool | Fails on |
|---|---|---|
| Front matter completeness | Custom script | Missing required fields |
| ZUSS naming compliance | `001-zuss-validate.sh` | Naming violations |
| Cross-reference integrity | `003-resolve-cross-refs.sh` | Broken links |
| YAML syntax | `yamllint` | Invalid YAML |
| Prohibited patterns | Banned word scan | `latest` tag, banned words |
| Code linting | `golangci-lint` / `eslint` | Lint violations |

### 4.2 Stage 2: Build

| Technology | Build command | Output |
|---|---|---|
| Go | `go build ./...` | Binary |
| JavaScript/TypeScript | `npm run build` | `dist/` directory |
| Python | `python3 -m build` | Wheel |
| Markdown docs | `python3 scripts/010-refresh-files.py` | Normalised markdown |

### 4.3 Stage 3: Deploy

| Target | Trigger | Deployment method |
|---|---|---|
| Cloudflare Pages | Push to `main` | Cloudflare Pages GitHub integration |
| Cloudflare Workers | Push to `main` | `wrangler deploy` via GitHub Actions |
| GitHub Pages | Push to `main` | `actions/jekyll-deploy-pages` or `peaceiris/actions-gh-pages` |
| R2 storage | Push to `main` | `wrangler r2 object put` |

### 4.4 Stage 4: Notify

| Notification | Channel | Trigger |
|---|---|---|
| Deployment success | Email to `hello@zarishsphere.com` | All events |
| Deployment failure | Email to `hello@zarishsphere.com` | Failed deploy |
| Validation failure | PR check annotation | Failed validate stage |

---

## 5. Standard workflows per repository

### 5.1 Documentation repositories

Applies to: `zs-docs`, and any repo with a `docs/` directory.

| ID | File | Trigger | Process |
|---|---|---|---|
| 101 | `101--on-push--validate-markdown.yml` | Push to any branch | ZUSS validation, cross-ref check |
| 102 | `102--on-pull-request--lint-yaml.yml` | PR opened/updated | YAML syntax, front matter |
| 201 | `201--on-push--publish-pages.yml` | Push to `main` | Build and deploy to Cloudflare Pages |

### 5.2 Code repositories

Applies to: `zs-platform`, `zs-g2a-engine`, `zs-fhir-server`, `zs-console`, etc.

| ID | File | Trigger | Process |
|---|---|---|---|
| 101 | `101--on-push--build-check.yml` | Push to any branch | Compile/type-check, lint |
| 102 | `102--on-pull-request--test.yml` | PR opened/updated | Unit tests, integration tests |
| 201 | `201--on-push--build-artifacts.yml` | Push to `main` | Build binaries, run tests |
| 301 | `301--on-release--publish.yml` | Release published | Build, tag, deploy to production |
| 401 | `401--on-schedule--nightly-security.yml` | Nightly schedule | Dependency vulnerability scan |

### 5.3 Infrastructure repository

Applies to: `zs-infra`.

| ID | File | Trigger | Process |
|---|---|---|---|
| 101 | `101--on-push--validate-terraform.yml` | Push to any branch | Terraform/Cloudflare config validation |
| 201 | `201--on-push--apply-infra.yml` | Push to `main` | Apply infrastructure changes to Cloudflare |

### 5.4 Every repository minimum

Every repository must have at minimum:

1. A validation workflow triggered on push (validate YAML, lint, build check)
2. A PR check workflow (runs tests, validates changes)
3. A deployment workflow (triggered on merge to main or release)

---

## 6. Environment strategy

### 6.1 Environment definitions

| Environment | Purpose | Branch | Review required | Deployment frequency |
|---|---|---|---|---|
| `development` | Feature development and local testing | Feature branches | No | Per PR commit |
| `staging` | Pre-production validation | `main` | Yes (PR review) | Per-merge |
| `production` | Live ecosystem | `main` (with tag) | Yes (release) | Per release |

### 6.2 Environment mapping

| Deployment target | Development | Staging | Production |
|---|---|---|---|
| Cloudflare Pages | Preview URL (per PR) | Custom domain | Custom domain |
| Cloudflare Workers | `dev` route | `staging` route | Production route |
| GitHub Pages | N/A | `staging` branch | `main` |
| R2 storage | Dev bucket | Staging bucket | Production bucket |

### 6.3 Environment protection rules

| Rule | Staging | Production |
|---|---|---|
| Required reviewers | 0 | 1 (admin) |
| Wait timer | 0 | 0 |
| Disallow bypass | N/A | Yes (admin must review) |
| Deployment branch | `main` | `main` with semver tag |

---

## 7. Secrets management

### 7.1 Secret categories

| Category | Scope | Examples |
|---|---|---|
| Cloudflare | Organisation-level | `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID` |
| R2 | Organisation-level | `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY` |
| Workers | Per-repo (when needed) | `WORKER_SECRET_*` |
| Notifications | Organisation-level | `SLACK_WEBHOOK` (future) |

### 7.2 Secret usage in workflows

```yaml
jobs:
  deploy-pages:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Cloudflare Pages
        uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          projectName: zs-docs
```

### 7.3 Secret rotation

| Secret | Rotation interval | Method |
|---|---|---|
| `CLOUDFLARE_API_TOKEN` | Every 90 days | Cloudflare Dashboard → API Tokens |
| `R2_ACCESS_KEY_ID` | Every 180 days | Cloudflare R2 → API Tokens |
| `R2_SECRET_ACCESS_KEY` | Every 180 days | Cloudflare R2 → API Tokens |

---

## 8. Deployment targets

### 8.1 Cloudflare Pages

| Project | Source repo | Build output | Subdomain |
|---|---|---|---|
| `zarishsphere-home` | `zs-home` | `/` | `zarishsphere.com` |
| `zs-console` | `zs-console` | `/` | `console.zarishsphere.com` |
| `zs-marketplace` | `zs-marketplace` | `/` | `marketplace.zarishsphere.com` |
| `zs-builder` | `zs-builder` | `/` | `builder.zarishsphere.com` |
| `zs-docs` | `zs-docs` | `/` | `docs.zarishsphere.com` |
| `zs-index` | `zs-zarish-index` | `/` | `index.zarishsphere.com` |
| `zs-standards` | `zs-zarish-standards` | `/` | `standards.zarishsphere.com` |
| `zs-status` | `zs-status` | `/` | `status.zarishsphere.com` |

> **Note:** The build output directory is normalized to `/` (the GitHub Pages default source) for all projects, matching → **006-zs-infrastructure/003-cloudflare-architecture.md** §7.1.

### 8.2 Cloudflare Workers

| Worker | Source repo | Route | Environment variables |
|---|---|---|---|
| `zs-api-gateway` | `zs-api` | `api.zarishsphere.com/*` | Per service |
| `zs-fhir-proxy` | `zs-fhir-server` | `fhir.zarishsphere.com/*` | FHIR server URL |
| `zs-identity` | `zs-services` | `identity.zarishsphere.com/*` | JWT secret |
| `zs-console-api` | `zs-console` | `console.zarishsphere.com/api/*` | Console config |
| `zs-builder-api` | `zs-builder` | `builder.zarishsphere.com/api/*` | Builder config |

### 8.3 R2 storage

| Bucket | Source repo | Content type | Upload trigger |
|---|---|---|---|
| `zs-standards-index` | `zs-zarish-index` | JSON metadata | Weekly schedule |
| `zs-standards-artifacts` | `zs-zarish-standards` | YAML standards | Weekly schedule |
| `zs-static-assets` | `zs-home` | Images, fonts | On push to main |
| `zs-backup` | `zs-infra` | SQL dumps | Nightly schedule |

### 8.4 GitHub Pages

| Repository | Custom domain | Source |
|---|---|---|
| `zs-docs` | `docs.zarishsphere.com` (via Cloudflare) | `main` branch root |
| `.github` | None (org profile) | `profile/README.md` |

---

## 9. Plane-aware branching strategy

### 9.1 Branch model

```
main (development / staging)
├── feature/* (feature branches)
├── release/plane-0 (air-gapped deployment snapshot)
├── release/plane-1 (edge deployment snapshot)
├── release/plane-2 (district deployment snapshot)
├── release/plane-3 (national cloud snapshot)
└── release/plane-4 (global SaaS snapshot)
```

### 9.2 Plane branch lifecycle

1. **Development** occurs on `main` and feature branches
2. **Release branches** (`release/plane-*`) are cut from `main` at stable points
3. **Plane branches** receive cherry-picked updates from `main` when needed
4. **Plane 0 releases** are bundled as `git bundle` files for USB transfer

### 9.3 Plane 0 release process

```
1. Branch: git checkout -b release/plane-0-v1.0.0 main
2. Remove cloud-dependent configs:
   - Remove wrangler.toml (Workers not needed)
   - Remove Cloudflare-specific environment vars
   - Set localhost URLs in config
3. Build offline-compatible artifacts:
   - Go binary (standalone server)
   - Static files (offline PWA)
   - SQLite schema and seed data
4. Bundle: git bundle create plane-0-v1.0.0.bundle --all
5. Transfer: USB drive → Plane 0 device
```

### 9.4 CI/CD behaviour per plane

| Plane | CI/CD engine | Connectivity | Deployment method |
|---|---|---|---|
| 0 | Manual (shell scripts) | None | USB git bundle |
| 1 | GitHub Actions (when connected) | Occasional | Git push on connect |
| 2 | GitHub Actions | Periodic | Standard push → deploy |
| 3 | GitHub Actions | Persistent | Standard push → deploy |
| 4 | GitHub Actions | Always-on | Full automated pipeline |

### 9.5 Staging vs production promotion

```
feature/* branch
  → PR review → merge to main
    → Staging deployment (Cloudflare Pages preview)
      → Verify on staging URL
        → Create release tag (v1.0.0)
          → Production deployment (Cloudflare Pages custom domain)
```

---

## 10. Workflow triggers reference

| Trigger | Event | Used for |
|---|---|---|
| `push` | Any branch push | Validation, build check |
| `pull_request` | PR opened, synchronised, reopened | PR checks, test suites |
| `pull_request_review` | PR review submitted | Conditional deployments |
| `pull_request_target` | PR with access to secrets | Safe secret-based workflows |
| `release` | Release published | Production deployment |
| `schedule` | Cron schedule | Nightly builds, security scans |
| `workflow_dispatch` | Manual trigger | One-off tasks, manual deploy |
| `workflow_run` | Another workflow completes | Dependent workflows |
| `create` | Branch or tag created | Branch initialisation |

---

## 11. Cross-references

→ **006-zs-infrastructure/002-github-org-architecture.md** — GitHub org, branch protection, and secrets
→ **006-zs-infrastructure/003-cloudflare-architecture.md** — Deployment targets (Pages, Workers, R2) and worker routes
→ **006-zs-infrastructure/004-domain-architecture.md** — Subdomain routing for deployed sites
→ **006-zs-infrastructure/005-email-architecture.md** — Notification email integration
→ **007-zs-tech-stack/006-oss-tool-catalog.md** — OSS tool versions used in pipeline stages
→ **003-zs-platform/003-deployment-planes.md** — Plane definitions behind the plane-aware branching strategy

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
