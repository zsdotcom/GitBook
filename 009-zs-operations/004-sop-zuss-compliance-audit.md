# 004-sop-zuss-compliance-audit.md
## SOP-004: Systematic ZUSS compliance and formatting audit checks
### Standard Operating Procedure — compliance audit

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

To define the procedure for running a systematic ZUSS compliance audit across the entire `zs-docs` repository, interpreting validation script results, and fixing common compliance issues before merging or deployment.

---

## 2. Scope

**In scope:** Full audit of all 67+ documents across all 10 folders in `zs-docs`. Running all 4 validation scripts. Fixing naming, numbering, front matter, footer, banned words, heading case, and cross-reference issues. Scheduled monthly audits and ad-hoc pre-merge audits.

**Out of scope:** Code repositories (`zs-platform`, etc.). Content accuracy review (handled by content review, not compliance audit). Stylistic preferences beyond ZUSS rules.

---

## 3. Roles

| Role | Who |
|---|---|
| **Auditor** | Reviewer Agent (AI) or human maintainer running the audit |
| **Fixer** | Documentation Agent (AI) or human contributor fixing detected issues |
| **Maintainer** | Mohammad Ariful Islam — approves audit reports and waives non-critical items |

---

## 4. Preconditions

- The repository is cloned locally and on the `main` branch (or the branch being audited).
- All 4 validation scripts exist in `scripts/`:
  - `scripts/010-refresh-files.py`
  - `scripts/001-zuss-validate.sh`
  - `scripts/002-pipeline-status.sh`
  - `scripts/003-resolve-cross-refs.sh`
- Python 3 and Bash are available.
- No uncommitted changes (audit should be run on a clean working tree).

---

## 5. Steps

### 5.1 When to run an audit

Run a full ZUSS compliance audit in these situations:

| Trigger | Frequency |
|---|---|
| Before merging any PR into `main` | Every PR |
| Scheduled repository health check | Monthly (first day of each month) |
| Before deployment to production | Every deployment |
| After bulk renames or restructures | On demand |
| When the repo is first pushed to GitHub | Once |

### 5.2 Step 1 — Run the refresh script

```bash
python3 scripts/010-refresh-files.py
```

**What it does:**
1. Normalises front matter — adds missing required fields to all documents in folders 004–009 and `001-zs-meta/004-zarishsphere-writing-rules.md`.
2. Regenerates `index.md` for all 10 folders plus root `index.md`.
3. Regenerates `llms.txt`.
4. Ensures every `.md` file ends with the canonical 3-line license footer.

**How to interpret output:**

| Output | Meaning | Action |
|---|---|---|
| `OK    file.md` | File is already correct | No action needed |
| `FIX   file.md` | File was repaired (missing field, footer, etc.) | Run the script again to verify stability |
| `UPDATE index.md` | Index was regenerated | Expected — indices always update after changes |
| Stack trace / exception | Malformed YAML or unexpected error | Open the reported file and check front matter integrity |

**Verify:** Run the script a second time. If no `FIX` lines appear (only `OK` and `UPDATE`), the repository is stable.

### 5.3 Step 2 — Run the ZUSS validator

```bash
bash scripts/001-zuss-validate.sh
```

This script runs 7 checks. Interpret each one:

**Check 1 — File naming (`═══ 1. File naming ═══`)**

Validates that every `.md` file (except `index.md`) matches the pattern `nnn-descriptive-name.md`.

| Output | Meaning | Fix |
|---|---|---|
| `✓ All non-index files match pattern` | All names are valid | None |
| `✗` with filename | File breaks naming rules | Rename the file. Use lowercase, hyphens, 3-digit prefix. Run `010-refresh-files.py` after rename. |

**Check 2 — Sequential numbering (`═══ 2. Sequential numbering ═══`)**

Checks that within each folder, numbers run sequentially from `001` to `N` with no gaps.

| Output | Meaning | Fix |
|---|---|---|
| `✓ 001-meta: sequential (001-007)` | All numbers in sequence | None |
| `✗ 009-operations: gap — missing 003` | A file number is skipped | Renumber files to fill gaps, or add the missing file. Run `010-refresh-files.py` after. |

