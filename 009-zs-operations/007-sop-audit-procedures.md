# 007-sop-audit-procedures.md
## SOP-007: Maintaining and verifying the ZarishSphere decision audit trail
### Standard Operating Procedure — audit procedures

**Document type:** SOP
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Scope](#2-scope)
3. [Roles & Responsibilities](#3-roles--responsibilities)
4. [Preconditions](#4-preconditions)
5. [Procedure](#5-procedure)
6. [Expected Outcome](#6-expected-outcome)
7. [Escalation](#7-escalation)

---

## 1. Purpose

To establish mandatory procedures for maintaining, reviewing, and verifying the complete audit trail of all ZarishSphere governance and platform decisions. Every decision — technical, governance, operational, or strategic — must be recorded in a verifiable, immutable, and time-stamped format. The primary audit record is the git commit history of `zs-docs`, supplemented by Architecture Decision Records (ADRs) in `008-zs-adrs/`, Standard Operating Procedures (SOPs) in `009-zs-operations/`, and GitHub issues. This SOP implements Constitution Law 10.

> **Constraint:** Every decision is auditable forever. There is no statute of limitations on a ZarishSphere decision. Verbal or undocumented decisions are not considered valid.

---

## 2. Scope

**In scope:** All ZarishSphere repositories:

- `zarishsphere/zs-docs` — documentation, governance, ADRs, SOPs
- `zarishsphere/zs-platform` — platform implementation
- `zarishsphere/zs-zarish-index` — ZARISH-INDEX implementation
- `zarishsphere/zs-zarish-standards` — ZARISH-STANDARDS implementation
- `zarishsphere/zs-fhir-hub` — FHIR Integration Hub
- `zarishsphere/zs-home` — landing site
- Any future `zarishsphere/zs-*` repositories

**In scope (activities):**

- All governance decisions (Constitution amendments, policy changes)
- All technical decisions (architecture, framework, dependency choices)
- All operational decisions (credential rotation, deployment approvals)
- All ADR creation, supersession, and deprecation
- All SOP creation, updating, and deprecation
- All pull requests and their review histories
- All GitHub issues tagged with decision-related labels

**Out of scope:** Personal decisions of individual contributors that do not affect the ecosystem. Implementation-level code comments and inline documentation (these are covered by standard development practices).

---

## 3. Roles & Responsibilities

| Role | Who | Responsibility |
|---|---|---|
| **Repository Owner** | Mohammad Ariful Islam | Ensures all decisions are documented; ultimate authority for audit findings |
| **Contributor** | Any ecosystem contributor | Documents decisions in commits and PRs; follows this SOP |
| **Auditor** | Designated reviewer (may be AI agent or human) | Conducts quarterly and annual audit reviews; files audit findings as GitHub issues |
| **Reviewer Agent** | AI agent (reviewer) | Automated ZUSS compliance checks; validates cross-references and front matter |
| **GitHub** | GitHub platform | Hosts the immutable audit log (git history, issues, PRs, Actions logs) |

---

## 4. Preconditions

- The `zarishsphere` GitHub organisation exists (→ **[002-github-org-architecture.md](006-zs-infrastructure/002-github-org-architecture.md)**).
- All repositories are initialised with `main` branch protection enabled.
- → **[003-adr-github-as-government.md](008-zs-adrs/003-adr-github-as-government.md)** — ADR-003 is adopted: GitHub is the governance control plane.
- ZUSS compliance validation is configured (→ **[004-sop-zuss-compliance-audit.md](009-zs-operations/004-sop-zuss-compliance-audit.md)**).
- All contributors have Git configured with their real name and email (`git config user.name` and `user.email`).
- (Recommended) GPG signing keys are configured for commit signing.

---

## 5. Procedure

### 5.1 Every decision is a commit

No ZarishSphere decision is valid unless recorded in a Git commit. Verbal decisions, undocumented Slack messages, or unrecorded conversations do not constitute valid decisions.

**To record a decision:**

1. Make the change in the repository (e.g., create an ADR, update a policy, modify an SOP).
2. In **VS Code → Source Control tab** (`` Ctrl+Shift+G ``), review the staged changes.
3. Write a commit message that clearly documents the decision:

   ```
   <type>(<scope>): <imperative description of the decision>

   <rationale — why this decision was made, what alternatives were considered>

   References: ZS-XXX
   ```

   Examples:
   ```
   docs(008): add ADR-013 selecting PostgreSQL as primary database

   PostgreSQL was chosen over SQLite (Plane 0/1 constraint) and CockroachDB
   (operational complexity) due to its maturity, ecosystem support, and
   suitability for Planes 2-4 deployments.

   References: ZS-ADR-013
   ```

4. Commit and push following the GitHub workflow (→ **[002-sop-github-workflow.md](009-zs-operations/002-sop-github-workflow.md)**).

**Best practices for audit-ready commits:**

- Use signed commits with GPG to verify authorship:
  ```bash
  git commit -S -m "docs(008): add ADR-013 selecting PostgreSQL"
  ```
  Configure GPG signing in GitHub: **Settings → SSH and GPG keys → New GPG key**.
- Never amend or force-push commits that have been reviewed or merged.
- If a correction is needed, make a new commit with a message like `fix(008): correct ADR-013 database version requirement`.

### 5.2 Pull request audit trail

Every change to `zs-docs` must go through a pull request. The PR itself becomes part of the audit trail.

**To ensure a complete PR audit trail:**

1. Open the repository on GitHub at **https://github.com/zsdotcom/zs-docs**.
2. Click **Pull requests → New pull request**.
3. Fill in the PR template (if configured) or use the following required sections:

   ```markdown
   ## Summary

   Brief description of the decision and change.

   ## Decision rationale

   Why is this change needed? What problem does it solve?

   ## Alternatives considered

   What other approaches were evaluated and why were they rejected?

   ## Cross-reference checklist

   - [ ] ADR created or updated (if applicable)
   - [ ] SOP updated (if applicable)
   - [ ] All cross-references resolve to existing files
   - [ ] ZUSS validation passes
   - [ ] No banned words

   ## Related

   Closes #ISSUE-NUMBER (if applicable)
   ```

4. Click **Create pull request**.

**The PR discussion becomes part of the audit trail.** All review comments, approvals, and change requests are preserved by GitHub and are considered part of the decision record.

### 5.3 Git log as the primary audit query

The first step in any audit is to examine the git log. The commit history is the immutable, time-stamped master record of all decisions.

**To perform a basic audit query:**

```bash
# View the full decision history
git log --oneline --since="2026-01-01" --until="2026-06-11"

# View all ADR-related changes
git log --oneline --all --grep="ADR"

# View all SOP-related changes
git log --oneline --all --grep="SOP"

# View commits by a specific author
git log --oneline --author="Mohammad Ariful Islam"

# View the full diff of a specific commit
git show <commit-hash> --stat
```

**To verify commit authorship (signed commits):**

```bash
# Verify GPG signature on the most recent commit
git verify-commit HEAD

# List all commits with signature information
git log --show-signature -5
```

**Checklist item:** ☐ Every commit in the log has a clear decision-oriented message.

**Checklist item:** ☐ Every commit has a verified author identity (signed commits preferred, name+email required).

### 5.4 ADR supersession and deprecation

Architecture Decision Records are the primary mechanism for recording technical and governance decisions. When a decision is superseded, the old ADR is never deleted — a new ADR is created that references and supersedes the old one.

**To supersede an existing ADR:**

1. Create a new ADR file in `008-zs-adrs/` following the naming convention (→ **[001-sop-new-document-creation.md](009-zs-operations/001-sop-new-document-creation.md)**).
2. In the new ADR front matter, add:
   ```yaml
   supersedes:
     - "ZS-00N-ADR"
   ```
3. In the new ADR's **Reason** or **Context** section, explicitly state which ADR is being superseded and why:

   ```
   ## 3. Context

   This decision supersedes ADR-00X ([filename.md]) which previously
   selected [old technology/approach]. The supersession is necessary because
   [rationale].
   ```

4. Update the superseded ADR's front matter to mark its status:
   ```yaml
   status: "superseded-by-ZS-00N-ADR"
   ```

5. Add an entry in the superseded ADR's document body noting the supersession:

   ```
   ## Status

   This ADR has been superseded by a new ADR (file named per the convention in → **[001-sop-new-document-creation.md](009-zs-operations/001-sop-new-document-creation.md)**).
   ```

6. Update the ADR index (maintained in the `008-zs-adrs/` domain) to reflect the new status.

**Rules for ADR supersession:**

- Never delete an ADR file. Deletion destroys the audit trail.
- Never edit a merged ADR to change its decision. Create a new ADR instead.
- Minor corrections (typos, formatting) are permitted via PR with a `fix(008):` commit message.
- The supersession chain must be traceable: each ADR links forward to its superseder.

### 5.5 Quarterly audit review

Once every quarter, a designated Auditor (human or AI agent) must conduct a review of all ADRs, SOPs, and the decision audit trail.

**To perform the quarterly audit review:**

1. **Run the full validation pipeline** (→ **[004-sop-zuss-compliance-audit.md](009-zs-operations/004-sop-zuss-compliance-audit.md)**):

   ```bash
   python3 scripts/010-refresh-files.py
   bash scripts/001-zuss-validate.sh
   bash scripts/002-pipeline-status.sh
   bash scripts/003-resolve-cross-refs.sh
   ```

   **Checklist item:** ☐ All 4 scripts pass with exit code 0.

2. **Review all ADRs for supersession status:**

   ```bash
   # List all ADRs and their statuses
   grep -r "status:" 008-zs-adrs/*.md | grep -v index.md
   ```

   **Checklist item:** ☐ No ADR has `status: "draft"` for more than 2 quarters.

3. **Review all SOPs for accuracy:**

   ```bash
   # List all SOPs and their last_updated dates
   grep -r "last_updated:" 009-zs-operations/*.md | grep -v index.md
   ```

   **Checklist item:** ☐ Every SOP's `last_updated` is within the past 12 months.

4. **Review the git log for undocumented decisions:**

   ```bash
   # Look for commits that mention decisions without ADR references
   git log --oneline --all --grep="decision" --since="<first-day-of-quarter>"
   ```

   **Checklist item:** ☐ All decision-related commits reference an ADR or SOP.

5. **Verify cross-reference integrity:**

   ```bash
   bash scripts/003-resolve-cross-refs.sh
   ```

   **Checklist item:** ☐ All cross-references resolve to existing files.

6. **Review the credential inventory** (→ **[006-sop-credential-succession.md](009-zs-operations/006-sop-credential-succession.md)**):

   **Checklist item:** ☐ Credential inventory is current. No rotation dates are overdue.

7. **File a quarterly audit report:**

   ```bash
   gh issue create \
     --title "Quarterly audit review: QN-2026" \
     --body "## Audit summary

   - **Period:** [first date] — [last date]
   - **Auditor:** [name / agent ID]
   - **Validation scripts:** ✅ All pass
   - **ADR status:** [count] active, [count] superseded, [count] draft
   - **SOP currency:** [count] current, [count] needs update
   - **Credential inventory:** ✅ All current / ⚠️ [issues found]
   - **Findings:** [list any findings]
   " \
     --label "audit" \
     --label "quarterly-review"
   ```

   Or via the GitHub web UI:
   1. Open **https://github.com/zsdotcom/zs-docs/issues**.
   2. Click **New issue**.
   3. Select the **Audit report** template (if configured) or paste the body above.
   4. Add labels `audit` and `quarterly-review`.
   5. Click **Submit new issue**.

### 5.6 Annual external audit

Once per year, a designated Auditor who has not been involved in the day-to-day operations of the ZarishSphere ecosystem must conduct a comprehensive external audit.

**Scope of the annual audit:**

- ☐ **ADR completeness** — Every significant decision has a corresponding ADR. No decision was implemented without documentation.
- ☐ **ADR supersession chain** — Every superseded ADR correctly links to its successor. No broken chains.
- ☐ **Credential rotation** — Every credential has been rotated within the past 90 days (→ **[006-sop-credential-succession.md](009-zs-operations/006-sop-credential-succession.md)**).
- ☐ **SOP accuracy** — Every SOP accurately reflects current procedures. No SOP describes a process that has changed without the SOP being updated.
- ☐ **SOP coverage** — No critical operational procedure lacks an SOP.
- ☐ **Undocumented decisions** — A random sample of 10% of commits since the last annual audit are reviewed to ensure each documents a valid decision or refers to one.
- ☐ **Cross-reference integrity** — 100% of cross-references resolve correctly.
- ☐ **ZUSS compliance** — 100% of documents pass ZUSS validation.
- ☐ **Branch protection** — All repositories have `main` branch protection enabled.

**To conduct the annual audit:**

1. The Auditor creates a GitHub issue with the label `audit` and `annual-audit`:

   ```bash
   gh issue create \
     --title "Annual audit: 2026" \
     --body "See checklist in SOP-007 §5.6" \
     --label "audit" \
     --label "annual-audit"
   ```

2. The Auditor works through each checklist item, updating the issue with findings.
3. Each finding is filed as a separate GitHub issue with labels `audit`, `finding`, and `remediation`.
4. The audit is complete when the Auditor files a summary report issue with no unresolved findings.

### 5.7 Audit findings and remediation

All audit findings — from quarterly or annual reviews — are documented as GitHub issues with specific labels.

**To file an audit finding:**

1. Open **https://github.com/zsdotcom/zs-docs/issues**.
2. Click **New issue**.
3. Set the title to a clear description of the finding.
4. In the body, include:
   - **Finding:** What is wrong or missing
   - **Severity:** Critical / High / Medium / Low
   - **Source:** Which audit identified this (quarterly QN-2026, annual 2026)
   - **Remediation:** What needs to be done to fix it
   - **Deadline:** By when it must be fixed
5. Add labels:
   - `audit` — identifies this as an audit-related issue
   - `finding` — identifies this as a finding (not a feature request or bug)
   - `remediation` — identifies this as requiring action
   - Severity label: `severity-critical`, `severity-high`, `severity-medium`, `severity-low`
6. Click **Submit new issue**.

**Finding resolution timeline:**

| Severity | Resolution deadline | Escalation if unresolved |
|---|---|---|
| Critical | 7 days | Immediate escalation to Repository Owner |
| High | 30 days | Escalation to Repository Owner after 30 days |
| Medium | 90 days | Escalation to Repository Owner after 90 days |
| Low | Next quarterly review | Documented as deferred |

**Remediation verification:**

After the finding is resolved, the Auditor verifies the fix and closes the issue with a comment:

```
Verified: [description of verification performed]. Finding resolved.
```

---

## 6. Expected Outcome

| Check | Frequency | Method |
|---|---|---|
| Validation pipeline passes | Every commit | Run `python3 scripts/010-refresh-files.py && bash scripts/001-zuss-validate.sh && bash scripts/003-resolve-cross-refs.sh` |
| Signed commits used | Every commit | Verify `git log --show-signature -1` shows a valid GPG signature |
| ADR supersession table is current | Quarterly | Query all ADR `status` fields and verify the supersession chain (→ Step 5.4) |
| SOP `last_updated` within 12 months | Quarterly | `grep "last_updated:" 009-zs-operations/*.md` — flag any older than 12 months |
| Audit findings resolved | Quarterly | Check all open issues with label `audit` and `finding` — verify none are past their resolution deadline |
| Credential inventory current | Quarterly | Run the verification checklist in → **[006-sop-credential-succession.md](009-zs-operations/006-sop-credential-succession.md)** §6 |
| No undocumented decisions | Annually | Random sample of 10% of commits since last audit — verify each commit documents a decision or references one |
| Branch protection enabled | Annually | Check **GitHub → [repo] → Settings → Branches** — verify `main` has required PRs and status checks |

---

## 7. Escalation

| Issue | Action |
|---|---|
| Audit finding unresolved after deadline | Escalate to Repository Owner via email at `security@zarishsphere.com`. File a GitHub issue with label `escalated`. |
| Finding unresolved after 90 days | Document as a constitutional issue in → **[001-zarishsphere-constitution.md](001-zs-meta/001-zarishsphere-constitution.md)** — file a constitutional amendment issue. |
| ADR supersession chain broken | Create a new ADR that restores the chain. Reference all affected ADRs. Run the validation pipeline to confirm all cross-references resolve. |
| Git commit with undocumented decision | Create an ADR retroactively documenting the decision. Update the commit message if possible (note: amending pushed commits is discouraged — add a follow-up commit instead). |
| Validation pipeline fails during audit | Do not proceed with the audit until the pipeline passes. File a `finding` issue. Fix the validation issue before continuing. |
| No Auditor available for annual audit | The Repository Owner must designate an Auditor within 14 days. If no suitable internal Auditor exists, engage an external reviewer. Document the designation in a GitHub issue. |
| Conflicting decisions found (two ADRs that contradict each other) | Create a new ADR that explicitly resolves the conflict, stating which ADR is superseded and why. Reference both conflicting ADRs. |
| SOP describes a procedure that no longer matches reality | Update the SOP immediately. If the discrepancy was caused by an undocumented decision, document that decision first (create an ADR or update an existing one), then update the SOP. |

---

## Cross-references

- → **[001-zarishsphere-constitution.md](001-zs-meta/001-zarishsphere-constitution.md)** — Law 10: "Every decision is auditable forever"
- → **[003-adr-github-as-government.md](008-zs-adrs/003-adr-github-as-government.md)** — ADR-003: GitHub as governance control plane
- → **[002-sop-github-workflow.md](009-zs-operations/002-sop-github-workflow.md)** — PR process and commit conventions
- → **[004-sop-zuss-compliance-audit.md](009-zs-operations/004-sop-zuss-compliance-audit.md)** — ZUSS compliance audit procedures
- → **[006-sop-credential-succession.md](009-zs-operations/006-sop-credential-succession.md)** — Credential rotation audit checks
- → **[002-github-org-architecture.md](006-zs-infrastructure/002-github-org-architecture.md)** — GitHub organisation setup and branch protection

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
