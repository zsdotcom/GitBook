# 009-sop-deployment.md
## SOP-009: Deploying a new service version via GitOps (Argo CD)
### Standard Operating Procedure — service deployment and rollback

**Document type:** SOP
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Scope](#2-scope)
3. [Roles](#3-roles)
4. [Preconditions](#4-preconditions)
5. [Steps](#5-steps)
6. [Expected outcome](#6-expected-outcome)
7. [Escalation](#7-escalation)

---

## 1. Purpose

To define the exact procedure for deploying a new service version to the ZarishSphere Platform. ZarishSphere uses GitOps via Argo CD 3.4.6 — in normal operation, deployments are fully automated (merging to `main` triggers Argo CD auto-sync). This SOP covers the automated flow, manual intervention when auto-sync fails, pod-level force rollout, rollback of failed deployments, and post-deployment verification.

---

## 2. Scope

**In scope:** Standard automated deployments triggered by GitHub merge events. Manual Argo CD sync operations when auto-sync fails. Force rollout of stuck pods. Rollback to a previous version after a failed deployment. Post-deployment smoke tests and Grafana verification.

**Out of scope:** Deployments to non-Kubernetes targets. Infrastructure provisioning (see → **[006-ci-cd-architecture.md](006-zs-infrastructure/006-ci-cd-architecture.md)**). Documentation-only deployments (see → **[005-sop-deployment-checklist.md](009-zs-operations/005-sop-deployment-checklist.md)**). Database migrations that are not part of the application deployment.

---

## 3. Roles

| Role | Who |
|---|---|
| **Developer** | Contributor opening a PR with the change to deploy |
| **Reviewer** | Maintainer who approves the PR |
| **Deployer** | Argo CD (automated) or operator executing manual sync |
| **Verifier** | Operator running post-deployment smoke tests |
| **Incident Commander** | On-call engineer declared for failed deployments |

---

## 4. Preconditions

- The target service repository exists under `github.com/zarishsphere/`.
- Argo CD 3.4.6 is installed and configured on the target Kubernetes cluster (see → **[017-adr-argocd-gitops.md](008-zs-adrs/017-adr-argocd-gitops.md)**).
- The deployer has access to:
  - GitHub repository (read/write).
  - Argo CD UI at `https://argocd.zarishsphere.com`.
  - Grafana dashboard at `https://grafana.zarishsphere.com`.
- All CI checks pass on the PR before merge.
- The previous working version tag is known (for rollback scenarios).

---

## 5. Steps

### 5.1 Standard automated deployment (no action required)

Under normal conditions, no manual intervention is needed. The GitOps pipeline runs automatically:

```
Developer opens PR → CI passes → Reviewer approves → Merge to main
    ↓
GitHub Actions builds and pushes Docker image to GHCR
    ↓
Argo CD detects new image tag in values.yaml
    ↓
Argo CD syncs Kubernetes — rolling update (zero-downtime)
    ↓
Smoke tests execute (zs-agent-smoke-tester)
    ↓
Deployment complete ✅
```

**Checklist item:** ☐ PR merged to `main` with passing CI checks.

**Checklist item:** ☐ Argo CD auto-sync completes within 5 minutes of merge.

### 5.2 Monitor the automated pipeline

1. Open **Argo CD UI** at `https://argocd.zarishsphere.com` in a browser.
2. Find the application (e.g., `zs-svc-patient`).
3. Verify the status shows **"Healthy"** and **"Synced"** (green checkmark).
4. If status shows **"OutOfSync"** for more than 5 minutes, proceed to §5.3.

**Checklist item:** ☐ Argo CD shows "Healthy" and "Synced" status.

### 5.3 Manual sync (if Argo CD fails to auto-sync)

**Use when:** Argo CD shows "OutOfSync" but has not auto-synced within 5 minutes.

1. Open **Argo CD UI** at `https://argocd.zarishsphere.com` in a browser.
2. Click on the application name (e.g., `zs-svc-patient`).
3. Click the **"Sync"** button in the top toolbar.
4. In the sync dialog, select **"Prune"** and **"Apply only"** (default settings).
5. Click **"Synchronize"**.
6. Watch the sync progress in the UI — should complete in less than 2 minutes.

**Checklist item:** ☐ Manual sync completes successfully and status returns to "Synced".

### 5.4 Force rollout (if pods are stuck)

**Use when:** The new image is deployed but pods are stuck in `CrashLoopBackOff`, `Pending`, or `ImagePullBackOff`.

1. Open a terminal and run:
   ```bash
   kubectl rollout restart deployment/{service-name} -n zs-{namespace}
   ```
2. Watch the rollout status:
   ```bash
   kubectl rollout status deployment/{service-name} -n zs-{namespace}
   ```
3. If pods continue crashing after restart: **STOP immediately** and proceed to §5.5 (Rollback).

**Checklist item:** ☐ All pods reach `Running` state with `READY` showing `1/1` or `2/2`.

### 5.5 Rollback a failed deployment

**Use when:** Error rate > 1% after deployment, P0 incident declared, smoke tests failed, clinical data corruption detected, or service is unreachable (503 responses).

**Do not wait — patient safety depends on the system working.**

#### 5.5.1 Identify the previous version

1. In a browser, open `https://github.com/zsdotcom/{service-repo}/tags`.
2. Find the last tag before the current one.
   - Example: If current is `v1.2.0`, previous is `v1.1.3`.

#### 5.5.2 Revert in Git

**Option A — Revert the merge commit (preferred):**

1. In a terminal, clone the repository if not already local:
   ```bash
   git clone https://github.com/zsdotcom/{service-repo}.git
   cd {service-repo}
   ```
2. Revert the merge commit:
   ```bash
   git revert {merge-commit-hash} --no-edit
   git push origin main
   ```
3. Argo CD auto-deploys the reverted version.

**Option B — Direct tag update (urgent):**

1. Edit `deploy/helm/values.yaml` in the repository.
2. Change `image.tag` from the current version (e.g., `v1.2.0`) to the previous version (e.g., `v1.1.3`).
3. Commit and push to `main`.

#### 5.5.3 Watch Argo CD rollback

1. Open **Argo CD UI** at `https://argocd.zarishsphere.com`.
2. Find the service application.
3. Watch the sync — should complete in less than 2 minutes.
4. Verify the pods show the old version:
   ```bash
   kubectl get pods -n zs-clinical -o wide | grep {service-name}
   ```

#### 5.5.4 Verify service health after rollback

1. Run a health check:
   ```bash
   curl -s https://api.zarishsphere.com/{service}/health
   ```
   Expected response: `{"status": "ok"}`

2. Check **Grafana** at `https://grafana.zarishsphere.com` — the error rate should return to less than 0.1%.

3. Run a real FHIR request:
   ```bash
   curl -s -H "Authorization: Bearer {test-token}" \
     https://api.zarishsphere.com/fhir/R5/metadata | jq .resourceType
   ```
   Expected: `"CapabilityStatement"`

#### 5.5.5 Notify and document

1. Post in the platform owner group:
   ```
   Rollback of {service} v{X} complete. Now running v{Y}.
   ```
2. Open a GitHub Issue to track the root cause investigation:
   ```
   Title: [Rollback] {Service} v{X} rolled back
   Body: Include deployment time, rollback time, symptoms observed.
   ```
3. Do not re-deploy the failed version until root cause is identified.
4. Write a postmortem within 7 days using the postmortem template.

**Checklist item:** ☐ Rollback completed and service health verified.

**Checklist item:** ☐ GitHub Issue opened for root cause analysis.

**Checklist item:** ☐ Postmortem scheduled within 7 days.

### 5.6 Post-deployment verification (after any deployment)

Run these checks after every successful deployment or rollback:

| Check | Method | Expected |
|---|---|---|
| Argo CD status | UI at `https://argocd.zarishsphere.com` | "Healthy" and "Synced" |
| Grafana error rate | Dashboard at `https://grafana.zarishsphere.com` | < 0.1% |
| Smoke tests | GitHub Actions run result | All green |
| FHIR API metadata | `curl` to `/fhir/R5/metadata` | `CapabilityStatement` |

---

## 6. Expected outcome

- A new service version is deployed to production with zero downtime under normal conditions.
- Argo CD shows "Healthy" and "Synced" status after deployment.
- All smoke tests pass, and the FHIR API responds correctly.
- If deployment fails, the rollback procedure restores the previous version within 5 minutes.
- A GitHub Issue is opened for any rollback to document the root cause investigation.
- Postmortem is completed within 7 days of a failed deployment.

---

## 7. Escalation

| Issue | Action |
|---|---|
| Argo CD UI unreachable | Check Argo CD pod status: `kubectl get pods -n argocd`. Restart if needed: `kubectl rollout restart deployment/argocd-server -n argocd`. |
| Manual sync fails with error | Check the sync log in Argo CD UI for specific error messages. Common issues: missing image tag in GHCR, invalid Helm values, resource quota exceeded. |
| Pods remain in CrashLoopBackOff after force rollout | Stop all attempts. Roll back immediately (§5.5). Do not modify the deployment spec until root cause is identified. |
| Rollback fails | Use `git revert` directly on the merge commit. If git revert conflicts, force-update the image tag in `values.yaml` and push. |
| Grafana not showing updated metrics after deployment | Grafana metrics may take 1–5 minutes to refresh. Check Prometheus targets: `kubectl get pods -n zs-monitoring | grep prometheus`. |
| Emergency — clinical data corruption detected | Immediately scale down all write services: `kubectl scale deployment -n zs-core --replicas=0 --all`. Follow → **[012-sop-backup-dr.md](009-zs-operations/012-sop-backup-dr.md)** for database recovery. |
| Unsure which version to roll back to | Check Git tags in the service repository. The last stable tag is typically the one immediately preceding the current version. If uncertain, contact `@devopsariful`. |

---

## Cross-references

- → **[002-sop-github-workflow.md](009-zs-operations/002-sop-github-workflow.md)** — SOP-002: GitHub branching and PR workflow for deployments
- → **[005-sop-deployment-checklist.md](009-zs-operations/005-sop-deployment-checklist.md)** — SOP-005: Pre-deployment checklist for documentation releases
- → **[010-sop-incident-response.md](009-zs-operations/010-sop-incident-response.md)** — SOP-010: Incident response if deployment triggers P0
- → **[012-sop-backup-dr.md](009-zs-operations/012-sop-backup-dr.md)** — SOP-012: Database backup and recovery for data corruption scenarios
- → **[016-adr-opentofu-infrastructure-as-code.md](008-zs-adrs/016-adr-opentofu-infrastructure-as-code.md)** — ADR-016: OpenTofu for IaC provisioning
- → **[017-adr-argocd-gitops.md](008-zs-adrs/017-adr-argocd-gitops.md)** — ADR-017: Argo CD as GitOps deployment engine
- → **[006-ci-cd-architecture.md](006-zs-infrastructure/006-ci-cd-architecture.md)** — CI/CD architecture
- → **[003-cloudflare-architecture.md](006-zs-infrastructure/003-cloudflare-architecture.md)** — Cloudflare routing for API endpoints

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
