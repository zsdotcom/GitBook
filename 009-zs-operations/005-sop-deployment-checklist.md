# 005-sop-deployment-checklist.md
## SOP-005: Pre-deployment verification and documentation release checklist
### Standard Operating Procedure — deployment checklist

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

To provide a verifiable pre-deployment checklist that ensures all documentation in `zs-docs` passes validation, is correctly built, and is successfully deployed to production (GitHub Pages and/or Cloudflare Pages) before a release.

---

## 2. Scope

**In scope:** Deployment of `zs-docs` to GitHub Pages (primary). Deployment of `zs-docs` to Cloudflare Pages (secondary, if configured). Validation and build steps required before any deployment. Post-deployment URL verification.

**Out of scope:** Code deployments to `zs-platform` or other repositories. Infrastructure changes. Domain/DNS configuration changes (handled by → **[004-domain-architecture.md](006-zs-infrastructure/004-domain-architecture.md)**).

---

## 3. Roles

| Role | Who |
|---|---|
| **Deployer** | Documentation Agent (AI) or maintainer executing the deployment |
| **Reviewer** | Reviewer Agent (AI) — validates pre-deployment checks pass |
| **Maintainer** | Mohammad Ariful Islam — authorised to approve production deployments |

---

## 4. Preconditions

- All changes are committed and merged to `main` (see → **[002-sop-github-workflow.md](009-zs-operations/002-sop-github-workflow.md)**).
- The local working tree is clean: `git status` shows no uncommitted changes.
- The deployer has `admin` access to the `zarishsphere/zs-docs` GitHub repository.
- GitHub Actions is enabled on the repository.
- (Optional) Cloudflare Pages project `zs-docs` is configured if Cloudflare deployment is used.

---

## 5. Steps

### 5.1 Pre-deployment validation

Run all validation scripts in order. Any `✗` failure stops the deployment.

```bash
# Step 1: Refresh and normalise
python3 scripts/010-refresh-files.py

# Step 2: ZUSS compliance
bash scripts/001-zuss-validate.sh

# Step 3: Pipeline status (informational)
bash scripts/002-pipeline-status.sh

# Step 4: Cross-reference integrity
bash scripts/003-resolve-cross-refs.sh
```

**Checklist item:** ☐ All 4 scripts pass with exit code 0.

If any script fails:
1. Note the specific error message.
2. Fix the issue (refer to → **[004-sop-zuss-compliance-audit.md](009-zs-operations/004-sop-zuss-compliance-audit.md)** for common fixes).
3. Re-run all 4 scripts from the start.
4. Do not proceed until all pass.

### 5.2 Verify Git state

```bash
git status
git log --oneline -3
```

**Checklist item:** ☐ Working tree is clean (no modified or untracked files).

**Checklist item:** ☐ HEAD is on `main` and is the most recent commit.

If there are uncommitted changes:
```bash
git add -A
git commit -m "chore: pre-deployment cleanup"
git push origin main
```

### 5.3 GitHub Pages deployment

GitHub Pages is the primary hosting platform for `zs-docs`. The deployment is triggered automatically by pushing to `main` if the GitHub Actions workflow is configured.

**To verify the workflow exists:**

1. In the browser, open `https://github.com/zsdotcom/zs-docs/actions`.
2. Verify a workflow named `Deploy to GitHub Pages` (or similar) is present.
3. If missing, create `.github/workflows/deploy-pages.yml` in the repository:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**Checklist item:** ☐ GitHub Actions workflow exists and is configured for `main` branch pushes.

**Trigger the deployment:**

```bash
git push origin main
```

This push triggers the GitHub Actions workflow.

**Monitor the deployment:**

1. In the browser, open `https://github.com/zsdotcom/zs-docs/actions`.
2. Click the running workflow run.
3. Wait for all jobs to complete (green checkmark).

**Checklist item:** ☐ GitHub Actions deployment workflow completes successfully.

### 5.4 Optional: Cloudflare Pages deployment

If Cloudflare Pages is used as a secondary deployment target:

1. Open the browser and navigate to **Cloudflare Dashboard → Pages → zs-docs**.
2. Click **Set up deployment** if not already configured, or **Trigger deploy** for manual deployment.
3. Configuration:
   - **Project name:** `zs-docs`
   - **Production branch:** `main`
   - **Build command:** (leave blank — zs-docs is static Markdown, no build step)
   - **Build output directory:** `/` (root)
4. Enable automatic deployments: **GitHub → zarishsphere/zs-docs → main branch**.

**Checklist item:** ☐ Cloudflare Pages deployment completes (if configured).

### 5.5 Verify live URLs

After deployment completes, verify the site is accessible:

