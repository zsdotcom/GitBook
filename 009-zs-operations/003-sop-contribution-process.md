# 003-sop-contribution-process.md
## SOP-003: Review pipeline and external patch merge processing
### Standard Operating Procedure — contribution lifecycle

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

To define the complete lifecycle of an external contribution to any ZarishSphere project — from initial issue or pull request through triage, review, validation, and final merge or rejection — ensuring quality and ZUSS compliance in all accepted changes.

---

## 2. Scope

**In scope:** Contributions to all ZarishSphere repositories: `zs-docs`, `zs-platform`, `zs-zarish-index`, `zs-zarish-standards`, `zs-fhir-hub`, `zs-home`. Contributions submitted via fork-and-PR model. Documentation-only contributions and code contributions.

**Out of scope:** Internal contributions by the maintainer (handled by → **[002-sop-github-workflow.md](009-zs-operations/002-sop-github-workflow.md)**). Security vulnerability reports (handled via private channel — email the maintainer directly). Contributions to third-party dependencies.

---

## 3. Roles

| Role | Who |
|---|---|
| **Contributor** | External person submitting an issue or pull request |
| **Triage Agent** | AI agent or maintainer who performs initial assessment |
| **Reviewer** | Reviewer Agent (AI) — automated ZUSS compliance check |
| **Maintainer** | Mohammad Ariful Islam — final decision authority for all merges |

---

## 4. Preconditions

- The contributor has read the → **[004-contributor-guidelines.md](002-zs-foundation/004-contributor-guidelines.md)**.
- The contributor has signed the CLA (if one exists for the repository).
- The repository has a `CONTRIBUTING.md` or `CONTRIBUTING.md` pointing to this SOP.
- The maintainer has access to `gh` CLI and repository admin permissions.

---

## 5. Steps

### 5.1 Receive the contribution

Contributions arrive through one of two channels:

**Channel A — Issue (suggestion before code):**
1. Contributor opens an issue on the target repository.
2. Issue template includes: description, rationale, affected files, and whether the contributor intends to submit a PR.
3. The issue is automatically labelled via GitHub Actions (if configured) or manually labelled by the maintainer.

**Channel B — Pull Request (fork + PR):**
1. Contributor forks the repository on GitHub.
2. Contributor creates a feature branch in their fork and commits changes.
3. Contributor opens a PR from their fork branch to `main` in the upstream repo.

### 5.2 Initial triage

1. Open the issue or PR in the GitHub web UI.
2. Assess whether the contribution fits the ZarishSphere scope:
   - Does it align with the → **[001-zarishsphere-constitution.md](001-zs-meta/001-zarishsphere-constitution.md)** (12 laws)?
   - Does it belong to one of the 40 indexed domains (→ **[002-domain-taxonomy-40.md](004-zs-index/002-domain-taxonomy-40.md)**)?
   - Is the contribution type appropriate for the target repository?
3. Apply labels:
   - `triage/accepted` — contribution is in scope and ready for review
   - `triage/needs-info` — more information required from the contributor
   - `triage/out-of-scope` — contribution does not fit ZarishSphere (see step 5.7)
4. If `triage/needs-info`: post a comment requesting clarification and set a 14-day response window. Close if no response.

### 5.3 ZUSS compliance review (for documentation contributions)

For contributions to `zs-docs` (documentation):

1. Checkout the PR branch locally:
   ```bash
   gh pr checkout <PR-number>
   ```
2. Run the ZUSS compliance scripts:
   ```bash
   python3 scripts/010-refresh-files.py
   bash scripts/001-zuss-validate.sh
   bash scripts/002-pipeline-status.sh
   bash scripts/003-resolve-cross-refs.sh
   ```
3. Record results as a PR comment:
   ```
   ## Automated compliance check results

   - 010-refresh-files.py: ✅ Pass
   - 001-zuss-validate.sh: ⚠  See details below
   - 002-pipeline-status.sh: ✅ No skeletons remaining
   - 003-resolve-cross-refs.sh: ✅ All refs valid

   ### Issues found
   - File `XXX.md` — Check #3 (YAML front matter): missing `audience` field
   ```
4. For code contributions to `zs-platform` or other code repos:
   - Run the repository's build and test suite (Go tests, linters, etc.)
   - Verify no breaking changes to existing APIs
   - Run the FHIR validation suite if applicable

### 5.4 Content review

1. Review the substantive quality of the change:
   - Does the documentation accurately reflect the ZarishSphere ecosystem?
   - Does the code follow the repository's coding standards (Go style, etc.)?
   - Are there any factual errors or outdated references?
