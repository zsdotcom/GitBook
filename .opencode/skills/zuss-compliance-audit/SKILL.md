---
name: zuss-compliance-audit
description: Audits finished pages in gitbook-zarishsphere/ against the ZUSS writing-rules standard and reports a pass/fail score per document with specific fixes. Use as the last step before considering any page or batch "done", and periodically to re-check the whole space.
---

# ZUSS compliance audit

## When to use
- Immediately after `gitbook-restructure` writes or rewrites any page
- Whenever Ariful asks for a compliance score or a repeat of the earlier
  63/100-style audit, but against the full `gitbook-zarishsphere/` output
  instead of four core documents

## What it checks (read the live standard first — do not rely on memory of it)
Always re-read `_raw/docs/001-zs-meta/004-zarishsphere-writing-rules.md`
before scoring, since it is the single source of truth and may have changed
since the last audit. At minimum, check each page for:

1. Mandatory header block present and correctly filled in
2. 3-digit zero-padded index prefix in the filename
3. Lowercase-hyphenated filename (no spaces, no camelCase, no underscores)
4. Table of Contents present if the document has more than 5 sections
5. Mandatory license footer present
6. No banned words: "genuinely", "honestly", "straightforward"
7. No internal cross-reference pointing at a path that doesn't exist
8. No unresolved placeholder text (TODO, TBD, Lorem ipsum, etc.)
9. GitBook-specific: page is actually linked from `SUMMARY.md`; internal
   links are relative and resolve to a real file

## Output format
Report one row per audited document:

```markdown
| Document | Score | Failing checks | Fix required |
|---|---|---|---|
| 002-zs-foundation/001-foundation-charter.md | 92/100 | missing license footer | add footer per ZUSS §X |
```

Then a total for the batch. Never round a failing check up to "close enough"
— either the check passes exactly as ZUSS defines it, or it's listed as a
failure with the specific fix needed.

## Hard constraints
- This skill only reads and reports; it does not fix issues itself. Hand
  fixes back to the `editor` agent / `gitbook-restructure` skill.
- Never audit from a cached memory of what ZUSS says — re-read the current
  `004-zarishsphere-writing-rules.md` every time, since it is a living
  document that may be revised between audits.