```bash
# GitHub Pages URL
curl -s -o /dev/null -w "%{http_code}" https://zarishsphere.github.io/zs-docs/

# Custom domain (if configured)
curl -s -o /dev/null -w "%{http_code}" https://docs.zarishsphere.com/
```

Expected response: `200`

**Checklist item:** ☐ GitHub Pages URL returns HTTP 200.

**Checklist item:** ☐ Custom domain URL returns HTTP 200 (if configured).

Verify key files are accessible:

```bash
curl -s -o /dev/null -w "%{http_code}" https://zarishsphere.github.io/zs-docs/index.md
curl -s -o /dev/null -w "%{http_code}" https://zarishsphere.github.io/zs-docs/llms.txt
```

Expected: `200` for both.

**Checklist item:** ☐ `index.md` is accessible at the live URL.

**Checklist item:** ☐ `llms.txt` is accessible at the live URL.

### 5.6 Verify llms.txt in production

The `llms.txt` file is the AI-consumable index of the entire documentation set. Verify its content is correct:

```bash
curl -s https://zarishsphere.github.io/zs-docs/llms.txt | head -20
```

Check that:
- The header contains `# zs-docs` and the correct description.
- All 10 folders are referenced.
- No skeleton files are listed (or if they are, they are clearly marked).
- The file is valid Markdown.

**Checklist item:** ☐ `llms.txt` is valid and contains expected content.

### 5.7 Post-deployment monitoring

For 24 hours after deployment, monitor:

1. **GitHub Actions status** — Check that no scheduled workflows fail.
2. **Cloudflare analytics (if configured)** — Check for 404 errors in the **Analytics → Pages** section.
3. **Visitor access** — Open the site in a private/incognito browser window to verify no caching issues.
4. **Mobile view** — Open the site on a mobile browser to verify responsive layout.

**Checklist item:** ☐ No 404 errors in analytics within 24 hours.

**Checklist item:** ☐ Site renders correctly in private browser session.

### 5.8 Rollback procedure

If the deployment introduces issues:

**Rollback GitHub Pages:**
1. In the browser, open `https://github.com/zsdotcom/zs-docs/actions`.
2. Click the **previous successful workflow run** (before the problematic deployment).
3. Click **Re-run all jobs**.
4. This redeploys the previous version.

Alternatively, revert the Git commit:
```bash
git revert HEAD --no-edit
git push origin main
```

**Rollback Cloudflare Pages:**
1. Open **Cloudflare Dashboard → Pages → zs-docs**.
2. Click **Deployments**.
3. Find the last known-good deployment.
4. Click the **...** menu → **Rollback to this deployment**.

**Checklist item:** ☐ Rollback procedure is documented and accessible.

---

## 6. Expected outcome

- All validation scripts pass before deployment is attempted.
- GitHub Pages workflow completes successfully and the site is live at `https://zarishsphere.github.io/zs-docs/`.
- All key files (`index.md`, `llms.txt`, root documents) return HTTP 200.
- Cloudflare Pages deployment (if configured) is synchronised.
- Post-deployment monitoring shows no errors.
- Rollback procedure is ready if needed.

---

## 7. Escalation

| Issue | Action |
|---|---|
| GitHub Actions workflow fails | Open the Actions log in GitHub. Check for configuration errors in the workflow YAML. Verify `deploy-pages.yml` is in `.github/workflows/`. |
| GitHub Pages URL returns 404 | Verify the Pages setting: **Settings → Pages → Source** is set to "GitHub Actions". Check that the artifact uploaded correctly. |
| Cloudflare deployment fails | Check the build log in Cloudflare Dashboard. Ensure the GitHub connection is active. Re-trigger from the Cloudflare dashboard. |
| `llms.txt` is missing from live site | Run `python3 scripts/010-refresh-files.py` locally and push. The refresh script regenerates `llms.txt`. |
| Site shows stale content | Clear Cloudflare cache: **Cloudflare Dashboard → Caching → Purge everything**. Wait 5 minutes and recheck. For GitHub Pages, wait up to 10 minutes for DNS propagation. |
| Emergency rollback required | Contact the maintainer immediately: **security@zarishsphere.com**. Execute the rollback procedure in step 5.8. |
| Custom domain SSL certificate issue | Verify DNS settings in Cloudflare. Ensure the CNAME or A record points correctly. SSL is auto-provisioned by Cloudflare — may take up to 15 minutes. |

---

## Cross-references

- → **[002-sop-github-workflow.md](009-zs-operations/002-sop-github-workflow.md)** — SOP-002: Merging changes to `main` before deployment
- → **[004-sop-zuss-compliance-audit.md](009-zs-operations/004-sop-zuss-compliance-audit.md)** — SOP-004: Validation scripts and common fixes
- → **[004-domain-architecture.md](006-zs-infrastructure/004-domain-architecture.md)** — Domain and DNS configuration for `zarishsphere.com`

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
