# 004-zarishsphere-writing-rules.md
## ZarishSphere Universal Serialization Standard (ZUSS)
### Documentation, Naming, and Formatting Rules — V1

**Document type:** Normative Standard — V1
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative. All ZarishSphere documents, repos, and workflows must comply.

**Note:** This is a correction pass on the original 002-writing-rules.md, not a new version — per Section 9, no version increments occur pre-launch. Seven gaps found by auditing ZUSS against documents already in production (naming exemptions, frontmatter conflict, versioning contradiction, dead rules, missing validation, ID collisions, banned-word scope) are fixed below. Nothing here changes intent; it closes the gap between what ZUSS says and what the ecosystem already does.

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Core Naming Mechanics](#2-core-naming-mechanics)
3. [Tooling-Reserved Filenames](#3-tooling-reserved-filenames)
4. [Repository Naming (zs-* Catalog)](#4-repository-naming-zs--catalog)
5. [Entity Identifiers and ID Registry](#5-entity-identifiers-and-id-registry)
6. [Document Classes and Structure Rules](#6-document-classes-and-structure-rules)
7. [Writing Style Rules](#7-writing-style-rules)
8. [Technical Documentation Subtypes](#8-technical-documentation-subtypes)
9. [Version Policy](#9-version-policy)
10. [License Block (Mandatory Footer)](#10-license-block-mandatory-footer)
11. [Cross-Reference Standard](#11-cross-reference-standard)
12. [Validation and Enforcement](#12-validation-and-enforcement)

---

## 1. Purpose

ZUSS is the single, consistent rule set governing how every file, folder, repository, workflow, identifier, and document is named, structured, and written within the ZarishSphere ecosystem. Consistency at this level makes the ecosystem machine-explorable and human-navigable simultaneously.

Every repository matching the `zs-*` catalog format must align its taxonomy with ZUSS. No exceptions — except the tooling-reserved filenames explicitly listed in Section 3, which exist because external tools (GitHub, MkDocs, Copilot) require them.

---

## 2. Core Naming Mechanics

### 2.1 Universal Syntax Rules

| Rule | Requirement |
|---|---|
| Case | Lowercase only. No uppercase anywhere in file or folder names, except tooling-reserved filenames (Section 3). |
| Separator | Hyphen (`-`) only. No underscores, no spaces, no camelCase, no PascalCase. |
| Index prefix | All content files and folders begin with a 3-digit zero-padded sequence number: `001`, `002`, `099`. |
| Extension | Always explicit, exactly one: `.md`, `.yml`, `.json`, `.go`, `.yaml`. Never omit. Never double (`.md.md` is invalid). |
| Pattern | `nnn-descriptive-name.ext` |

**Valid examples:**
```
001-zarishsphere-constitution.md
002-zarishsphere-foundation-profile.md
003-zarishsphere-founder-profile.md
004-zarishsphere-writing-rules.md
005-zarishsphere-ecosystem-architecture.md
006-zarishsphere-glossary.md
099-zarishsphere-appendix-references.md
```

**Invalid examples:**
```
Profile Context.md              ← spaces + uppercase
profileContext.md               ← camelCase
001_profile_context.md          ← underscores
profile-context                 ← missing extension
001-overview.md.md              ← doubled extension
ZarishSphere_Ecosystem.md.md.md ← PascalCase, underscores, tripled extension
```

### 2.2 Folder Naming

Same rules as files, but without numbers and extensions:

```
zs-core/
zs-health/
zs-geography/
zs-index/
```

### 2.3 Workflow File Naming

CI/CD and automation workflow files follow a three-segment format:

```
[id]--[trigger]--[process].yml
```

| Segment | Rules | Example |
|---|---|---|
| `[id]` | 3-digit zero-padded integer | `101` |
| `[trigger]` | What fires the workflow (kebab-case) | `on-push`, `on-schedule`, `on-release` |
| `[process]` | What the workflow does (kebab-case) | `validate-markdown`, `build-artifacts`, `publish-pages` |
| Separator | Double hyphen `--` between each segment | — |

**Valid examples:**
```
101--on-push--validate-markdown.yml
102--on-schedule--sync-iso-data.yml
201--on-release--publish-pages.yml
301--on-pull-request--lint-yaml.yml
```

### 2.4 Asset Naming (Images, Diagrams, Non-Code Files)

Assets follow the same lowercase-hyphen rule, with a type prefix:

```
[type]-[nnn]-[descriptive-name].[ext]

Types: img (raster images), svg (vector/diagrams), pdf (documents), data (CSV/JSON seed files)
```

**Valid examples:**
```
img-001-fhir-engine-architecture.png
svg-002-g2a-pipeline-flow.svg
data-001-domain-taxonomy-seed.csv
```

---

## 3. Tooling-Reserved Filenames

The following filenames are **exempt** from Section 2.1 because external tooling requires their exact case and spelling to function. This is the one deliberate, permanent exception to ZUSS naming — not a violation.

| Filename | Reason for exemption |
|---|---|
| `README.md` | GitHub renders this automatically on repo landing pages |
| `LICENSE` / `LICENSE.md` | GitHub license detection requires this exact name |
| `CONTRIBUTING.md` | GitHub links this automatically in PR/issue templates |
| `CODE_OF_CONDUCT.md` | GitHub community health file convention |
| `SECURITY.md` | GitHub security tab convention |
| `CHANGELOG.md` | Standard tooling convention (Keep a Changelog, semantic-release) |
| `AGENTS.md` | AI agent bootstrap convention used across the ecosystem |
| `TODO.md` | Root-level roadmap file, referenced by name in AGENTS.md |
| `llms.txt` | Emerging AI-crawler convention; must be lowercase, no prefix |
| `mkdocs.yml` | MkDocs requires this exact filename |
| `.github/**` | GitHub Actions and templates require fixed paths under `.github/` |
| `index.md` | Standards indexing format for multiple platforms |
| `SKILL.md` | Standards SKILL format for multiple platforms and agents |

No other filename may deviate from Section 2.1. If a new tool requires a reserved name, add it to this table via a PR — do not create silent exceptions elsewhere.

---

## 4. Repository Naming (zs-* Catalog)

All ZarishSphere repositories follow the `zs-` prefix convention:

| Category | Pattern | Example |
|---|---|---|
| Core platform | `zs-core` | `zs-core` |
| Domain modules | `zs-[domain]` | `zs-health`, `zs-logistics` |
| FHIR engine | `zs-fhir-[component]` | `zs-fhir-server`, `zs-fhir-g2a` |
| Infrastructure | `zs-infra-[component]` | `zs-infra-cloudflare`, `zs-infra-k3s` |
| Content (data) | `zs-content-[type]` | `zs-content-forms`, `zs-content-protocols` |
| Documentation | `zs-docs` | `zs-docs` |
| ZarishIndex | `zs-index` | `zs-index` |
| ZarishStandards | `zs-standards` | `zs-standards` |

**Docker image naming:**

```
zarishsphere/[service-name]:[v1.0.0]
```

Example: `zarishsphere/zs-fhir-server:v1.0.0`

Never use a `latest` tag in any production or pinned config.

---

## 5. Entity Identifiers and ID Registry

All system entities use identifier patterns from a fixed namespace. Every namespace's prefix must be unique across the whole registry — check this table before adding a new one.

| Entity | Prefix | Pattern | Example | Owning system |
|---|---|---|---|---|
| FHIR Profile | `zs-` | `https://zarishsphere.com/fhir/StructureDefinition/zs-[resource]` | `zs-patient` | Platform |
| Form ID | `zs-form-` | `zs-form-[domain]-[name]-v1` | `zs-form-ncd-intake-v1` | Platform |
| Service URL | — | `https://[service].zarishsphere.com/[path]` | `https://api.zarishsphere.com/fhir/R5/` | Infrastructure |
| ADR | `ADR-` | `ADR-[NNN]-[title-kebab-case]` | `ADR-001-go-as-primary-language` | Governance |
| ZarishIndex Resources ID | `ZI-` | `ZI-[DOMAIN_CODE]-[NNNNN]` | `ZI-HEALTH-00001` | ZarishIndex (legacy internal ref) |
| ZarishIndex master ID | `[DOMAIN_CODE]-` | `[DOMAIN_CODE]-[ISSUER_CODE]-[SHORT_ID]-[YEAR]` | `HL-ISO-15189-2022` | ZarishIndex (canonical, per 003-zs-index-metadata-schema.md) |

**Collision rule:** No two owning systems may claim the same prefix. `ZI-` and the domain-code scheme (`HL-`, `HR-`, etc.) both belong to ZarishIndex and do not collide with each other or with `ZS-`/`zs-`/`ADR-`/`zs-form-`. Any new ID scheme must be added to this table with an explicit prefix check before use.

---

## 6. Document Classes and Structure Rules

ZUSS recognizes two document classes. Every document declares which class it is via its opening block. Do not mix the two schemas in one file.

### 6.1 Class A — Narrative Documents

Governance, architecture, ADRs, SOPs, direction papers — anything meant to be read top-to-bottom by a human first, an agent second. Use the **markdown header block**:

```markdown
# [nnn]-[document-name].md
## [Human-Readable Title]
### [Subtitle or Scope]

**Document type:** [Reference / Specification / Direction / Proposal / ADR / SOP / Report]
**Date:** [Full date — August 01, 2026]
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** [Apache 2.0 (code) · CC BY 4.0 (documentation) OR as applicable]
**Status:** V1 — [One-line status description]
```

### 6.2 Class B — Structured Entry Documents

Standards-index entries, catalog rows, anything meant to be parsed by a pipeline first and read by a human second. Use **YAML frontmatter**:

```yaml
---
id: "ZS-NNN-XXX"
title: "Document title"
domain: "domain-slug"
doc-type: "specification"
entity-type: "specification"
descriptions: >-
  One to three sentence machine-readable summary.
version: "1.0.0"
status: "stable"
tags: ["tag-one", "tag-two"]
isolation_tier: "global"
audience: ["contributors", "ai-agents"]
last_updated: "2026-08-01"
---
```

**Note on the `version` field:** this is the document's own schema/content revision number, independent of Section 9's platform release policy. A Class B document can say `version: "1.0.0"` while the platform itself is still pre-launch V1 — these are two different counters. Class A documents do not carry a `version` field; they carry `Status: V1` in the header block instead.

Both classes share Sections 7 (Writing Style), 10 (License footer), and 11 (Cross-referencing).

### 6.3 Table of Contents

Required for all Class A documents with more than 5 sections:

```markdown
## Table of Contents

1. [Section Title](#1-section-title)
2. [Section Title](#2-section-title)
```

### 6.4 Section Numbering

- Top-level sections: `## 1. Title`, `## 2. Title`
- Subsections: `### 1.1 Subtitle`, `### 1.2 Subtitle`
- Sub-subsections: `#### 1.1.1 Detail` (use sparingly)
- Never skip a level

### 6.5 Tables

Use tables whenever a list has 3 or more items and each item carries 2 or more attributes worth comparing. If the content is a single flat list with no attributes to compare, use a list instead of a table.

```markdown
| Column A | Column B | Column C |
|---|---|---|
| Value | Value | Value |
```

### 6.6 Code Blocks

All code, commands, configuration, and identifiers use fenced code blocks with a language specifier. Never show a command inline without a code block if it needs to be executed.

````
```bash
sudo apt update
```

```yaml
version: "3.8"
services:
```

```go
func main() {
```
````

---

## 7. Writing Style Rules

### 7.1 Tone

| Rule | What it means |
|---|---|
| Semi-formal to direct | No corporate fluff. No apologies. No padding. |
| Knowledgeable, not condescending | Assume the reader is competent in their domain. |
| Specific, not vague | Every claim must be actionable or verifiable. |

### 7.2 Banned and Discouraged Language

**Banned outright** (never appears in any ZarishSphere document): "genuinely," "honestly," "straightforward."

**Discouraged marketing language** (flag on review, rewrite unless quoting a source): "seamless / seamlessly," "cutting-edge," "powerful," "robust," "revolutionary," "game-changing," "best-in-class," "world-class," "state-of-the-art." These words describe a feeling, not a fact. Replace with the specific, verifiable claim instead — e.g., not "a powerful FHIR engine" but "a FHIR R5 engine that runs in under 150MB of RAM on a Raspberry Pi 5."

### 7.3 Plain Language First

Technical concepts always get a plain-language framing before the technical detail.

Pattern:
```
[What it is in one plain sentence.]
[Technical detail follows.]
```

Example:
> **Infrastructure as a Service (IaaS)** — You rent the raw foundation; you still build what goes on top.
> *Technical:* On-demand provisioning of virtual machines, storage, and networking. The customer manages OS, runtime, middleware, and applications.

Apply this pattern to all ZarishSphere service descriptions and deployment plane documentation.

### 7.4 Service Model Language (XaaS Mapping)

ZarishSphere uses the XaaS mental model to communicate its deployment options:

| ZarishSphere Tier | XaaS Equivalent | What the deployer owns |
|---|---|---|
| Plane 0 — Air-Gapped | On-Premises | Everything |
| Plane 1 — Edge (Raspberry Pi) | IaaS + self-managed | OS + apps + data |
| Plane 2 — District Server | PaaS-like | Apps + data |
| Plane 3 — National Cloud | SaaS + config | Config + data |
| Plane 4 — Global SaaS | Full SaaS | Data only |

### 7.5 Heading Rules

- Use `##` for primary sections, `###` for subsections in the document body.
- Never use `#` for anything other than the document title line.
- Body headings (`##`/`###` within numbered sections) should not exceed 8 words. This limit does not apply to the three-line title block (title / human-readable title / subtitle), which may run longer to carry a full project tagline.
- All body headings: sentence case (first word capitalised, rest lowercase unless proper nouns).

### 7.6 Lists

- Use bullet lists (`-`) only for unordered items with no natural sequence.
- Use numbered lists (`1.`, `2.`) for procedures, steps, or prioritised items.
- Maximum 7 items in a bullet list before converting to a table.
- No single-item lists. If there is only one item, it is prose.

---

## 8. Technical Documentation Subtypes

### 8.1 ADR (Architecture Decision Record)

File: `nnn-adr-[short-title].md` — Class A.

Required sections:
1. Decision
2. Context
3. Alternatives Considered
4. Reason for Decision
5. Consequences
6. Status (Accepted / Superseded / Proposed)

### 8.2 SOP (Standard Operating Procedure)

File: `nnn-sop-[process-name].md` — Class A.

Required sections:
1. Purpose
2. Scope
3. Roles Responsible
4. Preconditions
5. Step-by-Step Procedure (numbered, GUI-first)
6. Expected Outcome
7. Escalation Path

### 8.3 PRD (Product Requirements Document)

File: `nnn-prd-[feature-name].md` — Class A.

Required sections:
1. Problem Statement
2. Goals & Non-Goals
3. User Stories
4. Functional Requirements
5. Non-Functional Requirements
6. Standards Compliance
7. Acceptance Criteria

### 8.4 Standards-Index Entry

File: `[zarish_id].md` or catalog row — Class B. See `003-zs-index-metadata-schema.md` for the full 22-field schema. This is the one document subtype that uses YAML frontmatter as its primary and only metadata layer — no separate header block.

### 8.5 README.md

Every repository must have a `README.md` (tooling-reserved, Section 3) with:
1. One-line description
2. Status badge
3. Quick start (GUI-first, 5 steps maximum)
4. Architecture overview link
5. License block

---

## 9. Version Policy

**V1 until launch.** No *platform release* version numbers are incremented during development. The platform itself is V1 from first document to launch. This is separate from the Class B `version` frontmatter field (Section 6.2), which tracks individual document/schema revisions and may legitimately read `1.0.0`, `1.1.0`, etc. even pre-launch.

| Stage | Platform Version Label | Document Schema Version |
|---|---|---|
| All development | V1 | Increments per document, e.g. `1.0.0` → `1.1.0` |
| First production launch | v1.0.0 | Unaffected — continues its own track |
| Post-launch patches | v1.0.1, v1.0.2… | Unaffected |
| Feature releases | v1.1.0, v1.2.0… | Unaffected |

---

## 10. License Block (Mandatory Footer)

Every Class A document must end with:

```
---

*ZarishSphere Foundation · V1 · [Date]*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
```

Class B documents carry equivalent information in frontmatter (`last_updated`, license implied by repository-level LICENSE file) and do not require the prose footer, though it is not forbidden.

---

## 11. Cross-Reference Standard

When referencing another ZarishSphere document, use the format:

```markdown
→ **[document-name].md** — [one-line description of what it contains]
```

Example:
```markdown
→ **004-zs-index/004-harvesting-policy.md** — Strategic direction for the ZarishIndex autonomous research project
→ **003-zs-platform/001-platform-overview.md** — Platform architecture, roadmap, and deployment model
```

When referencing a document in a different repository, include the repository name:

```markdown
→ **zs-index/docs/001-zs-index-project-charter.md** — (cross-project: see `zarishsphere/zs-index` repo) — ZarishIndex mission, scope, and vision
```

---

## 12. Validation and Enforcement

ZUSS is enforced the same way ZarishIndex enforces its own schema (`003-metadata-schema.md`, Section 8) — rules only count if something checks them.

### 12.1 Automated checks (`scripts/validate-zuss.py`, run in CI on every PR)

| Rule ID | Check |
|---|---|
| Z01 | Filename matches `^[0-9]{3}-[a-z0-9-]+\.[a-z]+$`, OR is listed in the Section 3 exemption table |
| Z02 | No filename contains a doubled extension (`.md.md`, `.yml.yml`, etc.) |
| Z03 | No uppercase characters in filename, except exempted names |
| Z04 | No underscore or space in filename |
| Z05 | Every Class A document opens with the full header block (Section 6.1) |
| Z06 | Every Class B document opens with valid YAML frontmatter containing all required fields |
| Z07 | No document mixes header-block and frontmatter schemas |
| Z08 | Every document ends with the correct footer for its class (Section 10) |
| Z09 | No banned word (Section 7.2, "banned outright" list) appears anywhere in the document |
| Z10 | Every new ID prefix is checked against the Section 5 registry before merge |
| Z11 | Every workflow file matches the `[id]--[trigger]--[process].yml` pattern (Section 2.3) |
| Z12 | Docker image references are never tagged `latest` |

Validation failures block merge. Discouraged-language hits (Section 7.2, second list) are logged as warnings, not blockers — a human call, not a machine one.

### 12.2 Human review

| Content type | Review required |
|---|---|
| New ID namespace | Curator approval against Section 5 registry |
| New tooling-reserved filename exemption | Curator approval, added to Section 3 |
| Class A/B classification for a new document type | Curator approval before first use |

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
