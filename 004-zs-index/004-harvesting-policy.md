# 004-harvesting-policy.md
## ZARISH-INDEX — Strategic Direction and Harvesting Policy
### World's First Universal Machine-Readable Standards Index · V1

**Document type:** Direction — V1
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable. Authoritative strategic direction for the ZARISH-INDEX project.
**Repository:** https://github.com/zarishsphere/zs-module-zarish-index

---

## Table of Contents

1. [What ZARISH-INDEX is](#1-what-zarish-index-is)
2. [Why this project exists](#2-why-this-project-exists)
3. [Mission, vision and guiding principles](#3-mission-vision-and-guiding-principles)
4. [Scope — the full universe of human standards](#4-scope--the-full-universe-of-human-standards)
5. [Master domain taxonomy — all 40 domains](#5-master-domain-taxonomy--all-40-domains)
6. [Scale and quantitative scope](#6-scale-and-quantitative-scope)
7. [Data architecture — three layers](#7-data-architecture--three-layers)
8. [Data schema — 22 fields per entry](#8-data-schema--22-fields-per-entry)
9. [ID convention](#9-id-convention)
10. [Free data sources](#10-free-data-sources)
11. [Free tool stack](#11-free-tool-stack)
12. [Phased build plan — 9 phases over 24 months](#12-phased-build-plan--9-phases-over-24-months)
13. [Quality assurance and validation protocol](#13-quality-assurance-and-validation-protocol)
14. [8-week execution sprint (current)](#14-8-week-execution-sprint-current)
15. [Integration with ZarishSphere G2A engine](#15-integration-with-zarishsphere-g2a-engine)
16. [Governance and maintenance model](#16-governance-and-maintenance-model)
17. [Milestones and success metrics](#17-milestones-and-success-metrics)
18. [Risk register](#18-risk-register)
19. [Cross-references](#19-cross-references)

---

## 1. What ZARISH-INDEX is

ZARISH-INDEX is an **autonomous, open research project** that maintains the world's most complete, freely accessible, machine-readable unified index of every global standard, framework, treaty, guideline, and classification system that governs human civilisation.

As of the last harvest, it covers **40 domains, 88,204+ measured entries, and 20,140+ relationship edges**. All data is published on GitHub under CC BY 4.0 and downloadable in CSV, JSON, and Parquet formats with no registration, paywall, or subscription. Entry targets quoted in this document — 5,000 for v1.0 and 50,000 for v3.0 — are targets, not measured counts.

ZARISH-INDEX is its own project. It serves any researcher, NGO, developer, or government worldwide. Its relationship to ZarishSphere is that of an **upstream data provider**, not a sub-module. It operates under CC BY 4.0; ZarishSphere code runs under Apache 2.0. No license conflict exists.

> **Plain-language statement:** The world has no single place where you can look up any global standard and find verified, structured, machine-readable information about it. ZARISH-INDEX is that place — built entirely with free tools, published entirely for free.

---

## 2. Why this project exists

### 2.1 The fragmentation crisis

Global standards are the intellectual infrastructure of civilisation. They define how medicines are tested, how buildings stand, how data is protected, how refugees are treated. But this infrastructure is profoundly fragmented:

| Standards body | Volume | Access |
|---|---|---|
| ISO | 25,703+ published standards | USD 120–400 per document |
| IEC | 12,000+ electrotechnical standards | EUR 200–350 per document |
| ITU | Thousands of telecommunications recommendations | Partially free |
| WHO | Hundreds of guidelines and classifications | Free but scattered |
| UN Treaties | 560+ multilateral treaties | Free but poorly structured |
| ILO NORMLEX | 190 Conventions, 206 Recommendations | Navigable only via their interface |
| Codex Alimentarius | 3,000+ food safety standards | Free but in 8 separate databases |

No single searchable, filterable, cross-referenced index covers even a fraction of this landscape.

### 2.2 Who is most harmed

The entities most harmed are those with fewest resources: NGOs in humanitarian settings, government agencies in low-income countries, researchers at underfunded institutions. A Bangladeshi health worker looking for the WHO standard, a refugee protection officer checking UNHCR guidelines, or a water engineer in a camp needing the Sphere WASH standard should not need to know which organisation to look in, in which database, with which syntax.

### 2.3 Why now

Three developments make this feasible for the first time in 2026:

1. **ISO Open Data** — all 25,703+ standard metadata records freely downloadable in CSV/JSON/Parquet with no registration required.
2. **Wikidata** — structured linked data on thousands of standards bodies, queryable via free SPARQL endpoint.
3. **GitHub + GitHub Pages** — free hosting, version control, and static website publishing at this dataset scale.

---

## 3. Mission, vision and guiding principles

### 3.1 Vision

> A world in which any person, in any country, in any sector, can instantly find, verify, and understand any global standard that affects their work or life — for free, in their language, in the format they need.

### 3.2 Mission

> Build, maintain, and freely publish the world's most complete, accurate, and usable unified index of global standards, standards bodies, frameworks, treaties, and guidelines — covering every domain of human society, in a machine-readable and human-navigable format, requiring no subscription or payment to access.

### 3.3 Guiding principles

| # | Principle | In practice |
|---|---|---|
| 1 | Free forever | No paywall. No credit card. No registration. No subscription. Ever. |
| 2 | Completeness over perfection | Include everything with a credible claim to global standing, then verify iteratively. An incomplete entry is better than no entry. |
| 3 | Source truth over opinion | Every entry cites its authoritative source. We describe standards; we do not evaluate them. |
| 4 | Open by default | CC BY 4.0 license. All data on GitHub. All formats downloadable. |
| 5 | Machine-readable first | CSV, JSON, Parquet are primary outputs. Excel and web portal are derived views. |
| 6 | Community maintained | No single point of failure. Anyone can contribute a correction via GitHub pull request. |
| 7 | Layer neutral | International, regional, and national standards are equally valid. No governance level is privileged. |
| 8 | Politically neutral | We describe what a standard says and who issued it. No positions on contested standards or disputed territories. |
| 9 | Continuously updated | Standards change. Automated checks and community reports keep the index current. |
| 10 | Domain agnostic | No domain is too obscure. Sports, arts, space, military — all are in scope. |

---

## 4. Scope — the full universe of human standards

### 4.1 Inclusion criteria

A standard is included if it meets **all three** of the following:

1. **Formal issuance** — Published by an identifiable standards body, international organisation, intergovernmental body, recognised professional association, or national government agency acting in a standardisation capacity.
2. **Cross-jurisdictional reach** — Intended to apply across at least two countries, or formally adopted/referenced by international bodies, or forms the basis of national legislation in multiple jurisdictions.
3. **Referenceability** — Has a stable, citable identifier (standard number, convention number, resolution number, treaty name, etc.) and a verifiable primary source URL.

### 4.2 Explicitly in scope

- All ISO standards (25,703+ published documents)
- All IEC standards (12,000+ electrotechnical standards)
- All ITU Recommendations (ITU-T, ITU-R, ITU-D)
- All UN Treaties and Conventions (560+ multilateral treaties)
- All ILO Conventions and Recommendations (190 + 206)
- All WHO Guidelines, Frameworks, Classifications (ICD-11, ICF, EML, IHR, and hundreds of guidelines)
- All UN Specialised Agency standards (ICAO SARPs, IMO Conventions, IAEA Safety Standards, Codex, WIPO treaties, WMO Technical Regulations)
- All financial stability standards (BCBS Basel Accords, IOSCO Principles, FATF 40 Recommendations, IFRS/IAS)
- All human rights instruments (UDHR, ICCPR, ICESCR, CEDAW, CRC, CRPD, and all Optional Protocols)
- All humanitarian standards (Sphere, CHS, INEE, IASC Guidelines, UNHCR Policies)
- All environmental treaties (Paris Agreement, CBD, Ramsar, CITES, Stockholm, Basel, Rotterdam, Minamata, Montreal, UNCLOS)
- All major regional standards (CEN/CENELEC/ETSI for Europe; ASEAN; AU/ARSO)
- All NSB national standards where the NSB is an ISO member (175 national bodies)
- All major industry/professional standards (IEEE, IETF RFCs, W3C Recommendations, ASTM, ASME, NFPA, SAE)

### 4.3 Explicitly out of scope

- Proprietary company-specific standards not adopted as industry or national standards
- Draft standards not yet formally adopted
- Purely local/municipal standards with no cross-jurisdictional reference
- Classified or restricted-access military technical specifications

---

## 5. Master domain taxonomy — all 40 domains

The index is organised into **40 primary domains** across **6 meta-layers**. The master taxonomy is defined canonically in → **004-zs-index/002-domain-taxonomy-40.md**; the tables below are the harvest-phase view of the 40 domains, and the taxonomy document governs where the two differ.

### Meta-layer 1: Life sciences & health (domains 1–6)

| # | Domain | Key coverage |
|---|---|---|
| 1 | Health & medical | WHO guidelines, ICD-11, ICF, ICHI, FHIR, SPHERE Health, MPEHS, EWARS |
| 2 | Food safety & agriculture | Codex Alimentarius (3,000+ standards), MRLs, food labelling |
| 3 | Animal health & veterinary | WOAH Terrestrial + Aquatic Codes, zoonoses, animal welfare |
| 4 | Plant health & phytosanitary | All ISPMs under IPPC, pest risk analysis, wood packaging |
| 5 | Occupational health & safety | ILO OSH Conventions, ISO 45001, OSHA, construction safety |
| 6 | Pharmaceuticals & medicines | ICH Q/S/E/M guidelines, GMP, pharmacopoeias, PIC/S |

### Meta-layer 2: Physical sciences & engineering (domains 7–14)

| # | Domain | Key coverage |
|---|---|---|
| 7 | Measurement & metrology | SI units, Metre Convention, BIPM, OIML, ILAC MRA |
| 8 | Manufacturing & industry | Industrial automation, pressure equipment, welding, materials |
| 9 | Electrical & electronics | IEC 60364, IEC 60601, IEC 61000, batteries, lighting |
| 10 | Construction & built environment | ISO TC 59, BIM (ISO 19650), smart cities (ISO 37120/37122), NFPA |
| 11 | Chemical & process industries | REACH, CLP, GHS, process safety, IUPAC nomenclature |
| 12 | Transportation & mobility | ICAO SARPs, IMO SOLAS, UNECE vehicle regulations, rail, cycling |
| 13 | Space & aerospace | CCSDS, ECSS, FAA/EASA technical standards, ISO TC 20 |
| 14 | Energy | IEC TC 82 (solar), IEC 61850 (smart grid), API petroleum, nuclear safety |

### Meta-layer 3: Society & governance (domains 15–22)

| # | Domain | Key coverage |
|---|---|---|
| 15 | Human rights | 9 core UN treaty instruments + Optional Protocols, UDHR, all GCs |
| 16 | Labour | All 190 ILO Conventions + 206 Recommendations, MLC 2006 |
| 17 | Humanitarian | Sphere 2018, CHS, INEE, all IASC Guidelines, UNHCR policies, ICRC |
| 18 | Migration & refugees | 1951 Convention + 1967 Protocol, UNHCR data protection, GCR |
| 19 | Governance & anti-corruption | UNCAC, UNCLAS, OECD Anti-Bribery, Open Government OGP |
| 20 | Education & research | UNESCO qualifications conventions, ISCED, FAIR principles, DOI |
| 21 | Cultural heritage & arts | UNESCO World Heritage, ICCROM, ICOM museum standards |
| 22 | Social protection | ILO Social Security Minimum Standards, CGAP financial inclusion |

### Meta-layer 4: Economy & trade (domains 23–30)

| # | Domain | Key coverage |
|---|---|---|
| 23 | Finance & banking | BCBS Basel III, IOSCO, FATF 40 Recommendations, ISO 20022 |
| 24 | Trade & customs | WTO agreements, Incoterms 2020, UNCITRAL, HS codes |
| 25 | Logistics & supply chain | GS1 (GTIN, GLN, SSCC), EPCIS, GS1-GTS2, WHO Cold Chain |
| 26 | Procurement | UNCITRAL Model Law, World Bank SBDs, PEFA framework |
| 27 | Insurance | IAIS Insurance Core Principles, OECD insurance guidelines |
| 28 | Aid & development | IATI standard, OECD DAC criteria, World Bank ESF, UN evaluation norms |
| 29 | Sustainability & ESG | GRI Standards, IFRS S1/S2, ISSB, GHG Protocol, TCFD, SBTi |
| 30 | Intellectual property | WIPO treaties, PCT, Berne Convention, Madrid System |

### Meta-layer 5: Technology & infrastructure (domains 31–36)

| # | Domain | Key coverage |
|---|---|---|
| 31 | Digital & ICT | IETF RFCs (9,000+), W3C Recommendations, ISO/IEC 27001, ISO 42001 |
| 32 | Cybersecurity | NIST CSF, CIS Controls, ISO/IEC 27000 family, SOC 2 framework |
| 33 | Data & AI governance | GDPR, ISO/IEC 42001, NIST AI RMF, ITU AI Standards 2025 |
| 34 | Telecommunications | ITU-T/R/D Recommendations, 3GPP, IEEE 802.x |
| 35 | Water & sanitation | SPHERE WASH, WHO/UNICEF JMP, ISO 24510/24511/24512 |
| 36 | Urban & infrastructure | ISO 37101, ISO 37120, smart city frameworks, IEC 61968 |

### Meta-layer 6: Environment & natural systems (domains 37–40)

| # | Domain | Key coverage |
|---|---|---|
| 37 | Climate & energy | Paris Agreement + all COP decisions, GHG Protocol, IPCC assessment frameworks |
| 38 | Biodiversity & ecosystems | CBD + protocols, CITES Appendices, Kunming-Montreal GBF (2022), Ramsar |
| 39 | Oceans & water | UNCLOS, IMO MARPOL, ISO 14001, GEF Ocean frameworks |
| 40 | Chemicals & waste | Stockholm, Basel, Rotterdam, Minamata Conventions, UNEP Mercury |

---

## 6. Scale and quantitative scope

| Metric | Value |
|---|---|
| Total master entries (measured, last harvest) | 88,204+ |
| Relationship edges (measured, last harvest) | 20,140+ |
| Canonical domains covered | 40 of 40 |
| Standards bodies catalogued | 600+ |
| ISO entries (auto-ingested) | 25,703 |
| IETF RFCs | 9,000+ |
| ILO Conventions | 190 |
| ILO Recommendations | 206 |
| UN multilateral treaties | 560+ |
| WHO guidelines (key) | 200+ |
| Humanitarian standards (IASC/Sphere/CHS) | 80+ |

---

## 7. Data architecture — three layers

ZARISH-INDEX uses a **three-layer architecture** to serve different query needs:

### 7.1 Tabular canonical layer (primary)

The master truth layer. All records live here.

- **Storage:** CSV (universal), JSON, Parquet (analytics)
- **Tool:** GitHub repository + Google Sheets (working database)
- **Purpose:** Source of truth, bulk download, machine ingestion
- **Analogy:** This is the warehouse. All other layers derive from it.

### 7.2 Graph layer (relationships)

Models inter-standard relationships that cannot be represented in flat tables.

- **Storage:** JSON edge files in GitHub + Neo4j Community (optional local)
- **Relationship types:** `supersedes`, `references`, `adopted_by`, `implements`, `aligned_with`, `supplements`, `national_adoption_of`
- **Confidence field:** Each edge carries `confidence` (0.0–1.0) and `provenance` (source that confirms the relationship)
- **Purpose:** Cross-domain linkage, dependency chains, G2A engine cross-referencing
- **Analogy:** This is the map of connections between warehouse items.

### 7.3 Vector layer (semantic search)

Enables natural language retrieval over standard metadata.

- **Storage:** Qdrant (open source, self-hosted)
- **Embeddings:** Generated from `name_full`, `why_it_matters`, `key_outputs` fields
- **Query pattern:** GraphRAG — symbolic relationship lookup + semantic retrieval combined
- **Purpose:** "Find standards about vaccination in humanitarian settings" — returns ranked results across 40 domains
- **Analogy:** This is the intelligent librarian who reads context, not just keyword matches.

---

## 8. Data schema — 22 fields per entry

The authoritative, complete schema definition is → **004-zs-index/003-metadata-schema.md**. The summary below is the harvest-phase view; where the two differ (the `status` and `entry_type` enum counts, and the empty-value convention), the schema document governs.

Every entry carries all 22 fields. Empty values are `null`, never omitted.

| # | Field | Type | Description | Example |
|---|---|---|---|---|
| 1 | `zarish_id` | String | Unique ZARISH identifier | `HL-ISO-15189-2022` |
| 2 | `entry_type` | Enum | Standard / Framework / Treaty / Guideline / Regulation / Classification / Code of Practice / Recommendation | `Standard` |
| 3 | `meta_layer` | Enum | L1–L6 (see taxonomy) | `L1 Life Sciences` |
| 4 | `domain` | String | Primary domain from 40-domain taxonomy | `Health & Medical` |
| 5 | `sub_domain` | String | Sub-category within domain | `Clinical Laboratories` |
| 6 | `name_full` | String | Complete official name | `Medical laboratories — Requirements for quality and competence` |
| 7 | `name_short` | String | Common name or acronym | `ISO 15189` |
| 8 | `standard_id` | String | Official identifier from issuing body | `ISO 15189:2022` |
| 9 | `issuer` | String | Name of issuing body | `ISO (TC 212)` |
| 10 | `issuer_type` | Enum | UN Agency / ISO / IEC / ITU / Industry SDO / Professional Body / NGO / Intergovernmental / National Government | `ISO` |
| 11 | `governance_layer` | Enum | International / Regional / National | `International` |
| 12 | `geographic_scope` | String | Countries or regions where formally applicable | `Global — 175 ISO member countries` |
| 13 | `year_published` | Integer | Year of current edition | `2022` |
| 14 | `year_first` | Integer | Year first published | `2003` |
| 15 | `status` | Enum | Active / Withdrawn / Superseded / Under Development / Under Review | `Active` |
| 16 | `mandate` | Enum | Mandatory / Voluntary / Voluntary-with-regulatory-adoption / Treaty-binding | `Voluntary` |
| 17 | `sector_applicability` | String | Who must or should use this standard | `Healthcare laboratories / accreditation bodies / regulators` |
| 18 | `why_it_matters` | String | Plain-language significance — LLM-assisted draft, human-reviewed | `Defines quality requirements for medical labs; basis for accreditation in 100+ countries` |
| 19 | `key_outputs` | String | Main standards, versions, or elements | `ISO 15189:2022 (third edition); covers pre-, examination, and post-examination processes` |
| 20 | `official_url` | URL | Primary source URL — authoritative and stable | `https://www.iso.org/standard/76677.html` |
| 21 | `data_source` | String | How this entry's data was obtained | `ISO Open Data CSV + manual verification` |
| 22 | `notes` | String | Additional contextual information | `Replaced 2012 edition; significant restructuring of management requirements` |

### 8.1 Supplementary tables

| Table | Purpose | Key fields |
|---|---|---|
| Table B — Standards bodies register | One record per issuing organisation | `org_id`, `org_name`, `org_acronym`, `org_type`, `founding_year`, `hq_country`, `wikidata_qid`, `official_url`, `standard_count` |
| Table C — Relationships map | Inter-standard and inter-organisation edges | `from_id`, `to_id`, `relationship_type`, `confidence`, `provenance`, `notes` |
| Table D — Ratification & adoption tracker | Treaty ratification status by country | `zarish_id`, `country_iso3`, `status`, `date`, `reservations`, `source_url` |

Table E — Research Task Matrix (research progress by domain and phase) is defined in → **004-zs-index/003-metadata-schema.md** §4.4.

---

## 9. ID convention

ZARISH IDs are deterministic and human-readable:

```
[DOMAIN_CODE]-[ISSUER_CODE]-[STD_NUMBER]-[YEAR]
```

| Example | Breakdown |
|---|---|
| `HL-ISO-15189-2022` | Health, ISO, standard 15189, 2022 edition |
| `HR-UN-CRC-1989` | Human Rights, UN, Convention on the Rights of the Child, 1989 |
| `ICT-IETF-RFC9110-2022` | ICT, IETF, RFC 9110, 2022 |
| `FS-CAC-GL21-2021` | Food Safety, Codex Alimentarius Commission, GL 21, 2021 |
| `FB-BCBS-BASELIII-2010` | Finance, BCBS, Basel III, 2010 |
| `HM-IASC-SPHERE-2018` | Humanitarian, IASC/Sphere, 2018 edition |

ZarishSphere ZARISH-INDEX Standard IDs use the `ZI-` prefix format for internal cross-referencing:

```
ZI-[DOMAIN_CODE]-[NNNNN]
```

Example: `ZI-HEALTH-00001`

---

## 10. Free data sources

### 10.1 Tier 1 — bulk free sources (highest priority)

| Source | Volume | URL | License |
|---|---|---|---|
| ISO Open Data | 25,703+ standards | iso.org/open-data.html | CC BY |
| Wikidata SPARQL | Thousands of bodies | query.wikidata.org | CC0 |
| IETF RFC Editor | 9,000+ RFCs | rfc-editor.org | IETF Trust |
| UN Treaty Collection | 560+ treaties | treaties.un.org | UN copyright (research free) |
| ILO NORMLEX + API | 190 Conventions + 206 Recommendations | normlex.ilo.org | ILO copyright (research free) |
| WHO IRIS (OAI-PMH) | WHO publications | iris.who.int | WHO copyright (research free) |
| Wikipedia standards list | 200+ bodies | en.wikipedia.org | CC BY-SA 3.0 |
| WTO TBT code bodies | 200+ bodies | tbtcode.iso.org | ISO/WTO (research free) |
| Codex Alimentarius | 3,000+ food standards | fao.org/codex-texts | FAO/WHO (research free) |
| NIST standards list | Free-access standards | nist.gov/standardsgov | Public domain |

### 10.2 Tier 2 — domain-specific free portals

| Portal | Domain | URL |
|---|---|---|
| OHCHR Human Rights Bodies | Human rights | ohchr.org/treaties |
| IAEA Safety Standards | Nuclear | iaea.org/safety-standards |
| GRI Standards | Sustainability | globalreporting.org/standards |
| SASB Standards (77 industry) | ESG | sasb.org/standards |
| Sphere Handbook | Humanitarian | spherestandards.org |
| INEE Minimum Standards | Education in emergencies | inee.org/minimum-standards |
| IATI Standard | Aid transparency | iatistandard.org |
| UNFCCC texts | Climate | unfccc.int |
| CBD texts | Biodiversity | cbd.int/convention/text |
| W3C Recommendations | Web standards | w3.org/standards |
| NIST publications | Digital/Cyber | nvlpubs.nist.gov |

---

## 11. Free tool stack

Every tool is **completely free** with no credit card requirement.

```
COLLECTION LAYER
├── Python 3 (open source)
├── requests + BeautifulSoup (HTTP + scraping)
├── SPARQLWrapper (Wikidata queries)
└── wget / curl (bulk download)

STORAGE & CURATION LAYER
├── GitHub (free — public repos, unlimited collaborators)
├── Google Sheets (10M cells, free Google account)
├── CSV / JSON / Parquet (universal formats)
└── SQLite (file-based, zero infrastructure)

AUTOMATION LAYER
├── GitHub Actions (2,000 minutes/month free)
│   ├── 101--on-push--validate-schema.yml
│   ├── 102--on-schedule--check-urls.yml
│   ├── 103--on-schedule--sync-iso-data.yml
│   ├── 201--on-release--build-artifacts.yml
│   └── 301--on-pull-request--lint-data.yml
└── Google Apps Script (Google Sheets automation)

PUBLICATION LAYER
├── GitHub Pages (free static hosting — zarishsphere.github.io/zarish-index)
├── Markdown documentation (renders on GitHub)
└── Observable (free interactive data notebooks — optional)

ADVANCED LAYERS (Phase 4+, still free)
├── Qdrant (self-hosted vector DB — open source, free)
├── NocoDB (Airtable alternative — MIT, free)
└── Baserow (open source, MIT, free cloud tier)
```

---

## 12. Phased build plan — 9 phases over 24 months

| Phase | Name | Duration | Primary domains | Deliverable |
|---|---|---|---|---|
| 0 | Setup & infrastructure | 2 weeks | All | GitHub org, repo, Pages, ISO seed, governance docs |
| 1 | Seed from bulk sources | 4 weeks | All (seeding) | 35,000–40,000 entries from ISO + IETF + ILO + UN + Codex |
| 2 | Human rights, humanitarian & development | 3 weeks | 15, 16, 17, 20 | Comprehensive verified coverage of free-text domains |
| 3 | Environment, climate & natural systems | 4 weeks | 37–40 | All environmental treaties, IAEA, GHG Protocol |
| 4 | Finance, trade & economic governance | 4 weeks | 23–27 | Basel, FATF, WTO agreements, IATI, OECD DAC |
| 5 | Technology, digital & ICT | 4 weeks | 31–34 | All 9,000+ IETF RFCs, W3C, NIST, ISO 27001 family |
| 6 | Physical sciences & engineering | 4 weeks | 7–14 | ISO TC coverage, IEC series, ICAO, IMO |
| 7 | Regional & national standards | 4 weeks | All | CEN/CENELEC/ETSI, ASEAN, AU/ARSO, 30 priority NSBs |
| 8 | Enrichment, QA & relationships | 4 weeks | All | `why_it_matters` content, 20K+ relationship edges, QA pass |
| 9 | Public API, search & community launch | 3 weeks | All | Faceted search, public API, contributor guides, v1.0 release |

---

## 13. Quality assurance and validation protocol

### 13.1 Automated quality gates (GitHub Actions)

All PRs and main-branch pushes trigger:

```yaml
# 301--on-pull-request--validate-data.yml
checks:
  - schema_validation: every field in 22-field master schema is present
  - id_uniqueness: no duplicate zarish_id values
  - url_reachability: official_url returns HTTP 200 (sample)
  - enum_conformance: status, mandate, entry_type match defined enums
  - relationship_integrity: all edge IDs reference valid zarish_ids
```

### 13.2 Human review requirements

| Content type | Review required | Reviewer |
|---|---|---|
| Auto-ingested ISO metadata | Spot-check 5% | Curator (Ariful) |
| LLM-generated `why_it_matters` | 100% review before publish | Curator (Ariful) |
| Relationship edges from new sources | 100% review | Curator (Ariful) |
| Domain expansions | Full review | Curator (Ariful) |
| Community-submitted corrections | Review and merge | Curator (Ariful) |

### 13.3 LLM-assisted enrichment (non-blocking)

LLM (Llama 3.1 8B local or Gemini free tier) assists with:

- Drafting `why_it_matters` notes from source metadata
- Suggesting domain/sub-domain tags for unlabelled entries
- Detecting potential duplicate entries with similar names
- Generating multilingual summaries for high-impact entries

**Rule:** LLM output is a draft. Human review is mandatory before any LLM-generated content is published. The `data_source` field records `LLM-draft (Llama 3.1 8B) — human reviewed` for every such entry.

---

## 14. 8-week execution sprint (current)

This is the active implementation sprint as of June 2026, beginning from the final roadmap layer:

### Weeks 1–2: Ingest & graph foundation

- Ingest current ISO Open Data + Wikidata updates into canonical CSV schema
- Stand up initial graph schema (`Table C — Relationships Map`) in JSON format
- Implement `ILO supersedes` relationship extraction (ILO Conventions that supersede earlier ones)
- Implement `ISO references` link extraction from ISO Open Data cross-references
- Validate all 88,204+ measured entries against the master schema

### Weeks 3–4: Automation & enrichment

- Launch `103--on-schedule--sync-iso-data.yml` — automated daily ISO delta ingestion
- Launch `301--on-pull-request--validate-data.yml` — schema validation gate on all PRs
- Enable first LLM-assisted `why_it_matters` enrichment pass (Llama 3.1 8B) for the top 500 health standards
- Human approval of all LLM-generated content before merge

### Weeks 5–6: Publication & API

- Publish MVP: GitHub Pages with faceted navigation by domain, issuer, status, and year
- Generate release artifacts: `zarish-index-v1.0.csv`, `zarish-index-v1.0.json`, `zarish-index-v1.0.parquet`, `relationships-v1.0.json`
- Expose lightweight public API (static JSON index on GitHub Pages — no server required)
- Publish changelog feed (GitHub Releases + RSS)

### Weeks 7–8: Expansion & partnerships

- Expand to 5 high-impact domains with enriched relationship mapping (Health, Humanitarian, Human Rights, Finance, Digital)
- Publish contributor onboarding guide (`CONTRIBUTING.md`)
- Publish partnership outreach package for universities and civil society
- Submit ZARISH-INDEX for Zenodo archival (permanent DOI)

---

## 15. Integration with ZarishSphere G2A engine

ZARISH-INDEX is the upstream knowledge base for the ZarishSphere Guideline-to-Action (G2A) Engine. The relationship is a one-way data feed.

```
ZARISH-INDEX (upstream, autonomous)
│
│  Provides to G2A Engine:
│  ├── Unique identifiers for 88,204+ global standards
│  ├── Domain taxonomy (40 domains, hierarchical)
│  ├── Standards body metadata (600+ SDOs)
│  ├── Type labels: TYPE-A, TYPE-B, TYPE-C
│  ├── Status flags: ACTIVE, BETA, DEPRECATED
│  ├── 20,140 relationship edges
│  └── Source URLs and authoritative references
│
▼
ZarishSphere G2A Engine — Stage 1 (INGEST) and Stage 2 (PARSE)
│
│  Consumes ZARISH-INDEX to:
│  ├── Identify which standard a document belongs to
│  ├── Validate standard citations in uploaded guidelines
│  ├── Resolve standard identifiers to canonical metadata
│  ├── Classify G2A outputs by domain
│  ├── Skip deprecated standards in form / rule generation
│  └── Build cross-domain linkage in generated content
```

### 15.1 How ZarishSphere pulls ZARISH-INDEX data

| Method | Frequency | Use case |
|---|---|---|
| GitHub Actions download | On every ZARISH-INDEX release | Sync master CSV/JSON/Parquet to `zs-module-zarish-index` |
| Local PostgreSQL table | Persistent after download | Runtime queries during G2A processing |
| Qdrant vector index | Built from text fields on sync | Semantic search over standards during G2A parsing |

The consuming repository is `zarishsphere/zs-module-zarish-index`.

### 15.2 Autonomy boundaries

| Rule | Reason |
|---|---|
| ZARISH-INDEX changes do not require ZarishSphere approval | ZARISH-INDEX serves 40+ domains globally |
| ZarishSphere changes do not affect ZARISH-INDEX content | ZarishSphere is a consumer, not a contributor |
| ZARISH-INDEX uses CC BY 4.0 | ZarishSphere code uses Apache 2.0 — no license conflict |
| ZARISH-INDEX can be used by any project | It is not ZarishSphere property |
| ZarishSphere can use any data source | ZARISH-INDEX is the preferred source, not the only one |

ZarishSphere pins to a specific ZARISH-INDEX release tag in `zs-module-zarish-index`. Upgrades are explicit, reviewed, and logged in an ADR.

---

## 16. Governance and maintenance model

### 16.1 Project governance

| Role | Person | Responsibilities |
|---|---|---|
| Project Lead / Curator | Mohammad Ariful Islam | Final approval on all content, schema changes, releases |
| Domain Researcher (volunteer) | Open to community | Research and submission of domain-specific entries |
| Technical Reviewer (volunteer) | Open to community | Schema validation, automation, data quality |

### 16.2 Contribution workflow

1. Contributor opens a GitHub Issue using the appropriate template (new entry / correction / URL update / domain expansion)
2. Curator reviews the issue and approves or requests changes
3. Contributor submits a GitHub Pull Request with changes following the ZUSS naming standard (→ **001-zs-meta/004-zarishsphere-writing-rules.md**)
4. Automated validation gates run on the PR
5. Curator reviews the PR and merges after human approval

### 16.3 Release cycle

- **Major releases** (new domains, schema changes): quarterly
- **Minor releases** (new entries, corrections): monthly
- **Patch releases** (URL updates, typo fixes): as needed via PR

All releases published to GitHub Releases with auto-generated CSV/JSON/Parquet artifacts.

---

## 17. Milestones and success metrics

| Milestone | Target | Metric |
|---|---|---|
| Phase 0 complete | July 2026 | GitHub repo live, ISO seed committed, GitHub Pages active |
| 50,000 verified entries (target) | August 2026 | 50K rows in canonical CSV passing all validation gates — a target aligned with the v3.0 entry target, not a claim of current measured counts |
| All 40 domains with seed entries | September 2026 | Zero domains showing 0 entries in coverage report |
| v1.0 public release | October 2026 | GitHub release with CSV + JSON + Parquet artifacts and changelog |
| Public faceted search live | October 2026 | GitHub Pages search functional for domain, issuer, status filters |
| Measured entries validated | December 2026 | All 88,204+ measured entries fully validated and enriched |
| Community contributors active | January 2027 | 5+ community PRs merged from external contributors |
| 100,000 entries (target) | March 2027 | 100,000 rows in canonical CSV |
| Zenodo archival (permanent DOI) | Ongoing | Every major release mirrored to Zenodo |

---

## 18. Risk register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Single-maintainer dependency | High | High | GitHub data is public; any researcher can fork. Contributor playbooks published. |
| ISO Open Data schema change | Medium | High | Pin to specific ISO release tags. Schema change triggers migration ADR. |
| Wikidata SPARQL timeout | Medium | Low | Paginate queries. Cache results in CSV. Backup from Wikipedia API. |
| LLM-generated content errors | High | Medium | 100% human review before publication. `data_source` field tracks LLM provenance. |
| URL link rot | High | Low | Monthly GitHub Actions URL health check. Community corrections via Issues. |
| Scope creep | Medium | Medium | 40-domain taxonomy is fixed. New domains require Curator ADR. |

---

## 19. Cross-references

- → **004-zs-index/001-zarish-index-overview.md** — Project charter and entry-count targets
- → **004-zs-index/002-domain-taxonomy-40.md** — Canonical 40-domain taxonomy; supersedes the §5 harvest-phase view where they differ
- → **004-zs-index/003-metadata-schema.md** — Authoritative 22-field schema; governs where the §8 summary differs
- → **004-zs-index/005-zarish-index-to-platform.md** — Integration architecture for the G2A Engine
- → **003-zs-platform/004-g2a-engine.md** — The G2A (Guideline-to-Action) Engine
- → **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS naming and formatting rules applied to contributions

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
