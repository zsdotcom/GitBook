---
name: raw-doc-classifier
description: Inventories every file under _raw/docs/**, extracting its ZUSS domain, slug, and a factual one-line summary, and writes a crosswalk index. Use this first, before any restructuring, whenever _raw/ content needs to be mapped, counted, or checked for gaps/duplicates. Read-only against _raw/.
---

# Raw doc classifier

## When to use
- Starting (or resuming) the `_raw` → `gitbook-zarishsphere` migration
- Ariful asks "what's in `_raw`", "what's covered under domain X", or "did we
  miss anything"
- Before the `architect` agent designs `SUMMARY.md`, so it has a full,
  accurate inventory to work from

## What it does
1. Walks `_raw/docs/<NNN-domain>/*.md` for all 10 domains.
2. For each file, records: path, 3-digit prefix, domain folder, and a
   one-line factual summary drawn only from that file's actual content —
   never inferred or invented.
3. Flags: files with broken-looking cross-references (a link to a path that
   doesn't exist under `_raw/`), files under 200 words (possible stubs), and
   any two files whose titles suggest overlapping scope.
4. Writes the result to `gitbook-zarishsphere/.internal/raw-inventory.md` —
   this file is a working artifact, never linked from `SUMMARY.md`, so it
   doesn't get published to the live GitBook space.

## Hard constraints
- Never edit, move, or write inside `_raw/`. Read-only tools only (`read`,
  `grep`, `glob`).
- Never guess at a file's content from its filename alone — open and read it.
- If a domain folder's `index.md` is missing or empty, say so explicitly
  rather than silently skipping it.

## Output format for `raw-inventory.md`
```markdown
# Raw inventory — <NNN-domain-name>

| File | Summary | Refs to | Flags |
|---|---|---|---|
| 001-foo.md | one-line factual summary | 002-bar.md | none |
```
