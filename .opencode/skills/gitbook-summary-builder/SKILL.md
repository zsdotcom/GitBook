---
name: gitbook-summary-builder
description: Builds or updates gitbook-zarishsphere/SUMMARY.md, README.md, and .gitbook.yaml so the folder is a valid GitBook Git Sync project. Use whenever a page is added, moved, renamed, or removed from gitbook-zarishsphere/, or when setting up the space for the first time.
---

# GitBook summary builder

## When to use
- First-time setup of `gitbook-zarishsphere/` as a Git Sync project
- Any time a page is added, renamed, renumbered, or removed, so `SUMMARY.md`
  and `.gitbook.yaml` stay in sync with what's actually on disk
- Before a commit that will trigger a GitBook sync, as a pre-flight check

## Ground rules (verified against GitBook's own Git Sync documentation)
- `README.md` at the project root is the space's landing page. Once Git Sync
  is enabled, it must be edited only in this repository — never through the
  GitBook web editor, or the next sync will conflict.
- `SUMMARY.md` is the literal table of contents. A page that exists on disk
  but is not linked from `SUMMARY.md` will not appear in the published space.
  If `SUMMARY.md` is missing entirely, GitBook infers structure from folders
  — never rely on this; always ship an explicit file.
- `SUMMARY.md` syntax: `#` title, then `##` group headings, then nested
  bullet links:
  ```markdown
  # Summary

  ## Foundation
  * [Foundation Charter](002-zs-foundation/001-foundation-charter.md)
  * [Governance Model](002-zs-foundation/002-governance-model.md)
  ```
- `.gitbook.yaml` declares `root`, `structure.readme`, `structure.summary`,
  and any `redirects` needed when a page's path changes (old-path → new-path)
  so inbound links don't 404:
  ```yaml
  root: ./
  structure:
    readme: README.md
    summary: SUMMARY.md
  redirects:
    old-folder/old-page: new-folder/new-page.md
  ```

## Procedure
1. Read the current `gitbook-zarishsphere/` file tree.
2. Diff it against the current `SUMMARY.md` — flag any page on disk that
   isn't linked, and any link in `SUMMARY.md` pointing at a file that doesn't
   exist.
3. Regenerate `SUMMARY.md` to match the approved `architect` plan, preserving
   manually-curated group titles where they still apply.
4. If a page moved since the last sync, add a `redirects` entry in
   `.gitbook.yaml` rather than silently breaking the old link.
5. Run `markdownlint-cli2` against `SUMMARY.md` and `README.md` before
   reporting done.
