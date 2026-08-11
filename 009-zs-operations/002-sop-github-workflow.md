# 002-sop-github-workflow.md
## SOP-002: Repository branching, protecting, merging, and pull request models
### Standard Operating Procedure — GitHub workflow

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

To define the standard Git and GitHub workflow for all ZarishSphere repositories, including branch naming conventions, commit standards, pull request creation, review process, merge strategy, and branch protection rules.

---

## 2. Scope

**In scope:** All ZarishSphere repositories under the `zarishsphere` GitHub organisation: `zs-docs`, `zs-platform`, `zs-zarish-index`, `zs-zarish-standards`, `zs-fhir-hub`, `zs-home`, and any future `zs-*` repositories.

**Out of scope:** Fork-based contributions from external contributors (handled by → **[003-sop-contribution-process.md](009-zs-operations/003-sop-contribution-process.md)**). Direct pushes to `main` (prohibited by branch protection).

---

## 3. Roles

| Role | Who |
|---|---|
| **Developer** | Anyone making changes on a feature branch |
| **Reviewer** | Reviewer Agent (AI) or maintainer who verifies ZUSS compliance and content |
| **Maintainer** | Mohammad Ariful Islam — approves and merges PRs, manages branch protection |

---

## 4. Preconditions

- Git is installed and configured on the local machine (Lenovo i3 / Ubuntu).
- The `gh` CLI (GitHub CLI) is installed and authenticated:
  ```bash
  gh auth status
  ```
- The repository has been cloned:
  ```bash
  git clone git@github.com:zarishsphere/zs-docs.git
  ```
- VS Code is available as the primary editor.
- The repository has branch protection rules configured on GitHub (see step 5.6).

---

## 5. Steps

### 5.1 Branch naming conventions

All branches must follow the pattern:

```
<type>/<short-description>
```

| Type | When to use |
|---|---|
| `feat/` | New document, new feature |
| `fix/` | Correction to existing content |
| `refactor/` | Restructuring without content change |
| `chore/` | Maintenance, script updates, CI changes |
| `docs/` | Documentation-only changes |

Examples:
```
feat/sop-backup-recovery
fix/broken-cross-ref-in-index
chore/update-validation-scripts
docs/clarify-fhir-architecture
```

### 5.2 Create a feature branch

1. Ensure you are on `main` with the most recent changes:
   - **VS Code → Source Control tab (`` Ctrl+Shift+G ``) → click the branch name in the status bar** (bottom-left corner).
   - If not on `main`, click the branch name and select `main` from the dropdown.