2. Check for cross-reference integrity:
   - Do all → references point to files that exist?
   - Are version references pinned (no `latest` tags)?
3. Post review comments on specific lines using the GitHub UI:
   - **Files changed tab** → hover over a line → click the **`+`** icon.
   - Write specific, actionable feedback.
   - Use the "Request changes" review status if corrections are needed.

### 5.5 Contributor revision cycle

1. The contributor addresses review comments by pushing additional commits to their fork branch.
2. The PR is automatically updated with the new commits.
3. Re-run validation scripts (step 5.3) after each revision.
4. Repeat until all review comments are resolved.

### 5.6 Merge path for accepted contributions

Once the PR is approved:

1. Verify the contributor has signed the CLA (check the `CLA` label or check status).
2. Squash-merge the PR:
   ```bash
   gh pr merge <PR-number> --squash --delete-branch
   ```
   Or via web UI:
   - Click **Merge pull request** → select **Squash and merge**.
   - Edit the commit message to follow the convention:
     ```
     docs(009): add SOP for contribution process

     Closes #NN
     Co-authored-by: Contributor Name <contributor@example.com>
     ```
   - Click **Squash and merge**.
3. Add the contributor to the README contributors list (if applicable).
4. Thank the contributor publicly in a comment.

### 5.7 Rejection path for declined contributions

If the contribution is out of scope or does not meet quality standards:

1. Post a clear, constructive comment explaining the reason for rejection with references to relevant policies.
2. Apply the `invalid` or `wontfix` label.
3. Close the PR without merging:
   ```bash
   gh pr close <PR-number> --comment "Reason for closing..."
   ```
4. If the contributor is interested in revising, invite them to open a new PR after addressing the feedback.

### 5.8 Repository-specific considerations

| Repository | Validation steps | Special considerations |
|---|---|---|
| `zs-docs` | Run all 4 validation scripts (refresh, validate, status, cross-refs) | ZUSS compliance mandatory. Status must move from skeleton to draft. |
| `zs-platform` | Run Go build (`go build ./...`), Go tests (`go test ./...`), linter (`golangci-lint`) | Must not break existing API contracts. Go-native only (no JVM/HAPI FHIR). |
| `zs-zarish-index` | Validate metadata schema compliance. Run schema validator against → **[003-metadata-schema.md](004-zs-index/003-metadata-schema.md)**. | Entries must use the correct metadata schema. Domains must reference the 40-domain taxonomy. |
| `zs-zarish-standards` | Validate transformation model compliance. Run pipeline integrity checks. | Standards must be transformable through the G2A pipeline. |

---

## 6. Expected outcome

- Accepted contributions are merged into `main` with proper attribution.
- Rejected contributions receive a clear explanation and are closed gracefully.
- All merged contributions pass ZUSS compliance (docs) or build/tests (code).
- The PR description and commit history accurately reflect the contribution.
- The contributor is acknowledged for their work.

---

## 7. Escalation

| Issue | Action |
|---|---|
| Contributor disputes the rejection | Provide specific citations from the Constitution or ZUSS rules. Offer to discuss via email (security@zarishsphere.com). |
| PR introduces security vulnerability | Close the PR immediately. Do not merge. Assess the vulnerability privately. Contact the contributor via private email. |
| Fork PR contains merge conflicts | Request the contributor to rebase: `git fetch upstream && git rebase upstream/main`. Offer guidance if needed. |
| CLA not signed | Block merge until CLA is signed. Use the `CLA` check on the PR. Post a reminder comment. |
| Large PR with many unrelated changes | Request the contributor to split into smaller, focused PRs. Reference the contributor guidelines. |

---

## Cross-references

- → **[004-contributor-guidelines.md](002-zs-foundation/004-contributor-guidelines.md)** — Contributor expectations and guidelines
- → **[001-zarishsphere-constitution.md](001-zs-meta/001-zarishsphere-constitution.md)** — Constitution: 12 laws governing scope
- → **[002-domain-taxonomy-40.md](004-zs-index/002-domain-taxonomy-40.md)** — The 40 indexed ZarishIndex domains
- → **[003-metadata-schema.md](004-zs-index/003-metadata-schema.md)** — ZarishIndex entry metadata schema
- → **[002-sop-github-workflow.md](009-zs-operations/002-sop-github-workflow.md)** — SOP-002: Internal GitHub workflow (squash merge, branch protection)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
