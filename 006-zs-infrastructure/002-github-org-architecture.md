# 002-github-org-architecture.md
## GitHub organisation architecture and configuration
### Repositories, teams, permissions, branch protection, and automation

**Document type:** Architecture
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Draft

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Organisation structure](#2-organisation-structure)
3. [Repository inventory](#3-repository-inventory)
4. [Team model and permissions](#4-team-model-and-permissions)
5. [Branch protection rules](#5-branch-protection-rules)
6. [Issue and PR templates](#6-issue-and-pr-templates)
7. [GitHub Actions CI/CD](#7-github-actions-cicd)
8. [GitHub Pages](#8-github-pages)
9. [GitHub Secrets management](#9-github-secrets-management)
10. [Repository creation sequence](#10-repository-creation-sequence)
11. [Cross-references](#11-cross-references)

---

## 1. Purpose

This document defines the complete GitHub organisation architecture for the ZarishSphere Foundation (`github.com/zarishsphere`). It covers every repository, every team, every permission rule, branch protection policy, issue template, PR template, and automation configuration.

The GitHub organisation is the control plane for the entire ecosystem. Per ADR-003 (GitHub as Government), all infrastructure, governance, and configuration changes flow through this organisation. No system outside of GitHub has authority to change infrastructure state.

---

## 2. Organisation structure

### 2.1 Organisation profile

| Property | Value |
|---|---|
| Organisation name | `zarishsphere` |
| URL | `https://github.com/zsdotcom` |
| Plan | GitHub Free |
| Owner | Mohammad Ariful Islam (founder) |
| Organisation type | Open source project |
| Default repository permission | `Read` |
| Two-factor authentication | Required for all members |
| Base branch | `main` |

### 2.2 Organisation-level settings

| Setting | Value |
|---|---|
| Default branch name | `main` |
| Allow merge commits | Yes |
| Allow squash merging | Yes (preferred) |
| Allow rebase merging | No |
| Allow auto-merge | Yes |
| Delete head branches on merge | Yes |
| Issues enabled | Default on |
| Projects | Disabled |
| Wikis | Disabled |
| Sponsorships | Not enabled |
| Discussions | Disabled |

---

## 3. Repository inventory

### 3.1 Governance and documentation

| Repository | Purpose | Visibility | Default branch |
|---|---|---|---|
| `.github` | Organisation profile, community health files, issue/PR templates | Public | `main` |
| `zs-docs` | Master documentation repository — single source of truth | Public | `main` |

### 3.2 Core platform

| Repository | Purpose | Visibility | Default branch |
|---|---|---|---|
| `zs-platform` | ZarishSphere Platform core — platform-of-platforms infrastructure | Public | `main` |
| `zs-g2a-engine` | Guideline-to-Action engine — standard transformation automation | Public | `main` |
| `zs-fhir-server` | Go-native FHIR R5 server | Public | `main` |
| `zs-infra` | Infrastructure as Code — Cloudflare, GitHub Actions, config | Public | `main` |

### 3.3 Standards and indexes

| Repository | Purpose | Visibility | Default branch |
|---|---|---|---|
| `zs-zarish-index` | ZARISH-INDEX project — data, scripts, docs | Public | `main` |
| `zs-zarish-standards` | ZARISH-STANDARDS project — transformation, schemas, docs | Public | `main` |

### 3.4 Ecosystem components

| Repository | Purpose | Visibility | Default branch |
|---|---|---|---|
| `zs-console` | ZarishSphere Console — browser-based management centre | Public | `main` |
| `zs-marketplace` | ZarishSphere Marketplace — component discovery | Public | `main` |
| `zs-builder` | ZarishSphere Builder — no-code creation tool | Public | `main` |
| `zs-forms` | ZarishSphere Forms — dynamic form engine | Public | `main` |
| `zs-sdk` | ZarishSphere SDK — Go/JS/Python development kit | Public | `main` |
| `zs-cli` | ZarishSphere CLI — command-line interface | Public | `main` |
| `zs-api` | ZarishSphere API gateway and endpoint definitions | Public | `main` |
| `zs-services` | ZarishSphere backend services (identity, audit, sync) | Public | `main` |
| `zs-home` | ZarishSphere home page — public landing site | Public | `main` |
| `zs-status` | ZarishSphere status page — ecosystem status dashboard | Public | `main` |

> **Note:** `zs-home` and `zs-status` are the source repositories for the `zarishsphere-home` and `zs-status` Cloudflare Pages projects (see → **006-zs-infrastructure/003-cloudflare-architecture.md** §7.1).

### 3.5 Module repositories

| Repository | Purpose | Visibility | Default branch |
|---|---|---|---|
| `zs-modules-health` | Health domain module | Public | `main` |
| `zs-modules-education` | Education domain module | Public | `main` |
| `zs-modules-logistics` | Logistics domain module | Public | `main` |
| `zs-modules-protection` | Protection domain module | Public | `main` |
| `zs-modules-nutrition` | Nutrition domain module | Public | `main` |
| `zs-modules-wash` | WASH domain module | Public | `main` |
| `zs-modules-environment` | Environment domain module | Public | `main` |
| `zs-modules-human-rights` | Human rights domain module | Public | `main` |
| `zs-modules-governance` | Governance and public administration module | Public | `main` |
| `zs-modules-trade` | Trade and commerce module | Public | `main` |

### 3.6 Content repositories

| Repository | Purpose | Visibility | Default branch |
|---|---|---|---|
| `zs-content-forms` | Domain-agnostic form definitions (FHIR Questionnaire) | Public | `main` |
| `zs-content-protocols` | Clinical and operational protocol definitions | Public | `main` |
| `zs-content-templates` | Deployment templates and distribution packages | Public | `main` |
| `zs-content-reports` | Report templates and dashboard definitions | Public | `main` |

---

## 4. Team model and permissions

### 4.1 Team structure

| Team | Slug | Purpose | Parent |
|---|---|---|---|
| `admin` | `admin` | Full admin access to all repositories | None |
| `dev` | `dev` | Write access to development repositories | None |
| `docs` | `docs` | Write access to documentation repositories | None |

### 4.2 Team permissions

| Repository | Admin team | Dev team | Docs team |
|---|---|---|---|
| `.github` | Admin | Maintain | Write |
| `zs-docs` | Admin | Triage | Write |
| `zs-platform` | Admin | Write | Triage |
| `zs-zarish-index` | Admin | Write | Triage |
| `zs-zarish-standards` | Admin | Write | Triage |
| `zs-g2a-engine` | Admin | Write | Triage |
| `zs-fhir-server` | Admin | Write | Triage |
| `zs-infra` | Admin | Write | Triage |
| `zs-console` | Admin | Write | Triage |
| `zs-marketplace` | Admin | Write | Triage |
| `zs-builder` | Admin | Write | Triage |
| `zs-forms` | Admin | Write | Triage |
| `zs-sdk` | Admin | Write | Triage |
| `zs-cli` | Admin | Write | Triage |
| `zs-api` | Admin | Write | Triage |
| `zs-services` | Admin | Write | Triage |
| `zs-home` | Admin | Write | Triage |
| `zs-status` | Admin | Write | Triage |
| `zs-modules-*` | Admin | Write | Triage |
| `zs-content-*` | Admin | Write | Triage |

### 4.3 Team membership

| Team | Current members |
|---|---|
| `admin` | Mohammad Ariful Islam (owner) |
| `dev` | Open to contributors on acceptance |
| `docs` | Open to contributors on acceptance |

> **Constraint:** Only the founder (Mohammad Ariful Islam) has admin access until the Foundation governance model defines a succession process. This is a security measure for the pre-launch phase.

---

## 5. Branch protection rules

### 5.1 Rules for `main` branch (all repositories)

| Rule | Setting |
|---|---|
| Require pull request reviews | Yes |
| Required approving reviews | 1 |
| Dismiss stale reviews | Yes |
| Require review from Code Owners | No (post-launch: Yes for zs-docs, zs-infra) |
| Require status checks | Yes |
| Require branches up to date | Yes |
| Include administrators | Yes |
| Restrict push access | Yes (admin only) |
| Allow force pushes | No |
| Allow deletions | No |

### 5.2 Required status checks

The following checks must pass before merging to `main` in any repository that has CI/CD configured:

1. `validate-markdown` — ZUSS compliance check
2. `build-check` — Compilation or build verification
3. `lint-yaml` — YAML syntax validation for workflow files
4. `cross-ref-validate` — Cross-reference integrity check

### 5.3 Release branch rules

Release branches (`release/*`) have the same protection rules as `main` plus:

| Rule | Setting |
|---|---|
| Require signed commits | Yes |
| Block force pushes | Yes |
| Restrict push access | Admin only |

---

## 6. Issue and PR templates

### 6.1 Issue templates

All issue templates live in `.github/ISSUE_TEMPLATE/` at the organisation level (shared across all repos). Custom templates per repository may supplement but not replace organisation-level templates.

**Bug report** — `.github/ISSUE_TEMPLATE/01-bug-report.yml`

| Field | Required |
|---|---|
| Summary | Yes |
| Steps to reproduce | Yes |
| Expected behaviour | Yes |
| Actual behaviour | Yes |
| Environment (Plane 0/1/2/3/4) | Yes |
| Screenshots or logs | No |
| Related document | No |

**Feature request** — `.github/ISSUE_TEMPLATE/02-feature-request.yml`

| Field | Required |
|---|---|
| Problem statement | Yes |
| Proposed solution | Yes |
| Alternatives considered | Yes |
| ADR required | Yes (if architectural impact) |
| Plane compatibility | Yes |

**Documentation update** — `.github/ISSUE_TEMPLATE/03-docs-update.yml`

| Field | Required |
|---|---|
| Document affected | Yes |
| Change type | Yes (fix / addition / removal) |
| Description | Yes |
| ZUSS compliance | Yes |

### 6.2 Pull request template

Template location: `.github/PULL_REQUEST_TEMPLATE.md`

```markdown
## Summary
<!-- One sentence description -->

## Related issue
<!-- Closes #NNN or N/A -->

## Document type
<!-- Architecture / ADR / SOP / Spec / Other -->

## Changes
<!-- Bullet list of changes -->

## Plane 0 compatibility
<!-- Does this change affect air-gapped deployments? -->
- [ ] Yes — described below
- [ ] No

## Validation
- [ ] Scripts run: `010-refresh-files.py`
- [ ] ZUSS validated: `001-zuss-validate.sh`
- [ ] Cross-refs resolved: `003-resolve-cross-refs.sh`

## Checklist
- [ ] Front matter is complete
- [ ] Headers use sentence case
- [ ] No banned words
- [ ] License footer present
- [ ] All cross-references resolve to existing files
```

---

## 7. GitHub Actions CI/CD

GitHub Actions is the CI/CD engine for the ZarishSphere ecosystem. Every repository has at minimum one workflow. Workflow files follow ZUSS naming convention:

```
[id]--[trigger]--[process].yml
```

### 7.1 Workflow naming convention

| Segment | Rules |
|---|---|
| `[id]` | 3-digit zero-padded integer, unique per repo |
| `[trigger]` | What fires the workflow: `on-push`, `on-pr`, `on-schedule`, `on-release` |
| `[process]` | What the workflow does: `validate-markdown`, `build-artifacts`, `publish-pages` |
| Separator | Double hyphen `--` between segments |

Example: `101--on-push--validate-markdown.yml`

### 7.2 Standard workflows per repo

| Repository type | Standard workflow files |
|---|---|
| Documentation (`zs-docs`, any repo with `docs/`) | `101--on-push--validate-markdown.yml`, `102--on-pull-request--lint-yaml.yml`, `201--on-push--publish-pages.yml` |
| Code (`zs-platform`, `zs-g2a-engine`, `zs-fhir-server`, `zs-console`, etc.) | `101--on-push--build-check.yml`, `102--on-pull-request--test.yml`, `201--on-push--build-artifacts.yml`, `301--on-release--publish.yml`, `401--on-schedule--nightly-security.yml` |
| Infrastructure (`zs-infra`) | `101--on-push--validate-terraform.yml`, `201--on-push--apply-infra.yml` |

Full workflow definitions, triggers, and stage details live in → **006-zs-infrastructure/006-ci-cd-architecture.md** §5.

---

## 8. GitHub Pages

### 8.1 ZS-Docs site

| Property | Value |
|---|---|
| Repository | `zs-docs` |
| Source branch | `main` |
| Source directory | `/` |
| Custom domain | `docs.zarishsphere.com` |
| Enforce HTTPS | Yes |
| Build and deployment | GitHub Actions (not the default Pages builder) |

The `zs-docs` Pages site renders the markdown documentation repository as a browsable static site using GitHub's built-in Pages rendering. The site is also published to Cloudflare Pages for CDN delivery.

### 8.2 Organisation landing page

| Property | Value |
|---|---|
| Repository | `.github` |
| Source | `profile/README.md` |
| Custom domain | None (uses organisation URL) |
| Purpose | Organisation profile page shown at `github.com/zarishsphere` |

---

## 9. GitHub Secrets management

### 9.1 Organisation-level secrets

These secrets are available to all repositories:

| Secret name | Purpose | Source |
|---|---|---|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API access for Pages/Workers deployment | Cloudflare Dashboard |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare account identifier | Cloudflare Dashboard |
| `R2_ACCESS_KEY_ID` | R2 storage access key | Cloudflare R2 |
| `R2_SECRET_ACCESS_KEY` | R2 storage secret key | Cloudflare R2 |

### 9.2 Repository-level secrets

Secrets set per repository when needed:

| Secret name | Scope | Purpose |
|---|---|---|
| `PAGES_DEPLOY_TOKEN` | Per-repo with Pages | Cloudflare Pages deployment |
| `WORKER_DEPLOY_TOKEN` | Per-repo with Workers | Cloudflare Workers deployment |
| `SLACK_WEBHOOK` | All repos | Deployment notifications (future) |

### 9.3 Secret rotation policy

| Secret type | Rotation frequency |
|---|---|
| Cloudflare API tokens | Every 90 days |
| R2 access keys | Every 180 days |
| Deploy tokens | Every 90 days |

---

## 10. Repository creation sequence

Repositories are created in a specific order. No repository is created before its corresponding documentation exists in `zs-docs`.

```
 1. zs-docs              ← Create first
 2. .github              ← Organisation health files
 3. zs-zarish-index      ← After 004-zs-index/ is complete
 4. zs-zarish-standards  ← After 005-zs-standards/ is complete
 5. zs-platform          ← After 003-zs-platform/ is complete
 6. zs-g2a-engine        ← After 003-zs-platform/004-g2a-engine.md is complete
 7. zs-infra             ← After 006-zs-infrastructure/ is complete
 8. zs-fhir-server       ← After 007-zs-tech-stack/002-go-fhir-server.md is complete
 9-16. Ecosystem repos   ← After corresponding 010-zs-ecosystem/ specs are complete
17+. Module repos        ← After platform is operational
```

---

## 11. Cross-references

→ **006-zs-infrastructure/001-infrastructure-overview.md** — Top-level infrastructure map and GitHub-as-government overview
→ **006-zs-infrastructure/006-ci-cd-architecture.md** — Full workflow definitions, pipeline stages, and deployment targets
→ **006-zs-infrastructure/003-cloudflare-architecture.md** — Cloudflare Pages/Workers deployment targets and secrets use
→ **008-zs-adrs/003-adr-github-as-government.md** — ADR-003: the GitHub-as-government model
→ **003-zs-platform/009-repository-catalog.md** — Repository catalog and ownership across the ecosystem
→ **007-zs-tech-stack/006-oss-tool-catalog.md** — OSS tools referenced by CI/CD workflows
→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS naming and structure rules for workflow files

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
