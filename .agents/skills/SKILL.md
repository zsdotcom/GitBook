---
name: gitbook-restructure
description: Rewrites and reorganizes content from _raw/ into gitbook-zarishsphere/, merging or splitting source files as needed to fit an approved architect plan, without ever writing to _raw/. Use once a raw-doc-classifier inventory and an architect plan both exist.
---

# GitBook restructure

## When to use
- After `raw-doc-classifier` has produced an inventory and the `architect`
  agent's structure plan has been approved by Ariful
- When a single `gitbook-zarishsphere/` page needs to merge content from more
  than one `_raw/` file, or one `_raw/` file needs to be split across several
  output pages

## Preconditions (stop and ask if either is missing)
1. An approved architect plan exists (source-to-destination table).
2. Every `_raw/` file involved has already been read this session — never
   restructure from memory of an earlier summary.

## Procedure
1. For each destination page in the plan, gather every `_raw/` file it draws
   from and re-read them in full.
2. Draft the page content, reconciling any differences between source files
   explicitly (e.g. "source A and source B give different pinned versions for
   X — flagged for Ariful" rather than silently picking one).
3. Apply ZUSS formatting (header block, TOC if >5 sections, license footer,
   3-digit lowercase-hyphenated filename) and GitBook link conventions in the
   same pass — don't do a "ZUSS pass" and a "GitBook pass" separately, since
   that tends to produce pages that pass one audit and fail the other.
4. Write the file into `gitbook-zarishsphere/<NNN-domain>/<NNN-slug>.md`.
5. Hand off to `gitbook-summary-builder` to add the new page to `SUMMARY.md`.
6. Hand off to `zuss-compliance-audit` before marking the page complete.

## Hard constraints
- Never write, edit, or delete anything under `_raw/`.
- Never invent content to fill a gap between two source files — surface the
  gap to Ariful instead.
- Never carry over a banned word ("genuinely", "honestly", "straightforward")
  even if the source `_raw/` file itself contains one — those files predate
  ZUSS enforcement in some cases.
