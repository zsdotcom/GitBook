---
id: "ZS-INDEX-009"
title: "zarishsphere-operations"
domain: "009-zs-operations"
doc-type: "index"
entity-type: "landing-page"
description: >-
  Landing page for the Operations part of the ZarishSphere documentation. Indexes the fifteen documents covering the SOPs for document creation, GitHub workflow, contribution, ZUSS compliance audit, deployment checklist, credential succession, audit procedures, the credential inventory reference, deployment, incident response, security incidents, backup and DR, country and facility onboarding, and monitoring dashboards.
version: "1.0.0"
status: "stable"
tags:
  - "index"
  - "operations"
  - "sop"
  - "runbooks"
isolation_tier: "global"
canonical: true
audience:
  - "contributors"
  - "deployers"
  - "ai-agents"
last_updated: "2026-08-01"
---
# 000-operations.md
## Operations — SOPs and runbooks

The operational runbooks that keep the project running. Fifteen documents cover everything from creating a new document to deploying services, responding to incidents, onboarding countries and facilities, and backing up production data.

## Document index

| # | File | Description | Type | Status |
|---|---|---|---|---|
| 001 | [001-sop-new-document-creation.md](001-sop-new-document-creation.md) | SOP-001: provisioning and validating a new ZUSS-compliant document (naming, header block, ToC, validation, draft status) | SOP | Stable |
| 002 | [002-sop-github-workflow.md](002-sop-github-workflow.md) | SOP-002: Git and GitHub workflow for all repos — branch naming, commits, PRs, squash-merge, branch protection, cleanup | SOP | Stable |
| 003 | [003-sop-contribution-process.md](003-sop-contribution-process.md) | SOP-003: the full external-contribution lifecycle from issue and PR through triage, ZUSS review, revision, merge, or rejection | SOP | Stable |
| 004 | [004-sop-zuss-compliance-audit.md](004-sop-zuss-compliance-audit.md) | SOP-004: running and interpreting the validation scripts and generating audit reports | SOP | Stable |
| 005 | [005-sop-deployment-checklist.md](005-sop-deployment-checklist.md) | SOP-005: pre-deployment checklist and release procedure for zs-docs to GitHub Pages, including rollback | SOP | Stable |
| 006 | [006-sop-credential-succession.md](006-sop-credential-succession.md) | SOP-006: mandatory credential documentation, rotation, and succession; implements Law 11 and ADR-012 | SOP | Stable |
| 007 | [007-sop-audit-procedures.md](007-sop-audit-procedures.md) | SOP-007: maintaining and verifying the git-based decision audit trail; implements Law 10 | SOP | Stable |
| 008 | [008-credential-inventory.md](008-credential-inventory.md) | Reference template for the live credential registry mandated by SOP-006 §5.1 (currently no entries) | Reference | Stable |
| 009 | [009-sop-deployment.md](009-sop-deployment.md) | SOP-009: GitOps deployment of platform services via Argo CD 3.4.6 — sync, manual sync, force rollout, rollback, post-deploy verification | SOP | Stable |
| 010 | [010-sop-incident-response.md](010-sop-incident-response.md) | SOP-010: incident response including P0–P3 classification, first-15-minute P0 playbook, diagnosis checklist, postmortem template | SOP | Stable |
| 011 | [011-sop-security-incident.md](011-sop-security-incident.md) | SOP-011: security incident playbook — triggers, first-30-minute containment and forensics, three scenarios, GDPR notification templates | SOP | Stable |
| 012 | [012-sop-backup-dr.md](012-sop-backup-dr.md) | SOP-012: PostgreSQL 18.4 backup (WAL plus daily R2 dumps), monthly restore tests, three DR scenarios, RTO 4h and RPO 1h | SOP | Stable |
| 013 | [013-sop-country-onboarding.md](013-sop-country-onboarding.md) | SOP-013: onboarding a new country — repos, infrastructure, Keycloak realm, data load, CAMM L0 to L1 | SOP | Stable |
| 014 | [014-sop-facility-onboarding.md](014-sop-facility-onboarding.md) | SOP-014: onboarding a health facility — FHIR Location registration, Keycloak tenant, distro update, staff training | SOP | Stable |
| 015 | [015-sop-monitoring-dashboards.md](015-sop-monitoring-dashboards.md) | SOP-015: Grafana 13.1.1 monitoring — 9-dashboard inventory, daily health checks, thresholds, alert response, GitOps provisioning | SOP | Stable |

## See also

→ **README.md** — Space landing page and how the ten domains fit together
→ **002-zs-foundation/005-country-adoption-model.md** — The adoption model these onboarding SOPs operationalise
→ **008-zs-adrs/000-architecture-decision-records.md** — Decisions these procedures implement (ADR-003, ADR-012)