**Check 3 — YAML front matter (`═══ 3. YAML front matter ═══`)**

Verifies each file has `---` delimiters and all 12 required fields (id, title, domain, doc-type, entity-type, summary, tags, version, status, last_updated, isolation_tier, audience).

| Output | Meaning | Fix |
|---|---|---|
| `✓ file.md` | Front matter is valid | None |
| `✗ file.md` | Missing or invalid front matter | Open the file. Verify YAML syntax (no tabs, colons after keys). Run `010-refresh-files.py` first — it may fix missing fields automatically. |

**Check 4 — License footer (`═══ 4. License footer ═══`)**

Confirms every `.md` file ends with the canonical 3-line footer.

| Output | Meaning | Fix |
|---|---|---|
| (no output) | All pass | None |
| `✗ file.md` | Missing or incorrect footer | Run `010-refresh-files.py` to auto-fix. If the file still fails, manually append the footer. |

**Check 5 — Banned words (`═══ 5. Banned words ═══`)**

Searches for the three ZUSS-prohibited words outside code block and backtick contexts.

| Output | Meaning | Fix |
|---|---|---|
| (no output) | No banned words found | None |
| `✗ file.md:line` with highlighted word | Banned word found | Replace with an approved alternative that does not appear on the ZUSS ban list. |

**Check 6 — Heading case (`═══ 6. Heading case ═══`)**

Heuristic check for headings that may not use sentence case.

| Output | Meaning | Fix |
|---|---|---|
| `⚠ file.md:line` — warning | Possible case issue | Review the heading. If it is a proper noun (e.g., "GitHub", "ZarishSphere", "FHIR R5"), the case is correct. If not, change to sentence case. |

**Note:** This check is advisory only (exit code is not affected by warnings).

**Check 7 — No `latest` tag (`═══ 7. No latest tag ═══`)**

Ensures no bare `latest` references exist outside code blocks.

| Output | Meaning | Fix |
|---|---|---|
| (no output) | All pass | None |
| `✗ file.md:line` | Bare `latest` reference found | Replace with a specific version number (e.g., `v1.0.0`). |

**Final verdict:**

| Exit code | Meaning |
|---|---|
| `0` | All checks pass. The repository is ZUSS-compliant. |
| `1` | At least one check failed. Fix the reported issues and re-run. |

### 5.4 Step 3 — Run the pipeline status script

```bash
bash scripts/002-pipeline-status.sh
```

**What it does:** Shows a table of all 10 folders with document counts and completion status.

**How to interpret:**

```
001-meta:    7 files (  7 authored,   0 skeleton)  ✅ Complete
002-foundation:    4 files (  4 authored,   0 skeleton)  ✅ Complete
...
009-operations:    6 files (  1 authored,   5 skeleton)  ⬜ Skeleton
```

| Icon | Meaning | Action |
|---|---|---|
| ✅ Complete | All files have content (status is `draft`, `stable`, or `review`) | None |
| ⬜ Skeleton | At least one file has `status: skeleton` | Prioritise writing content for skeleton files |
| ⚠ Partial | Mixed statuses | Check individual files |

### 5.5 Step 4 — Run the cross-reference resolver

```bash
bash scripts/003-resolve-cross-refs.sh
```

**What it does:** Finds every `→ **[filename.md]**` cross-reference pattern across all `.md` files and verifies the target file exists.

**How to interpret:**

| Output | Meaning | Fix |
|---|---|---|
| `✓ file.md — all refs valid` | All cross-references in this file resolve | None |
| `✗ file.md — NN broken refs` | Cross-references point to non-existent files | For each broken ref: either create the missing file or update the reference to point to the correct path. |
| `SUMMARY: X files, Y refs, Z errors` | Final tally | If Z > 0, fix all broken refs and re-run. |

### 5.6 Common issues and fixes

