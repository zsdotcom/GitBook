# 007-adr-markdown-first-documentation.md
## ADR-007: Plaintext Markdown-First Documentation Architecture
### Every ZarishSphere document is a markdown file with YAML front matter, git-versioned, AI-readable

**Document type:** ADR
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Accepted

---

## Table of Contents

1. [Decision](#1-decision)
2. [Context](#2-context)
3. [Alternatives Considered](#3-alternatives-considered)
4. [Reason for Decision](#4-reason-for-decision)
5. [Consequences](#5-consequences)
6. [Status](#6-status)
7. [Cross-references](#7-cross-references)

---

## 1. Decision

All ZarishSphere documentation is authored, stored, and versioned as plaintext Markdown files with YAML front matter, following the ZarishSphere Universal Serialization Standard (ZUSS). No proprietary document formats, SaaS documentation platforms, or build-dependent documentation generators are used. Documentation is stored in git repositories alongside configuration, with `llms.txt` files providing AI-consumable index summaries.

---

## 2. Context

The ZarishSphere ecosystem requires a documentation architecture that satisfies multiple constraints:

- **Constitution Law 2** (documentation precedes existence): No system may be built before its specification is committed. Documentation is primary; code is secondary.
- **Constitution Law 10** (every decision is auditable forever): Documentation must be versioned, immutable, and retain full history.
- **Constitution Law 5** (zero-cost): No paid documentation platform.
- **Constitution Law 8** (deployment plane sovereignty): Documentation must be accessible offline, without internet, on any deployment plane.
- **Constitution Law 12** (borderless contribution): Anyone with a browser and a GitHub account must be able to contribute a documentation change.
- **AI agent consumption:** The repo is designed for programmatic reading by opencode agents. AI agents natively understand Markdown and YAML front matter.

The documentation must also serve multiple audiences: the founder (sole builder), future contributors, deployers in the field, and AI agents working on the codebase.

---

## 3. Alternatives Considered

| Alternative | Pros | Cons |
|---|---|---|
| **Plain Markdown + git + ZUSS** | Universal readability; git-versioned with full history; works offline (clone the repo); zero-cost; AI-readable natively; ZUSS ensures consistency; any text editor works; no build step for reading; simple PR workflow for contributions | No built-in search (rely on GitHub search or grep); no WYSIWYG editing; no automatic table of contents navigation; less visual than hosted platforms; cross-reference validation requires custom scripts |
| **GitBook** | Nice UI, automatic ToC and search, good developer docs | Requires GitBook account (SaaS dependency); limited free tier (3 users on free plan); documents not fully portable without GitBook renderer; no offline access without internet; violates Law 5 (zero-cost beyond small team) and Law 8 (offline sovereignty) |
| **Notion** | Excellent WYSIWYG editing, capable databases, rich media support | Proprietary format — documents cannot be git-versioned; no offline full-text search in free tier; export is lossy (markdown export drops databases); no git integration; AI agents cannot natively consume Notion pages without API access; violates Law 10 (auditability — no version history) |
| **Docusaurus / VitePress** | Generates beautiful documentation sites from Markdown; built-in search, versioning, i18n | Requires Node.js build step; adds complexity to the documentation pipeline; CI/CD dependency to generate the site; extra 20+ npm dependencies; exceeds what a documentation-only repo needs; separate from git reading workflow |
| **GitHub Wiki** | Integrated with GitHub, no setup, markdown-based | Not versioned as part of repo (separate git repo); cannot be cloned with the codebase; no YAML front matter support; limited structure (no subfolders); AI agents cannot navigate consistently |

---

## 4. Reason for Decision

1. **Law 2 alignment:** Documentation must be committed before any system exists. Markdown in git is the most direct implementation of this constitutional law. A document is authored in a text editor, committed as a `.md` file, and exists as part of the repository. No extra steps, no platform dependency.

2. **AI agent native readability:** The repository is designed for the `.opencode/` agent ecosystem. AI agents natively read and write Markdown. YAML front matter provides structured metadata (id, status, tags, capabilities) that agents parse programmatically. An `llms.txt` file at the repo root provides a flat, AI-consumable index. No API calls, no SDK integrations — just file I/O.

3. **Offline access:** Every deployment plane can carry a full copy of the documentation via `git clone`. Plane 0 (air-gapped) has the complete documentation without any internet connection. This is structurally impossible with SaaS documentation platforms.

4. **Zero cost with maximum durability:** Markdown files on GitHub are free, unlimited in number, and will outlast any documentation platform. A plaintext file from 1970 is still readable today. A Notion export from 2026 may not be. This aligns with Constitution Law 11 (platform outlives its creators).

5. **Contribution path:** A community health worker with a GitHub account and basic Markdown knowledge can submit a documentation fix via a pull request through the GitHub web editor. No tool installation required. This directly serves Constitution Law 12 (borderless contribution).

6. **ZUSS standard:** The ZUSS writing rules (→ **001-zs-meta/004-zarishsphere-writing-rules.md** — naming, front matter, structure, cross-references, and formatting) provide a complete specification for naming, front matter, structure, cross-references, and formatting. This ADR operationalizes ZUSS across all documentation.

---

## 5. Consequences

**Positive:**

- Complete documentation archive with full git history from the first commit
- Offline-capable on every deployment plane
- AI agents read and write documentation directly without API integration
- Contribution barrier is as low as possible (GitHub web editor + Markdown)
- No recurring cost for documentation hosting (GitHub Pages free tier)
- Custom validation scripts (scripts/001-zuss-validate.sh, 003-resolve-cross-refs.sh) enforce consistency automatically

**Negative:**

- No built-in visual navigation or search (rely on index.md files and grep)
- Cross-references between files must be manually maintained (mitigated by 003-resolve-cross-refs.sh script)
- No WYSIWYG editing — contributors must know Markdown syntax (mitigated by ZUSS reference docs)
- Large file reading performance — very large documents can be slow in browser (mitigated by keeping docs focused)
- `llms.txt` must be regenerated when documents are added or moved (automated via scripts/010-refresh-files.py)

---

## 6. Status

Accepted. This ADR implements Constitution Law 2 (documentation precedes existence) and provides the technical foundation for Law 10 (every decision is auditable forever). The ZUSS standard (→ **001-zs-meta/004-zarishsphere-writing-rules.md** — the normative rulebook this ADR activates) is the normative rulebook that this ADR activates.

---

## 7. Cross-references

→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS: naming, front matter, structure, cross-references, formatting
→ **008-zs-adrs/003-adr-github-as-government.md** — ADR-003: GitHub as the operational control plane
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 2 (documentation precedes existence), Law 5, Law 8, Law 10, Law 12

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
