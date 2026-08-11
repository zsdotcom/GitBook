# 007-zarishsphere-agent-ecosystem-strategy.md
## ZarishSphere agent-native documentation strategy
### Making every document a machine-consumable asset — V1

**Document type:** Strategic blueprint
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [What this document is](#1-what-this-document-is)
2. [The big picture: docs as everything](#2-the-big-picture-docs-as-everything)
3. [Current state assessment](#3-current-state-assessment)
4. [The agent-native layer: what makes docs AI-ready](#4-the-agent-native-layer-what-makes-docs-ai-ready)
5. [Machine-readable metadata: frontmatter and agent manifest](#5-machine-readable-metadata-frontmatter-and-agent-manifest)
6. [Knowledge graph: connecting every document](#6-knowledge-graph-connecting-every-document)
7. [Agent entry points: AGENTS.md and llms.txt](#7-agent-entry-points-agentsmd-and-llmstxt)
8. [Skills packaging: every doc folder as a skill](#8-skills-packaging-every-doc-folder-as-a-skill)
9. [MCP layer: exposing docs as AI resources](#9-mcp-layer-exposing-docs-as-ai-resources)
10. [Lifecycle management: keeping docs alive](#10-lifecycle-management-keeping-docs-alive)
11. [Practical roadmap: what to do in what order](#11-practical-roadmap-what-to-do-in-what-order)

---

## 1. What this document is

This document is a blueprint. It describes how to evolve `zs-docs` from a documentation repository into the **operating system for the entire ZarishSphere ecosystem** — where every document is simultaneously:

- A **law** (constitution, governance)
- A **blueprint** (architecture, design)
- A **template** (SOPs, workflows)
- A **skill** (loadable by AI agents)
- A **knowledge graph node** (connected to every other document)
- An **MCP resource** (consumable by any AI tool)

And where understanding all of this requires **zero coding skill** — only the ability to write plain markdown.

## 2. The big picture: docs as everything

The world of AI is moving in a direction that ZarishSphere already anticipated. In 2026, the industry has converged on a single truth:

> **Markdown is the universal interface between humans and AI agents.**

What this means for you:

- Every major AI tool (Claude, ChatGPT, VS Code Copilot, Cursor, GitHub Copilot) now reads **markdown files** natively
- The industry standard for AI agent configuration is **markdown files with YAML front matter** — ZUSS specifies frontmatter for Class B structured entries, while Class A documents carry the ZUSS header block plus a machine-readable agent manifest (Section 5)
- The Model Context Protocol (MCP) — the USB-C standard for AI connections — uses **markdown resources** as its primary data format
- Agent skills are packaged as **folders of markdown files** (SKILL.md + supporting docs)

Your documentation is not "just documentation." It is the **source code that AI agents will read, interpret, and act upon.** Every law, every architecture decision, every SOP you write is a set of instructions that future AI agents will execute.

## 3. Current state assessment

### What is already strong

| Strength | Why it matters for AI |
|---|---|
| ZUSS naming standard (nnn-prefix, lowercase, hyphens) | Agents can navigate predictably |
| YAML frontmatter on Class B entries (ZUSS §6.2) | Agents can filter structured entries by maturity |
| ZUSS header block on Class A documents (no frontmatter, per ZUSS Z07) | Agents classify documents without schema mixing |
| Header block with document type and date | Agents can judge authority and freshness |
| Numbered sections with stable anchors | Agents can deep-link to specific sections |
| Cross-reference format (`→ **file.md**`) | Agents can follow links like a graph |
| Constraint blocks (> **Constraint:**) | Agents recognize hard rules vs guidance |
| AGENTS.md already exists | Agents have an entry point |
| 12-law constitution | Supreme context for all decisions |
| Ecosystem architecture (005) | Master map for navigation |

### What needs to be added

| Gap | What it means for AI | Status |
|---|---|---|
| No `summary` field in Class B frontmatter / `.agents` manifest | Agent must read the whole file to know what it's about | ✅ Resolved — summaries live in Class B frontmatter, `index.md` files, and the `.agents` manifest |
| No `tags` field in Class B frontmatter | Agent cannot categorize or filter | ✅ Resolved — tags in Class B frontmatter and the `.agents` manifest |
| No `related` cross-links in machine-readable form | Agent doesn't know which docs to read next | ⬜ Pending — Phase 2 (`.agents` manifest edges for Class A, frontmatter `related` for Class B) |
| No `last_verified` field | Agent doesn't know if info is still accurate | ✅ Resolved — added to stable Class B entries and the `.agents` manifest |
| No `entity-type` or `doc-type` classification | Agent cannot distinguish laws from SOPs from ADRs | ✅ Resolved — present in Class B frontmatter and the `.agents` manifest |
| No llms.txt file | Agent has no quick index of what exists | ✅ Resolved — `llms.txt` created at root |
| No knowledge graph edges between documents | Agent sees isolated files, not a connected system | ⬜ Pending — Phase 2 |
| No index.md navigation files | Agent has no folder-level index | ✅ Resolved — 11 index.md files created |
| No validation scripts | No automated compliance checking | ✅ Resolved — 5 scripts in `scripts/` |
| No skills packaging | Agent cannot load a folder as a "capability" | ✅ Resolved — 10 skills in `.opencode/skills/` |
| No MCP server | Agent cannot query docs as a live resource | ✅ Resolved — MCP server at `.opencode/mcp-server-github.js` |
| No freshness/verification cadence | Docs get stale silently | ⬜ Pending — Phase 5 |
| Graph edges not derived from the ZUSS arrow format | Cross-references are prose, not structured relationships | ⬜ Pending — Phase 2 (derive edges from `→ **file.md**` refs and the `.agents` manifest) |

## 4. The agent-native layer: what makes docs AI-ready

An "agent-native" document is one that an AI agent can read, understand, and act upon **without a human explaining it.** This is different from "human-readable" documentation.

### The four-layer model

```
Layer 1: Discovery ─── Agent finds out the document exists
    │                    (llms.txt, AGENTS.md, folder structure)
    ▼
Layer 2: Routing ────── Agent decides if this doc is relevant
    │                    (Class B frontmatter · .agents manifest: summary, tags, status)
    ▼
Layer 3: Reading ────── Agent extracts the rules and facts
    │                    (numbered sections, constraint blocks, stable anchors)
    ▼
Layer 4: Acting ─────── Agent follows instructions or references
                         (cross-links, relationships, skills format)
```

### The "curl test"

A simple way to know if your docs are agent-ready: if an AI agent fetched the raw markdown of any document, could it do its job? Specifically:

1. Can it know what this document is about within 3 seconds? (summary in Class B frontmatter or the `.agents` manifest)
2. Can it know whether this document is current? (last_updated, status)
3. Can it know which other documents to read for context? (related, dependencies)
4. Can it know what type of document this is? (doc-type: law/architecture/sop/adr)
5. Can it follow a clear instruction without ambiguity? (constraint blocks, numbered steps)

## 5. Machine-readable metadata: frontmatter and agent manifest

ZUSS defines two document classes, and agent-facing metadata must follow that split (ZUSS §6, rule Z07):

- **Class A documents** (governance, architecture, ADRs, SOPs, direction papers) carry the **ZUSS header block only**. No YAML frontmatter is allowed in the file. Their agent-facing metadata — summary, tags, status, relationships — lives in a **machine-readable agent manifest** (`.agents/`) and in per-folder `index.md` navigation files.
- **Class B documents** (standards-index entries, catalog rows) carry the **enhanced YAML frontmatter** below. This is their primary and only metadata layer.

### Class B frontmatter (what AI agents need)

```yaml
---
id: "ZI-HEALTH-00001"
title: "Indexed standard entry"
domain: "004-zs-index"
doc-type: "standards-index-entry"      # classification: constitution/law/architecture/sop/adr/specification/blueprint/template
summary: >-                            # one-sentence routing hint for agents
  Machine-readable record for the indexed standard, carrying
  metadata for the G2A pipeline.
tags:                                 # flat list for categorization and filtering
  - health
  - guideline
entity-type: "standards-index-entry"  # what kind of node in the knowledge graph
version: "1.0.0"
status: "stable"                      # draft / stable / deprecated
last_updated: 2026-08-01
last_verified: 2026-08-01             # when a human confirmed it's still accurate
verified_by: "ZarishSphere Foundation" # who confirmed
next_review: 2026-11-01               # suggested next review date (90 days)
isolation_tier: "global"
canonical: true                       # true = this is a defended source of truth
depends_on:                           # documents that must be read first
  - "ZS-001-ZAR"
related:                              # documents that are closely connected
  - "ZS-004-WRI"
supersedes: ~                         # what this document replaces (if anything)
replaced_by: ~                        # what replaces this (when deprecated)
capabilities:
  - agent-skill: "parse_standard_entry"
  - mcp-resource: "standards-entry"
audience:                             # who should read this
  - "ai-agents"
---
```

### Agent manifest for Class A documents

Class A documents never carry frontmatter. Instead, one manifest file per folder records the agent-facing metadata for every Class A document in it:

```yaml
# .agents/001-zs-meta.yml
folder: "001-zs-meta"
documents:
  - file: "001-zarishsphere-constitution.md"
    doc-type: "constitution"
    summary: "Supreme governing document: 12 laws across 4 tiers."
    status: "stable"
    last_verified: "2026-08-01"
    depends_on: []
    related:
      - "004-zarishsphere-writing-rules.md"
      - "005-zarishsphere-ecosystem-architecture.md"
    capabilities:
      - agent-skill: "parse_001_zarishsphere_constitution"
      - mcp-resource: "constitution"
```

The manifest is a build-time input: agents, the MCP layer, and validation scripts all read it instead of parsing Class A files for frontmatter that ZUSS forbids.

### Field-by-field explanation (plain language)

| Field | What it tells the AI agent |
|---|---|
| `doc-type` | What kind of document this is — so the agent knows how to interpret it |
| `summary` | One sentence that lets the agent decide "do I need to read this?" without reading the whole thing |
| `tags` | Labels for searching and filtering |
| `entity-type` | What this document represents in the knowledge graph |
| `last_verified` | When a human last confirmed the info is still correct (different from when it was edited) |
| `verified_by` | Who confirmed it — so the agent can weigh authority |
| `next_review` | When this doc should be checked again — creates a maintenance schedule |
| `canonical` | Is this a "blessed" source of truth? Agents treat canonical:true docs as authoritative |
| `depends_on` | What must be read before this — agents build their context in the right order |
| `related` | What is closely connected — agents build a complete picture |
| `supersedes/replaced_by` | Version chain — agents follow the lineage |
| `audience` | Who this is for — agents filter to what's relevant |

## 6. Knowledge graph: connecting every document

AI agents work best when documents form a connected graph, not isolated files.

### What a knowledge graph is (plain language)

Think of every document as a node, and every cross-reference as a directed connection between nodes. When an agent reads document A and it says "see document B," the agent follows that link. When enough links exist, the agent can navigate the entire ecosystem by following connections — just like a human browsing Wikipedia.

### Relationship types

When you link one document to another, specify **what kind** of relationship it is:

| Relationship | Meaning | Example |
|---|---|---|
| `depends_on` | Must read this first | Constitution depends on README |
| `related` | Closely connected, read alongside | Writing rules related to Glossary |
| `supersedes` | This document replaces another | ADR 002 supersedes ADR 001 |
| `implements` | This document implements something | Platform overview implements the Constitution's Law 7 |
| `references` | Mentions but doesn't depend on | Tech stack references Cloudflare architecture |
| `part_of` | This document is a section of a larger whole | SOP-001 is part of Operations |
| `conflicts_with` | These documents disagree (note it explicitly) | (rare, use with caution) |

### How to express relationships

**In machine-readable metadata** (Class B frontmatter or the `.agents` manifest):
```yaml
depends_on:
  - "ZS-001-ZAR"
  - "ZS-005-ECO"
related:
  - "ZS-004-WRI"
```

**In body text** (for both human and machine reading), using the ZUSS cross-reference format:
```markdown
→ **001-zarishsphere-constitution.md** — Supreme governing document: 12 laws across 4 tiers
→ **004-zarishsphere-writing-rules.md** — ZUSS: naming, formatting, and documentation rules
```

**Using the ZUSS arrow format** (required — no wikilinks):
```markdown
This architecture follows → **001-zarishsphere-constitution.md** and uses the naming rules from → **004-zarishsphere-writing-rules.md**.
```

### The graph shapes that emerge

```
                    ┌── 001-zarishsphere-constitution.md
                    │
    README.md ──────┼── 004-zarishsphere-writing-rules.md (ZUSS) ──── 006-zarishsphere-glossary.md
                    │
                    ├── 005-zarishsphere-ecosystem-architecture.md ─── 009-zs-operations/*.md
                    │
                    ├── 007-zarishsphere-agent-ecosystem-strategy.md (this doc)
                    │
                    └── TODO.md
```

## 7. Agent entry points: AGENTS.md and llms.txt

AI agents need a front door — a short file they read first that tells them what exists and where to start.

### AGENTS.md (already created)

This is the file created at the repo root. It serves as the **bootstrap loader** for any AI agent entering the repository for the first time. It should be kept short (under 100 lines) and point to the most critical docs.

**What it should contain:**
1. What this repo is (one sentence)
2. First-read order (4-5 critical docs)
3. Non-negotiable rules (ZUSS naming, document structure)
4. Current state and key constraints
5. Validation checklist before merging

### llms.txt (created)

An `llms.txt` file at the root of the repository gives AI agents a quick index of all available content. It is a simple markdown file with links organized by category.

**What it should contain:**
```markdown
# zs-docs
> ZarishSphere Foundation — master documentation index

## Essentials
- [001-zarishsphere-constitution.md](001-zarishsphere-constitution.md): Supreme governing document

- [004-zarishsphere-writing-rules.md](004-zarishsphere-writing-rules.md): ZUSS naming and formatting standard

- [005-zarishsphere-ecosystem-architecture.md](005-zarishsphere-ecosystem-architecture.md): Master repository map

- [006-zarishsphere-glossary.md](006-zarishsphere-glossary.md): All terms defined

## Foundation governance (002-zs-foundation/)
- [001-foundation-charter.md](../002-zs-foundation/001-foundation-charter.md): Mission and scope
... (continue for all documents)
```

### How agents use these files

```
Agent enters repo
       │
       ▼
  Reads AGENTS.md ───────────► Gets: what this repo is, first-read order,
       │                          non-negotiable rules, current state
       ▼
  Reads llms.txt ────────────► Gets: complete index of all documents
       │
       ▼
  Follows first-read order ──► Constitution → ZUSS → Architecture → Glossary
       │
       ▼
  Proceeds to task
```

## 8. Skills packaging: every doc folder as a skill

This is the most effective pattern for making your documentation directly loadable by AI agents.

### What a skill is (plain language)

A skill is a folder of documents that teaches an AI agent how to do something. For example, a "Foundation Governance" skill would contain all the documents about how the foundation operates, its charter, its rules, its licensing policies.

When an AI agent loads a skill, it reads the folder and understands: "I now know how to handle foundation governance tasks."

### The industry standard skill format

```
002-zs-foundation/                ◄── skill folder (name = folder name)
├── SKILL.md                       ◄── required: the main skill instruction file
├── 001-foundation-charter.md      ◄── supporting documents
├── 002-governance-model.md
├── 003-licensing-policy.md
└── 004-contributor-guidelines.md
```

Every folder in `zs-docs` is already structured as a potential skill:

| Folder | Skill name | What the skill teaches |
|---|---|---|
| `001-zs-meta/` | `meta` | Foundation rules, ZUSS, architecture, glossary |
| `002-zs-foundation/` | `foundation` | Foundation governance, charter, licensing |
| `003-zs-platform/` | `platform` | Platform architecture, modules, API design |
| `004-zs-index/` | `zarish-index` | ZARISH-INDEX data engine |
| `005-zs-standards/` | `zarish-standards` | ZARISH-STANDARDS transformation layer |
| `006-zs-infrastructure/` | `infrastructure` | GitHub, Cloudflare, domain architecture |
| `007-zs-tech-stack/` | `tech-stack` | Technology choices and decisions |
| `008-zs-adrs/` | `adrs` | Architecture Decision Records |
| `009-zs-operations/` | `operations` | SOPs, workflows, runbooks |
| `010-zs-ecosystem/` | `ecosystem` | Ecosystem components: Console, Marketplace, Builder, Apps, Forms, SDK, CLI, API, Services, Modules, Distributions, Engine, System, Content repos, Home, FHIR Hub |

### SKILL.md format

Each folder gets a `SKILL.md` that describes what the skill does, when to load it, and what documents it contains. Example:

```yaml
---
name: "foundation"
title: "Foundation governance"
summary: "Governance documents for the ZarishSphere Foundation — charter, decision-making, licensing, contributor rules"
version: "1.0.0"
status: "stable"
tags: [governance, foundation, charter, licensing]
depends_on:
  - "meta"
documents:
  - "001-foundation-charter.md"
  - "002-governance-model.md"
  - "003-licensing-policy.md"
  - "004-contributor-guidelines.md"
---
# Foundation governance skill

Load this skill when the task involves:

- Understanding how the foundation is governed
- How to make decisions within the foundation
- Licensing rules for ZarishSphere projects
- How contributors can participate

## Dependencies

This skill depends on the `meta` skill (constitution, ZUSS, architecture, glossary).
Load the `meta` skill first if it is not already in context.
```

## 9. MCP layer: exposing docs as AI resources

MCP (Model Context Protocol) is the industry standard for connecting AI agents to external data. It is like a USB-C port for AI — a universal way for any AI tool to read your documentation as a live resource.

### What MCP gives you

1. **Any AI tool can read your docs** — Claude, ChatGPT, VS Code Copilot, Cursor, and all MCP-compatible clients
2. **Live access** — agents read the current version, not a cached copy
3. **Structured access** — docs are organized as named resources with metadata
4. **Skill discovery** — agents can list available skills and load them on demand

### How to build the MCP layer

This is a **future step** (Phase 4 in the roadmap). You do not need to do this now. But the architecture should be designed so it is possible.

The MCP server would:
1. Read all markdown files from `zs-docs`
2. Parse Class B frontmatter and the `.agents` manifest to extract metadata; derive metadata from Class A header blocks
3. Expose each document as an MCP resource (e.g., `docs://001-zs-meta/001-zarishsphere-constitution`)
4. Expose each skill (folder) as an MCP skill (e.g., `skill://foundation`)
5. Expose a search/discovery endpoint

### What you need to do now to prepare

Nothing technical. Just make sure every document has:
- A unique `id` for Class B entries in frontmatter, and an `.agents` manifest entry for Class A documents (already done ✓)
- Clear section headings with stable anchors ✓
- Complete cross-references ✓
- Machine-readable metadata ✓

This is enough for any MCP server to consume your docs.

## 10. Lifecycle management: keeping docs alive

Documentation that is not maintained becomes misinformation. AI agents are worse than humans at detecting stale information — they will confidently cite a 2-year-old draft as truth.

### The three states every document goes through

```
     ┌──────────┐     ┌──────────┐     ┌──────────────┐
     │  DRAFT   │ ──► │  STABLE  │ ──► │  DEPRECATED  │
     │ (write)  │     │(maintain)│     │ (archive)    │
     └──────────┘     └──────────┘     └──────────────┘
```

- **DRAFT**: Being written, not yet authoritative. Agents should not cite drafts as truth.
- **STABLE**: Reviewed and approved. Agents can quote stable docs verbatim.
- **DEPRECATED**: No longer current. Agents should not use it; the document should point to its replacement.

### The verification cycle

Every document needs to be **verified** periodically — not just edited, but confirmed as still accurate.

| Document type | Suggested verification cadence |
|---|---|
| Constitution, laws | Every 6 months |
| Architecture docs | Every 3 months |
| ADRs | At project milestones |
| SOPs, runbooks | Every 3 months or after any process change |
| Tech stack docs | Monthly during active development |

### How to track this

The `last_verified` and `next_review` fields — in Class B frontmatter and the `.agents` manifest for Class A — create a simple tracking system. You can periodically run a review by checking which documents have `next_review` dates that have passed.

## 11. Practical roadmap: what to do in what order

This roadmap is designed for a single person with no coding experience. Every step uses plain markdown editing — no programming required.

### Phase 0: Foundation (complete)

- [x] Update this document with any changes from discussion
- [x] Scope YAML frontmatter to Class B entries (ZUSS §6.2); add the `.agents` manifest for Class A documents
- [x] Create `llms.txt` at the root with a complete index of all documents
- [x] Update `AGENTS.md` to reference this strategy document
- [x] Create index.md navigation files for all 10 folders and root
- [x] Create validation scripts (naming, status, cross-refs)

### Phase 1: Complete all primary documents

- [x] Write all 8 documents in `001-zs-meta/`
- [x] Write all 5 documents in `002-zs-foundation/`
- [x] Write all 10 documents in `003-zs-platform/`
- [x] Write all 5 documents in `004-zs-index/`
- [x] Write all 17 documents in `005-zs-standards/`
- [x] Write all 10 documents in `006-zs-infrastructure/`
- [x] Write all 6 documents in `007-zs-tech-stack/`
- [x] Write all 23 documents in `008-zs-adrs/`
- [x] Write all 15 documents in `009-zs-operations/`
- [x] Write all 19 documents in `010-zs-ecosystem/` (Console, Marketplace, Builder, Apps, Forms, SDK, CLI, API, Services, Modules, Distributions, Engine, System, Content Forms, Content Protocols, Content Templates, Content Reports, Home, FHIR Hub)

### Phase 2: Knowledge graph enrichment

As you write each document, add:

- [ ] Machine-readable metadata: `depends_on`, `related`, `supersedes` in Class B frontmatter and the `.agents` manifest
- [ ] Body: ZUSS cross-reference format (`→ **file.md** — description`)
- [ ] End of document: `## Related` section listing all connected docs

### Phase 3: Skills packaging

After each folder has all its documents:

- [ ] Create `SKILL.md` in each of the 10 folders (001-zs-meta through 010-zs-ecosystem)
- [ ] Verify each SKILL.md has: name, title, summary, version, status, tags, document list
- [ ] Test: ask an AI agent to load a skill and confirm it works

### Phase 4: MCP server (future)

When the content is complete and verified:

- [ ] Set up a simple MCP server (can be done with minimal technical help)
- [ ] Expose all documents as MCP resources
- [ ] Expose all skills via the MCP skills extension
- [ ] Deploy on Cloudflare (free tier)

### Phase 5: Automation and maintenance

- [ ] Set up a 90-day review reminder for all documents
- [ ] Create a review template for verifying document accuracy
- [ ] Run a cross-reference validation script before any commit
- [ ] Set up GitHub repository and push all documents

---

## Summary: what success looks like

When this strategy is fully implemented, this is what happens:

1. A new AI agent enters `zs-docs`
2. It reads AGENTS.md → learns the repo's purpose and rules
3. It reads index.md → gets navigation routes for all 10 folders
4. It reads llms.txt → sees all 128 files across 10 domains — 118 numbered documents — organized by category
5. It reads the Constitution → learns the 12 laws
6. It reads ZUSS → learns the naming and formatting rules
7. It loads a skill (e.g., `foundation`) → gets all governance documents
8. It follows cross-references → navigates the entire knowledge graph
9. It checks last_verified → knows the information is current
10. It produces output that follows all ZarishSphere rules automatically

No coding. No programming. Just well-structured markdown documents connected by clear relationships.

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
