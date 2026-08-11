# 005-zarish-index-to-platform.md
## ZARISH-INDEX — ZarishSphere Integration Architecture
### How the Standards Index Feeds the G2A Engine

**Document type:** Integration Reference — Authoritative
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable. Authoritative integration specification.

---

## Table of Contents

1. [Overview — two projects, one pipeline](#1-overview--two-projects-one-pipeline)
2. [The relationship](#2-the-relationship)
3. [Data flow in detail](#3-data-flow-in-detail)
4. [How ZarishSphere consumes ZARISH-INDEX](#4-how-zarishsphere-consumes-zarish-index)
5. [Autonomy boundaries](#5-autonomy-boundaries)
6. [Version and sync strategy](#6-version-and-sync-strategy)
7. [G2A engine stages that use ZARISH-INDEX](#7-g2a-engine-stages-that-use-zarish-index)
8. [Cross-references](#8-cross-references)

---

## 1. Overview — two projects, one pipeline

Two projects. One pipeline.

**ZARISH-INDEX** is an autonomous, open research project that maintains the world's most complete machine-readable index of all global standards across 40 domains. It is its own project under CC BY 4.0 and serves any researcher or organisation globally.

**ZarishSphere** is a Digital Public Infrastructure platform that converts international standards into deployable digital workflows. Its core engine — the G2A (Guideline-to-Action) Engine — needs to know what standards exist, what they mean, and how they relate to each other before it can generate anything.

ZARISH-INDEX solves this problem for ZarishSphere. It is the upstream knowledge base.

Entry counts cited in this document (88,204+ standards, 20,140 relationship edges) are measured harvest counts from the last ZARISH-INDEX harvest.

---

## 2. The relationship

```
ZARISH-INDEX (autonomous upstream project)
│
│  Provides:
│  ├── Unique identifiers for 88,204+ global standards (zarish_id)
│  ├── Domain taxonomy (40 domains, 6 meta-layers)
│  ├── Standards body metadata (600+ SDOs)
│  ├── Entry type labels: Standard / Framework / Treaty / Guideline / Regulation
│  ├── Status flags: Active / Withdrawn / Superseded / Under Development
│  ├── 20,140 relationship edges (supersedes, supplements, implements, national_adoption_of)
│  ├── Mandate type: Mandatory / Voluntary / Treaty-binding
│  └── Source URLs and authoritative primary references
│
▼
G2A Engine — Stage 1 (INGEST) + Stage 2 (PARSE)
│
│  Consumes ZARISH-INDEX to:
│  ├── Identify which standard a document belongs to
│  ├── Validate standard citations in uploaded guidelines
│  ├── Resolve standard identifiers to canonical metadata
│  ├── Classify G2A outputs by domain (health, WASH, logistics, etc.)
│  ├── Skip deprecated standards in form and rule generation
│  └── Build cross-domain linkage in generated content
│
▼
G2A Engine — Stage 3 (EXTRACT) + Stage 6 (DEPLOY)
│
│  Outputs enriched with ZARISH-INDEX metadata:
│  ├── FHIR Questionnaires with correct standard citations
│  ├── PlanDefinition rules linked to authoritative standard references
│  ├── DHIS2 datasets with correct governance body attribution
│  └── Training materials with verified source references
│
▼
ZarishSphere Deployment (Planes 0–4)
│
└── Working digital workflows for field health workers
    tagged with authoritative, verifiable standard metadata
```

---

## 3. Data flow in detail

### 3.1 ZARISH-INDEX fields consumed by ZarishSphere

| ZARISH-INDEX field | Used by G2A stage | Purpose |
|---|---|---|
| `zarish_id` | Stage 1 — INGEST | Match uploaded document to known standard |
| `name_full` + `name_short` | Stage 2 — PARSE | AI uses to identify standard references in text |
| `domain` + `sub_domain` | Stage 3 — EXTRACT | Classify generated assets by sector |
| `entry_type` | Stage 3 — EXTRACT | Determine output type: form vs API spec vs governance rule |
| `status` | Stage 4 — VALIDATE | Block generation from deprecated or withdrawn standards |
| `issuer` | Stage 3 — EXTRACT | Attribution in generated SOPs and training materials |
| `official_url` + `standard_id` | Stage 5 — STAGE | Reference links in generated content and GitHub PRs |
| `mandate` | Stage 3 — EXTRACT | Tag generated outputs as mandatory vs voluntary |
| `why_it_matters` | Stage 3 — EXTRACT | Human-readable explanation embedded in generated materials |
| `supersedes` relationship edge | Stage 3 — EXTRACT | Use most current standard; note predecessor for FHIR R4 bridge |
| `supplements` relationship edge | Stage 3 — EXTRACT | Cross-reference related standards in generated content |
| `national_adoption_of` edge | Stage 6 — DEPLOY | Verify national regulatory equivalence (e.g., BSTI vs ISO) |

### 3.2 Fields not currently consumed

The following ZARISH-INDEX fields are not yet consumed by ZarishSphere but are retained in the index for future phases:

| Field | Future use |
|---|---|
| `year_first` | Historical timeline; audit trail for standard evolution |
| `sector_applicability` | Targeted deployment recommendation per programme type |
| `geographic_scope` | Multi-country deployment routing in Phase 3 ZarishSphere |
| `key_outputs` | Summary panel in ZarishSphere Standards Browser (Phase 4) |
| `ratification_tracker` | Country-level compliance profiling (Phase 5 ZarishSphere) |

---

## 4. How ZarishSphere consumes ZARISH-INDEX

ZARISH-INDEX is published on GitHub under CC BY 4.0. ZarishSphere consumes it in three methods, operating in parallel:

| Method | Frequency | Use case |
|---|---|---|
| GitHub Actions download | On every ZARISH-INDEX tagged release | Sync master CSV/JSON/Parquet to `zs-module-zarish-index` |
| Local PostgreSQL table | Persistent after download | Runtime queries during G2A processing |
| Qdrant vector index | Rebuilt on each major release | Semantic search: "find standards about vaccination in emergencies" |

**Consuming repository:** `zarishsphere/zs-module-zarish-index`

**Sync command (in `zs-module-zarish-index` Makefile):**

```bash
sync-zarish-index:
	curl -L https://github.com/zarishsphere/zarish-index/releases/current/download/zarish_master.csv \
	  -o data/zarish_master.csv
	python3 scripts/import_to_postgres.py data/zarish_master.csv
	python3 scripts/rebuild_vector_index.py
```

---

## 5. Autonomy boundaries

ZARISH-INDEX and ZarishSphere are **independent projects** that share data. This independence is intentional and must be preserved.

| Rule | Reason |
|---|---|
| ZARISH-INDEX changes do not require ZarishSphere approval | ZARISH-INDEX serves 40+ domains globally, not just ZarishSphere needs |
| ZarishSphere changes do not affect ZARISH-INDEX content | ZarishSphere is a consumer, not a contributor, of standards metadata |
| ZARISH-INDEX uses CC BY 4.0 | ZarishSphere uses Apache 2.0 for code — no license conflict |
| ZARISH-INDEX can be used by any project or organisation | It is not ZarishSphere property |
| ZarishSphere can use any additional data sources | ZARISH-INDEX is preferred but not exclusive |
| Breaking schema changes in ZARISH-INDEX require migration ADR | Schema stability protects all consumers, not just ZarishSphere |

---

## 6. Version and sync strategy

| ZARISH-INDEX event | ZarishSphere action |
|---|---|
| New domain added | Automatically available via next data sync |
| Standard status changed (Active → Deprecated) | G2A validation stage blocks generation from that standard at next sync |
| New relationship edges added | Cross-domain linking in G2A improves automatically |
| Major schema change (new fields added) | `zs-module-zarish-index` migration PR required; documented in ADR |
| Breaking schema change (field renamed or removed) | Semantic versioning bump in ZARISH-INDEX release; explicit migration plan |

ZarishSphere pins to a specific ZARISH-INDEX release tag in the consuming module's workflow file. Upgrades are explicit, reviewed, and logged in an ADR.

**Example pin in `.github/workflows/101--on-schedule--sync-zarish-index.yml`:**

```yaml
env:
  ZARISH_INDEX_VERSION: "v1.0.0"  # Never use 'latest' — pin to specific release
```

---

## 7. G2A engine stages that use ZARISH-INDEX

The ZarishSphere G2A Engine converts any international standard document into deployable digital assets in 6 stages. ZARISH-INDEX is used in stages 1, 2, 3, 4, and 5.

| Stage | Name | ZARISH-INDEX role |
|---|---|---|
| Stage 1 | INGEST | Identify and validate the uploaded document against ZARISH-INDEX records |
| Stage 2 | PARSE | Use `name_full`, `name_short`, `standard_id` to extract and cross-reference citations |
| Stage 3 | EXTRACT | Map domain, sub-domain, mandate, entry type, and relationships to structure outputs |
| Stage 4 | VALIDATE | Check `status` field — block generation if standard is Withdrawn or Superseded |
| Stage 5 | STAGE | Embed `official_url` and `zarish_id` into all generated content as authoritative references |
| Stage 6 | DEPLOY | Use `national_adoption_of` relationship edges to validate local regulatory equivalence |

---

## 8. Cross-references

- → **004-zs-index/001-zarish-index-overview.md** — ZARISH-INDEX mission, scope, and governance
- → **004-zs-index/003-metadata-schema.md** — Full 22-field schema, supplementary tables, and the zarish_id convention
- → **004-zs-index/004-harvesting-policy.md** — Harvesting policy and build plan; §13 carries the QA and validation protocol (a dedicated QA protocol document is planned)
- → **003-zs-platform/004-g2a-engine.md** — Full G2A Engine description — the 6-stage pipeline with worked example
- → **005-zs-standards/002-transformation-model.md** — INDEX-to-STANDARDS pipeline: data mapping and field translation rules
- → **005-zs-standards/003-standards-schema.md** — The formal schema every standard entity must conform to for platform consumption
- **zarishsphere/zs-module-zarish-index** — GitHub repository — the consuming module within ZarishSphere

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