2. Pull the most recent changes:
   - Open the terminal: **Terminal → New Terminal** (`` Ctrl+` ``).
   - Run:
     ```bash
     git pull origin main
     ```
3. Create and switch to a new branch:
   ```bash
   git checkout -b feat/sop-backup-recovery
   ```
   Alternatively, use the VS Code UI: **Click the branch name in status bar → Create new branch...** and type `feat/sop-backup-recovery`.

### 5.3 Make changes and commit

1. Edit files in VS Code.
2. After editing, stage the changed files:
   - **VS Code → Source Control tab** → hover over each changed file → click the **`+` (Stage Changes)** icon.
   - Or use the terminal:
     ```bash
     git add 009-zs-operations/012-sop-backup-dr.md
     ```
3. Run validation scripts before committing:
   ```bash
   python3 scripts/010-refresh-files.py
   bash scripts/001-zuss-validate.sh
   bash scripts/003-resolve-cross-refs.sh
   ```
   Confirm all pass (exit code 0).
4. Commit with a descriptive message:
   - **VS Code → Source Control tab** → type the commit message in the text box above the staged files → click **Commit**.
   - Commit message format:
     ```
     <type>(<scope>): <imperative description>

     <optional body>
     ```
     Examples:
     ```
     docs(009): add SOP-004 for ZUSS compliance audit

     Full SOP content with Purpose, Scope, Roles, Steps,
     Expected Outcome, and Escalation sections.
     ```
   - Or use the terminal:
     ```bash
     git commit -m "docs(009): add SOP-004 for ZUSS compliance audit"
     ```

### 5.4 Push the branch

Push the feature branch to GitHub:

```bash
git push origin feat/sop-backup-recovery
```

If this is the first push of a new branch, use:

```bash
git push -u origin feat/sop-backup-recovery
```

### 5.5 Create a pull request

Use the `gh` CLI to create a PR:

```bash
gh pr create \
  --title "docs(009): add SOP-004 for ZUSS compliance audit" \
  --body "## Summary

Adds full SOP content for ZUSS compliance audit procedures.

## Changes

- 009-zs-operations/004-sop-zuss-compliance-audit.md: complete rewrite from skeleton

## Validation

- [x] python3 scripts/010-refresh-files.py
- [x] bash scripts/001-zuss-validate.sh
- [x] bash scripts/003-resolve-cross-refs.sh

## Related

Closes #NN" \
  --base main
```

Alternatively, use the GitHub web UI:
1. Push the branch (step 5.4).
2. Open `https://github.com/zsdotcom/zs-docs` in a browser.
3. A yellow banner appears: **"feat/sop-backup-recovery had recent pushes"** → click **Compare & pull request**.
4. Fill in the PR title and body using the format above.
5. Click **Create pull request**.

### 5.6 Review process

1. The PR triggers the Reviewer Agent (AI) automatically if configured via GitHub Actions.
2. Manual review steps:
   - Open the PR in GitHub.
   - Click the **Files changed** tab.
   - Verify:
     - [ ] File naming follows ZUSS (`nnn-descriptive-name.md`)
     - [ ] YAML front matter has all required fields
     - [ ] Content follows the SOP format (Purpose, Scope, Roles, Preconditions, Steps, Expected Outcome, Escalation)
     - [ ] No banned words (enforced by ZUSS check #5)
     - [ ] All cross-references use the `→ **[file.md]**` format
     - [ ] Validation scripts pass (check the Actions tab)
3. If issues are found, leave comments on specific lines using the **`+`** icon next to the line number.
4. The author addresses comments and pushes additional commits.
5. Repeat validation after each push.

### 5.7 Merge the pull request

Once approved:

```bash
gh pr merge --squash --delete-branch
```

Or via web UI:
1. Open the PR in GitHub.
2. Click **Merge pull request** dropdown → select **Squash and merge**.
3. Edit the commit message if needed (it should match the branch purpose).
4. Click **Squash and merge**.
5. Click **Delete branch** (confirmation button).

**Merge strategy:** Always squash merge. This keeps `main` history linear and clean. Never use "Create a merge commit" or "Rebase and merge".

### 5.8 Branch protection configuration

To protect the `main` branch on GitHub:

1. Open `https://github.com/zsdotcom/zs-docs/settings/branches` in a browser.
2. Under **Branch protection rules**, click **Add rule**.
3. In **Branch name pattern**, enter `main`.
4. Enable the following settings:
   - ☑ **Require a pull request before merging**
     - ☑ **Require approvals** — set to 1
     - ☑ **Dismiss stale pull request approvals when new commits are pushed**
   - ☑ **Require status checks to pass before merging**
     - Search for and select the validation workflow (e.g., "ZUSS Validate" or "CI")
   - ☑ **Require branches to be up to date before merging**
   - ☑ **Include administrators**
   - ☑ **Restrict who can push to matching branches**
5. Click **Create** or **Save changes**.

### 5.9 Cleanup after merge

1. Delete the local branch:
   ```bash
   git checkout main
   git pull origin main
   git branch -d feat/sop-backup-recovery
   ```
2. Verify the remote branch was deleted:
   ```bash
   git fetch --prune
   ```
3. Verify the changes are visible on `main`:
   ```bash
   git log --oneline -5
   ```

---

## 6. Expected outcome

- All changes are committed, pushed, reviewed, and merged into `main`.
- The `main` branch is protected and never receives direct pushes.
- The commit history on `main` is linear (squash-merged PRs).
- Local and remote branches are cleaned up after merge.
- Validation scripts pass before every commit.

---

## 7. Escalation

| Issue | Action |
|---|---|
| `gh pr create` fails (not authenticated) | Run `gh auth login` and follow the browser-based authentication flow. |
| Merge conflict on PR | Pull `main`, rebase the feature branch: `git rebase main`, resolve conflicts in VS Code, `git add` resolved files, `git rebase --continue`. |
| Branch protection blocks the merge | Ensure all required status checks (validation scripts) pass. Check the **Checks** tab on the PR page. |
| Accidental push to `main` | Force-push is disabled. Create a revert PR: `git checkout -b fix/revert-accidental-push`, `git revert HEAD`, push, create PR. |
| Validation scripts fail in CI but pass locally | Check the CI runner logs in the **Actions** tab. Ensure the CI workflow uses the same script versions. |

---

## Cross-references

- → **[003-sop-contribution-process.md](009-zs-operations/003-sop-contribution-process.md)** — SOP-003: External contribution pipeline (fork-and-PR model)
- → **[001-sop-new-document-creation.md](009-zs-operations/001-sop-new-document-creation.md)** — SOP-001: Creating a ZUSS-compliant document before opening a PR
- → **[004-sop-zuss-compliance-audit.md](009-zs-operations/004-sop-zuss-compliance-audit.md)** — SOP-004: Validation scripts run before every commit
- → **[007-sop-audit-procedures.md](009-zs-operations/007-sop-audit-procedures.md)** — SOP-007: PR history as part of the decision audit trail

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