| Issue | Cause | Fix |
|---|---|---|
| `FIX` in refresh script for many files | Script version mismatch or new required fields added | Accept the fixes. Run the script again to confirm stability. |
| Check 2 fails: "gap — missing NNN" | A file was deleted without renumbering | Add the missing file or renumber all files in that folder to close the gap. |
| Check 3 fails: YAML front matter missing | New document created without front matter | Add the full YAML front matter block. Use the template in → **[001-sop-new-document-creation.md](009-zs-operations/001-sop-new-document-creation.md)**. |
| Check 4 fails: footer missing after refresh | File has a non-canonical footer variant | Manually replace the footer with the canonical 3-line block. |
| Cross-refs broken after rename | File was renamed but references not updated | Search for `→ **[old-name.md]**` across all files and update to `→ **[new-name.md]**`. |
| Banned word in front matter `summary` field | Summary contains a ZUSS-prohibited word | Edit the summary field in the YAML front matter. |

### 5.7 Generate the audit report

After running all 4 scripts, compile the results into a structured audit report:

```markdown
## ZUSS compliance audit report

**Date:** YYYY-MM-DD
**Branch:** main
**Auditor:** <name>

### Results

| Check | Status |
|---|---|
| 010-refresh-files.py | ✅ Pass / ⚠ Fixed N files |
| 001-zuss-validate.sh — naming | ✅ / ✗ |
| 001-zuss-validate.sh — numbering | ✅ / ✗ |
| 001-zuss-validate.sh — YAML front matter | ✅ / ✗ |
| 001-zuss-validate.sh — footer | ✅ / ✗ |
| 001-zuss-validate.sh — banned words | ✅ / ✗ |
| 001-zuss-validate.sh — heading case | ⚠ N warnings |
| 001-zuss-validate.sh — latest tag | ✅ / ✗ |
| 002-pipeline-status.sh | N skeletons remaining |
| 003-resolve-cross-refs.sh | ✅ / ✗ N broken |

### Issues found

1. [file.md] — issue description
2. [file.md] — issue description

### Actions taken

- [ ] Issue 1 fixed
- [ ] Issue 2 fixed
- [ ] All clear — repo is ZUSS-compliant
```

Post this report as a comment on the PR or save to a file for the monthly review.

---

## 6. Expected outcome

- All 4 validation scripts pass without `✗` failures.
- Every file in the repository is ZUSS-compliant (naming, numbering, front matter, footer, banned words, cross-refs).
- Audit report is generated and stored for the record.
- Skeleton files are identified and prioritised for content writing.
- The repository is ready for merge and/or deployment.

---

## 7. Escalation

| Issue | Action |
|---|---|
| Validation script reports false positive | Check the script version. If the rule is incorrectly flagging a valid case, open an issue in the repository. |
| Widespread YAML failures after bulk edit | Restore from Git: `git checkout -- <files>`. Re-apply edits one file at a time. |
| Cross-reference loop detected | A file references itself or files reference each other without reaching an existing target. Break the loop by fixing one reference to point to an actual file. |
| Script itself is broken or missing | Verify the working directory is the repository root. Run `ls scripts/` to confirm files exist. Check file permissions: `chmod +x scripts/*.sh` if needed. |
| Cannot fix all issues before deadline | Document remaining issues in the audit report. Prioritise critical failures (naming, front matter, broken refs). Flag non-critical issues for the next cycle. |

---

## Cross-references

- → **[001-sop-new-document-creation.md](009-zs-operations/001-sop-new-document-creation.md)** — SOP-001: Front matter template used by Check 3
- → **[003-sop-contribution-process.md](009-zs-operations/003-sop-contribution-process.md)** — SOP-003: Audit pipeline run on every contribution
- → **[007-sop-audit-procedures.md](009-zs-operations/007-sop-audit-procedures.md)** — SOP-007: Quarterly and annual audit reviews that reuse this pipeline
- → **[004-zarishsphere-writing-rules.md](001-zs-meta/004-zarishsphere-writing-rules.md)** — ZUSS standard enforced by this audit

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
