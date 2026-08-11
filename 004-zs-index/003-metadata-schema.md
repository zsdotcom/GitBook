# 003-metadata-schema.md
## ZARISH-INDEX — Data Schema
### Master Entry Schema · Supplementary Tables · ID Convention · V1

**Document type:** Specification — Canonical
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable. All ZARISH-INDEX entries must conform to this schema.

---

## Table of Contents

1. [Schema overview](#1-schema-overview)
2. [Master schema — 22 fields per entry](#2-master-schema--22-fields-per-entry)
3. [Field definitions and validation rules](#3-field-definitions-and-validation-rules)
4. [Supplementary tables](#4-supplementary-tables)
5. [ID convention](#5-id-convention)
6. [Relationship types](#6-relationship-types)
7. [Enum value registries](#7-enum-value-registries)
8. [Schema validation rules](#8-schema-validation-rules)
9. [Example entries](#9-example-entries)
10. [Cross-references](#10-cross-references)

---

## 1. Schema overview

Every ZARISH-INDEX entry is a structured record with exactly 22 fields. Four supplementary tables extend the master index for standards bodies, inter-standard relationships, ratification tracking, and research task management.

**Primary output file:** `zarish_master.csv`
**Schema version:** 1.0
**Maintained in:** `SCHEMA.md` (repo root) and this document

All field names use `snake_case`. All enum values use `Title Case` or `SCREAMING_SNAKE_CASE` as specified per field. Unknown values are recorded as `UNKNOWN` — never left blank.

**Schema reconciliation note:** the harvesting policy (→ **004-zs-index/004-harvesting-policy.md**) carries a condensed view of this schema in its §8. Where the two differ — the `status` enum (6 valid values here, 5 listed there), the `entry_type` enum (10 valid values here, 8 listed there), and the empty-value convention (`UNKNOWN` never blank here, `null` never omitted there) — this document is authoritative.

---

## 2. Master schema — 22 fields per entry

| # | Field Name | Type | Required | Description |
|---|---|---|---|---|
| 1 | `zarish_id` | String | Yes | Unique ZARISH identifier — deterministic, human-readable |
| 2 | `entry_type` | Enum | Yes | Classification of what this entry is |
| 3 | `meta_layer` | Enum | Yes | One of the 6 meta-layers |
| 4 | `domain` | String | Yes | One of the 40 canonical domains |
| 5 | `sub_domain` | String | Yes | Sub-category within domain; use `UNKNOWN` if not determinable |
| 6 | `name_full` | String | Yes | Complete official name as published by the issuing body |
| 7 | `name_short` | String | Yes | Common name, acronym, or short title |
| 8 | `standard_id` | String | Yes | Official identifier from issuing body; use `NONE` if no formal ID exists |
| 9 | `issuer` | String | Yes | Full name of issuing body |
| 10 | `issuer_type` | Enum | Yes | Classification of the issuing body |
| 11 | `governance_layer` | Enum | Yes | Geographic scope of authority |
| 12 | `geographic_scope` | String | Yes | Countries or regions where formally applicable |
| 13 | `year_published` | Integer | Yes | Year of current/most recent edition; use `UNKNOWN` if not confirmed |
| 14 | `year_first` | Integer | No | Year first published; use `UNKNOWN` if not confirmed |
| 15 | `status` | Enum | Yes | Current lifecycle status |
| 16 | `mandate` | Enum | Yes | Binding or voluntary nature |
| 17 | `sector_applicability` | String | Yes | Who must or should use this standard |
| 18 | `why_it_matters` | String | Yes | 1–3 sentence plain-language explanation of significance |
| 19 | `key_outputs` | String | No | Main components, versions, or deliverables of this standard |
| 20 | `official_url` | URL | Yes | Primary source URL from the issuing body — not Wikipedia or secondary sources |
| 21 | `data_source` | String | Yes | Where this entry's data was obtained |
| 22 | `notes` | String | No | Additional contextual information; use `NONE` if nothing to add |

---

## 3. Field definitions and validation rules

### 3.1 zarish_id

**Format:** `[DOMAIN_CODE]-[ISSUER_CODE]-[SHORT_ID]-[YEAR]`

**Rules:**
- All uppercase
- Hyphens as separators only
- DOMAIN_CODE: two or three letters from the domain code registry (see → **004-zs-index/002-domain-taxonomy-40.md**)
- ISSUER_CODE: ISO = `ISO`, IEC = `IEC`, WHO = `WHO`, ILO = `ILO`, UN = `UN`, W3C = `W3C`, NIST = `NIST`, IETF = `IETF`, etc.
- SHORT_ID: abbreviated standard identifier, no spaces, no colons
- YEAR: four-digit year of current edition

**Examples:**

| Entry | zarish_id |
|---|---|
| ISO 15189:2022 (Medical Labs) | `HL-ISO-15189-2022` |
| IHR 2005 (International Health Regulations) | `HL-WHO-IHR-2005` |
| Sphere Handbook 2018 | `HM-IASC-SPHERE-2018` |
| ILO Convention 87 | `LE-ILO-C87-1948` |
| RFC 9110 (HTTP Semantics) | `ICT-IETF-RFC9110-2022` |
| GRI 1 Foundation 2021 | `SE-GRI-GRI1-2021` |
| WADA Code 2021 | `SR-WADA-CODE-2021` |

**Collision rule:** If two entries would generate the same zarish_id, append a `-B` suffix to the second.

---

### 3.2 entry_type

Classifies what kind of document or body this entry represents.

**Valid values:**

| Value | Definition |
|---|---|
| `Standard` | A normative document establishing requirements, specifications, or guidelines for products, services, or processes |
| `Framework` | A structured set of principles or guidance not constituting legally binding requirements |
| `Treaty` | An international agreement binding on ratifying states under international law |
| `Guideline` | A document providing direction or recommendations without normative force |
| `Regulation` | A legally binding rule enacted by a government or regulatory authority |
| `Classification` | A systematic taxonomy or code system (ICD, HS Code, ISCO, etc.) |
| `Code of Practice` | A set of practical recommendations for specific activities or professions |
| `Recommendation` | A formal statement of best practice from an authoritative body (ITU-R, ILO Recommendations) |
| `Standards Body` | The issuing organisation itself, where the body is the subject of the entry |
| `Protocol` | A supplementary treaty amending or extending an existing convention |

---

### 3.3 meta_layer

**Valid values:**

| Value | Domains Covered |
|---|---|
| `L1 Life Sciences` | Health, Food Safety, Animal Health, Plant Health, OHS, Pharmaceuticals (Domains 1–6) |
| `L2 Physical Sciences` | Metrology, Manufacturing, Electrical, Construction, Chemical, Materials, Aerospace, Space (Domains 7–14) |
| `L3 Society and Governance` | Human Rights, Labour, Humanitarian, Legal, Governance, Education, Culture, Sports (Domains 15–22) |
| `L4 Economy and Trade` | Finance, Trade, Supply Chain, ESG, Taxation (Domains 23–27) |
| `L5 Technology and Infrastructure` | ICT, Cybersecurity, AI, Energy, Transport, WASH, Urban, Defence (Domains 28–35) |
| `L6 Environment` | Climate, Marine, Biodiversity, Disaster Risk, Extractives (Domains 36–40) |

---

### 3.4 domain

Must exactly match one of the 40 canonical domain names from → **004-zs-index/002-domain-taxonomy-40.md**. Use the full name, not the code.

**Example:** `Health and Medical` — not `HL`, not `Health`, not `Health & Medical`.

---

### 3.5 name_full

The complete official name exactly as published by the issuing body.

**Rules:**
- Copy verbatim from the primary source — do not paraphrase
- For ISO standards: use the full title as shown on the ISO standard page (e.g., `Medical laboratories — Requirements for quality and competence`)
- For treaties: use the full treaty title (e.g., `International Health Regulations (2005)`)
- For ILO Conventions: use the full convention name plus number (e.g., `Freedom of Association and Protection of the Right to Organise Convention, 1948 (No. 87)`)

---

### 3.6 standard_id

The official unique identifier as assigned by the issuing body.

**Examples by body:**

| Issuer | standard_id format | Example |
|---|---|---|
| ISO | `ISO [number]:[year]` | `ISO 15189:2022` |
| IEC | `IEC [number]:[year]` | `IEC 60601-1:2005` |
| ISO/IEC joint | `ISO/IEC [number]:[year]` | `ISO/IEC 27001:2022` |
| IETF RFC | `RFC [number]` | `RFC 9110` |
| ILO Convention | `C[number]` | `C87` |
| ILO Recommendation | `R[number]` | `R198` |
| WHO | Document series + number | `WHO/2019-nCoV/IPC/2020.4` |
| ISPM | `ISPM [number]` | `ISPM 15` |
| WADA | Document title | `World Anti-Doping Code 2021` |
| Treaty (UN) | Treaty name + UNTS number | `UNTS 14668` |

Use `NONE` if the issuing body assigns no formal identifier.

---

### 3.7 issuer_type

**Valid values:**

| Value | Examples |
|---|---|
| `UN Agency` | WHO, ILO, ICAO, IMO, IAEA, UNESCO, FAO, UNHCR |
| `Treaty Body` | OHCHR treaty committees, UNFCCC Secretariat, IAEA safeguards |
| `ISO` | ISO (all technical committees) |
| `IEC` | IEC (all technical committees) |
| `ITU` | ITU-T, ITU-R, ITU-D |
| `Industry SDO` | IEEE, IETF, W3C, ASTM, ASME, API, NFPA, SAE, OASIS, ECMA |
| `Professional Body` | ICOM, ICOMOS, ICCROM, WADA, IOC, FIFA, World Athletics |
| `NGO` | Sphere Project, CHS Alliance, INEE, GRI |
| `Intergovernmental` | OECD, WTO, WCO, FSB, BCBS, BIS, FATF, IOSCO |
| `National Government` | NIST (USA), BSI (UK), DIN (Germany), BSTI (Bangladesh), ANSI (USA) |
| `Regional Body` | CEN/CENELEC, ETSI, ARSO, SARSO, ASEAN Standards |

---

### 3.8 governance_layer

**Valid values:**

| Value | Definition |
|---|---|
| `International` | Issued by a global body; intended for worldwide application |
| `Regional` | Issued by a regional body; applicable to a defined group of countries |
| `National` | Issued by a national body; applicable to a single country |

---

### 3.9 status

**Valid values:**

| Value | Definition |
|---|---|
| `Active` | Currently in force; has not been withdrawn, superseded, or retired |
| `Under Review` | Under active revision; current edition remains in force |
| `Under Development` | Not yet published; in preparation |
| `Superseded` | Replaced by a newer edition or a different standard; no longer recommended for use |
| `Withdrawn` | Formally retired; no replacement issued |
| `UNVERIFIED` | Status could not be confirmed from primary source — flag for review |

---

### 3.10 mandate

**Valid values:**

| Value | Definition |
|---|---|
| `Mandatory` | Legally required by national law, regulation, or treaty — cannot be opted out of |
| `Voluntary` | Not legally required; adopted by choice |
| `Voluntary-with-regulatory-adoption` | Voluntary at issuer level but made mandatory by one or more national regulations |
| `Treaty-binding` | Binding on all states that have ratified the treaty; not applicable to non-parties |

**Important:** This field reflects the standard's own issuing-body mandate type. Use `notes` to document where a voluntary standard has been made mandatory by specific regulatory frameworks (e.g., ISO 9001 made mandatory in EU aerospace procurement).

---

### 3.11 why_it_matters

The single most valuable field in the schema for non-expert users. Write 1–3 sentences that explain:
- What this standard governs or establishes
- Who is affected by it
- Why its absence or violation would matter in practice

**Rules:**
- Plain language — write for a non-specialist audience
- Active voice — avoid passive constructions
- Specific — name the affected sector, the consequence, or the scale
- Do not copy issuer descriptions verbatim — rewrite in accessible language
- This field is **mandatory** — `UNKNOWN` is not acceptable; `TODO` is acceptable during data entry but must be resolved before publication

**Bad example:** `Specifies requirements for quality and competence.`
**Good example:** `Defines quality and competence requirements for medical laboratories. A laboratory without ISO 15189 accreditation cannot participate in national health systems in most countries and cannot provide results that meet international clinical standards.`

---

### 3.12 official_url

**Rules:**
- Must be the issuing body's own domain — not Wikipedia, not a PDF aggregator, not a news article
- Must be verified live (HTTP 200) at time of entry creation
- For ISO standards: `https://www.iso.org/standard/[number].html`
- For WHO guidelines: `https://www.who.int/publications/...`
- For IETF RFCs: `https://www.rfc-editor.org/rfc/rfc[number]`
- For ILO: `https://normlex.ilo.org/dyn/normlex/en/f?p=NORMLEXPUB:12100:0::NO::P12100_ILO_CODE:[code]`

---

### 3.13 data_source

Record exactly where the metadata for this entry was obtained. Use specific, verifiable references.

**Valid data source formats:**

| Source | data_source value |
|---|---|
| ISO Open Data CSV | `ISO Open Data CSV, iso_deliverables_metadata.csv, downloaded 2026-05-01` |
| Wikidata | `Wikidata Q[number], retrieved 2026-05-01` |
| Manual primary source research | `Manual research from [URL], verified 2026-05-15` |
| ILO NORMLEX API | `ILO NORMLEX API, retrieved 2026-05-01` |
| RFC Editor JSON | `RFC Editor index JSON, rfc-editor.org/rfc/index.json, retrieved 2026-05-01` |
| Google Sheet curation | `Google Sheet curation, curator: [name], date: 2026-05-15` |

---

## 4. Supplementary tables

### 4.1 Table B — Standards Bodies Register

One record per issuing organisation. Stored in `data/reference/standards_bodies.csv`.

| # | Field | Type | Description |
|---|---|---|---|
| 1 | `org_id` | String | Unique ID: `ORG-[ACRONYM]-[COUNTRY_ISO2]` |
| 2 | `org_name` | String | Full official name |
| 3 | `org_acronym` | String | Official acronym |
| 4 | `org_type` | Enum | Same values as `issuer_type` |
| 5 | `founding_year` | Integer | Year established |
| 6 | `hq_country` | String | ISO 3166-1 alpha-2 code |
| 7 | `hq_city` | String | City of headquarters |
| 8 | `geographic_scope` | Enum | International / Regional / National |
| 9 | `governance_structure` | String | Intergovernmental / Treaty body / NGO / Industry / Government agency |
| 10 | `iso_member` | Enum | Full / Correspondent / Subscriber / Non-member |
| 11 | `wikidata_qid` | String | Wikidata Q-identifier |
| 12 | `official_url` | URL | Homepage URL |
| 13 | `standard_count` | Integer | Approximate number of standards issued |
| 14 | `parent_org_id` | String | `org_id` of parent body if applicable |
| 15 | `notes` | String | Additional context |

---

### 4.2 Table C — Relationships Map

Captures inter-standard and inter-organisation relationships. Stored in `data/relationships/relationships.csv`.

| # | Field | Type | Description |
|---|---|---|---|
| 1 | `from_id` | String | zarish_id of the source standard or body |
| 2 | `to_id` | String | zarish_id of the target standard or body |
| 3 | `relationship_type` | Enum | Type of relationship (see Section 6) |
| 4 | `confidence` | Enum | `source-confirmed` / `curator-reviewed` / `llm-suggested` |
| 5 | `notes` | String | Source citation or rationale |

**Critical rule:** `llm-suggested` relationship edges must not be published in official releases until a human curator confirms the relationship from a primary source and changes `confidence` to `curator-reviewed` or `source-confirmed`.

---

### 4.3 Table D — Ratification and Adoption Tracker

For treaty and convention entries only. One row per country per treaty. Stored in `data/relationships/ratification_tracker.csv`.

| # | Field | Type | Description |
|---|---|---|---|
| 1 | `zarish_id` | String | zarish_id of the treaty |
| 2 | `country_iso3` | String | ISO 3166-1 alpha-3 country code |
| 3 | `country_name` | String | Country name |
| 4 | `status` | Enum | `Ratified` / `Acceded` / `Signatory` / `Not party` |
| 5 | `date` | String | ISO 8601 date of ratification/accession |
| 6 | `reservations` | String | Key reservations if any; `NONE` if no reservations |
| 7 | `source_url` | URL | Primary source URL for this ratification data |

**Primary source:** OHCHR Treaty Body Database (free JSON downloads), UN Treaty Collection.

---

### 4.4 Table E — Research Task Matrix

Tracks research progress by domain and phase. Used by automated `quality-gate` and `research-tasks` Makefile targets. Stored in `data/reports/research_tasks.csv`.

| # | Field | Type | Description |
|---|---|---|---|
| 1 | `task_id` | String | Unique task identifier: `TASK-[DOMAIN_CODE]-[NNN]` |
| 2 | `domain` | String | Domain name |
| 3 | `phase` | Integer | Phase number (1–9) |
| 4 | `description` | String | What needs to be done |
| 5 | `status` | Enum | `Not started` / `In progress` / `Complete` / `Blocked` |
| 6 | `assigned_to` | String | Contributor name or `Unassigned` |
| 7 | `target_entries` | Integer | Estimated number of entries this task will produce |
| 8 | `actual_entries` | Integer | Entries actually produced |
| 9 | `notes` | String | Blockers, references, or context |

---

## 5. ID convention

### 5.1 Full format

```
[DOMAIN_CODE]-[ISSUER_CODE]-[SHORT_ID]-[YEAR]
```

### 5.2 Rules

- All uppercase
- Hyphens only as separators
- No spaces, colons, slashes, or special characters
- SHORT_ID: take the standard number; remove spaces, colons, and slashes; use the most recognisable part

### 5.3 ID generation examples

| Standard | zarish_id logic | zarish_id |
|---|---|---|
| ISO 15189:2022 | HL + ISO + 15189 + 2022 | `HL-ISO-15189-2022` |
| IEC 60601-1:2005 | EE + IEC + 60601-1 + 2005 | `EE-IEC-60601-1-2005` |
| RFC 9110 (2022) | ICT + IETF + RFC9110 + 2022 | `ICT-IETF-RFC9110-2022` |
| ILO C87 (1948) | LE + ILO + C87 + 1948 | `LE-ILO-C87-1948` |
| FATF 40 Rec. (2012, rev. 2023) | FB + FATF + 40REC + 2012 | `FB-FATF-40REC-2012` |
| GDPR (2016/679) | CY + EU + GDPR + 2016 | `CY-EU-GDPR-2016` |
| Sphere Handbook 2018 | HM + IASC + SPHERE + 2018 | `HM-IASC-SPHERE-2018` |
| WHO IHR 2005 | HL + WHO + IHR + 2005 | `HL-WHO-IHR-2005` |
| CRC (UN, 1989) | HR + UN + CRC + 1989 | `HR-UN-CRC-1989` |
| ISPM 15 (2002) | PH + IPPC + ISPM15 + 2002 | `PH-IPPC-ISPM15-2002` |

---

## 6. Relationship types

| Relationship | Direction | Meaning |
|---|---|---|
| `supersedes` | A → B | A is the newer standard that replaces B |
| `superseded_by` | A → B | A has been replaced by B |
| `references` | A → B | A formally cites or incorporates B |
| `referenced_by` | A → B | A is formally cited by B |
| `harmonised_with` | A ↔ B | A and B are technically equivalent or aligned |
| `implements` | A → B | A operationalises the requirements of B |
| `national_adoption_of` | A → B | A is a national adoption of international standard B |
| `supplements` | A → B | A provides additional guidance for B |
| `inspired_by` | A → B | A drew significantly from B in its development |
| `treaty_body_of` | A → B | A is the treaty monitoring body established by treaty B |

---

## 7. Enum value registries

### 7.1 entry_type valid values

```
Standard, Framework, Treaty, Guideline, Regulation, Classification,
Code of Practice, Recommendation, Standards Body, Protocol
```

### 7.2 meta_layer valid values

```
L1 Life Sciences, L2 Physical Sciences, L3 Society and Governance,
L4 Economy and Trade, L5 Technology and Infrastructure, L6 Environment
```

### 7.3 governance_layer valid values

```
International, Regional, National
```

### 7.4 status valid values

```
Active, Under Review, Under Development, Superseded, Withdrawn, UNVERIFIED
```

### 7.5 mandate valid values

```
Mandatory, Voluntary, Voluntary-with-regulatory-adoption, Treaty-binding
```

### 7.6 confidence (relationships table) valid values

```
source-confirmed, curator-reviewed, llm-suggested
```

---

## 8. Schema validation rules

The automated schema validation script (`scripts/validate_schema.py`) enforces these rules on every CSV row:

| Rule | Check |
|---|---|
| V01 | `zarish_id` matches regex `^[A-Z]{2,5}-[A-Z]+-[A-Z0-9-]+-[0-9]{4}$` |
| V02 | `entry_type` is one of the 10 valid values |
| V03 | `meta_layer` is one of the 6 valid values |
| V04 | `domain` is one of the 40 canonical domain names |
| V05 | `governance_layer` is one of the 3 valid values |
| V06 | `status` is one of the 6 valid values |
| V07 | `mandate` is one of the 4 valid values |
| V08 | `year_published` is a 4-digit integer between 1850 and 2030, or `UNKNOWN` |
| V09 | `official_url` begins with `https://` |
| V10 | `why_it_matters` is not empty and not `UNKNOWN` (warn if `TODO`) |
| V11 | `data_source` is not empty |
| V12 | No duplicate `zarish_id` values |
| V13 | `name_full` is not empty |
| V14 | `issuer` is not empty |
| V15 | `issuer_type` is one of the valid values |

Validation failures block the `release` Makefile target. Validation warnings are logged but do not block release.

---

## 9. Example entries

Five fully populated entries demonstrating the schema in use.

### Entry 1 — Health Standard

```
zarish_id:          HL-ISO-15189-2022
entry_type:         Standard
meta_layer:         L1 Life Sciences
domain:             Health and Medical
sub_domain:         Clinical Laboratories
name_full:          Medical laboratories — Requirements for quality and competence
name_short:         ISO 15189
standard_id:        ISO 15189:2022
issuer:             ISO (TC 212)
issuer_type:        ISO
governance_layer:   International
geographic_scope:   Global — 175 ISO member countries
year_published:     2022
year_first:         2003
status:             Active
mandate:            Voluntary-with-regulatory-adoption
sector_applicability: Medical laboratories, clinical pathology services, accreditation bodies, national health regulators
why_it_matters:     Defines quality and competence requirements for medical laboratories. Labs without ISO 15189 accreditation cannot participate in national health systems in most countries and cannot provide results that meet international clinical standards. It is the basis for laboratory accreditation in over 100 countries.
key_outputs:        ISO 15189:2022 (third edition); covers pre-examination, examination, and post-examination processes; significant restructuring of management requirements from 2012 edition
official_url:       https://www.iso.org/standard/76677.html
data_source:        ISO Open Data CSV, iso_deliverables_metadata.csv; verified manually 2026-05-01
notes:              Replaced ISO 15189:2012. Key reference for lab accreditation bodies ILAC, DAkkS (Germany), INAB (Ireland), and UKAS (UK).
```

---

### Entry 2 — Treaty

```
zarish_id:          HR-UN-CRC-1989
entry_type:         Treaty
meta_layer:         L3 Society and Governance
domain:             Human Rights
sub_domain:         Children's Rights
name_full:          Convention on the Rights of the Child
name_short:         CRC
standard_id:        UNTS 27531
issuer:             United Nations (OHCHR)
issuer_type:        Treaty Body
governance_layer:   International
geographic_scope:   Global — 196 state parties (as of 2026)
year_published:     1989
year_first:         1989
status:             Active
mandate:            Treaty-binding
sector_applicability: All states parties; all humanitarian programmes affecting children; child protection, education, health, social services sectors
why_it_matters:     The most ratified UN human rights treaty in history, with 196 state parties. It defines the full spectrum of children's rights — civil, political, economic, social, and cultural — and requires states to report regularly to the UN Committee on the Rights of the Child. Every humanitarian programme affecting children must operate within this framework.
key_outputs:        CRC (1989); Optional Protocol on the Sale of Children (2000); Optional Protocol on Children in Armed Conflict (2000); Optional Protocol on a Communications Procedure (2011)
official_url:       https://www.ohchr.org/en/instruments-mechanisms/instruments/convention-rights-child
data_source:        OHCHR website, manual research, 2026-05-01; ratification data from UN Treaty Collection
notes:              USA is the only UN member state that has not ratified the CRC.
```

---

### Entry 3 — Humanitarian Guideline

```
zarish_id:          HM-IASC-SPHERE-2018
entry_type:         Framework
meta_layer:         L3 Society and Governance
domain:             Humanitarian and Emergency Response
sub_domain:         Minimum Standards
name_full:          Sphere Handbook: Humanitarian Charter and Minimum Standards in Humanitarian Response (2018 edition)
name_short:         Sphere Handbook 2018
standard_id:        NONE
issuer:             Sphere Project / IASC
issuer_type:        NGO
governance_layer:   International
geographic_scope:   Global — applicable in all humanitarian response contexts
year_published:     2018
year_first:         2000
status:             Active
mandate:            Voluntary
sector_applicability: All humanitarian organisations; NGOs, UN agencies, INGO, government disaster management agencies
why_it_matters:     The operational reference standard for humanitarian response worldwide. Defines minimum standards in WASH, food security and nutrition, shelter, and health. Any organisation running a humanitarian operation without Sphere knowledge is working without the sector's basic quality benchmark. Widely referenced in donor requirements, cluster guidance, and organisational accountability frameworks.
key_outputs:        Humanitarian Charter; WASH standards; Food Security and Nutrition standards; Shelter and Settlement standards; Health standards; Key indicators for each chapter
official_url:       https://spherestandards.org/humanitarian-standards
data_source:        Sphere website, manual curation, 2026-05-01
notes:              First published as Sphere 2000. Revised 2004, 2011, 2018. Digital edition available free at spherestandards.org.
```

---

### Entry 4 — Financial Standard

```
zarish_id:          FB-FATF-40REC-2012
entry_type:         Framework
meta_layer:         L4 Economy and Trade
domain:             Finance, Banking and Accounting
sub_domain:         Anti-Money Laundering and Counter-Terrorism Financing
name_full:          FATF Recommendations: International Standards on Combating Money Laundering and the Financing of Terrorism and Proliferation
name_short:         FATF 40 Recommendations
standard_id:        NONE
issuer:             Financial Action Task Force (FATF)
issuer_type:        Intergovernmental
governance_layer:   International
geographic_scope:   Global — 206 jurisdictions (FATF members + FATF-Style Regional Bodies)
year_published:     2012
year_first:         1990
status:             Active
mandate:            Voluntary-with-regulatory-adoption
sector_applicability: All financial institutions, designated non-financial businesses and professions (DNFBPs), national financial intelligence units, government financial regulators
why_it_matters:     The global AML/CFT framework adopted by over 200 jurisdictions. Countries not complying with FATF Recommendations risk being placed on the FATF grey or black list, which triggers correspondent banking withdrawal, trade finance restrictions, and credit rating impacts. For any NGO or financial institution operating internationally, FATF compliance is a non-negotiable operational requirement.
key_outputs:        40 Recommendations; Interpretive Notes to each Recommendation; FATF Guidance documents; Mutual Evaluation methodology
official_url:       https://www.fatf-gafi.org/en/topics/fatf-recommendations.html
data_source:        FATF website, manual research, 2026-05-01
notes:              Originally published as 40 Recommendations (1990). Major revision 2003. Current version 2012, with amendments through 2023. Bangladesh assessed under FATF/APG mutual evaluation framework.
```

---

### Entry 5 — Sustainability Standard

```
zarish_id:          SE-GRI-GRI1-2021
entry_type:         Standard
meta_layer:         L4 Economy and Trade
domain:             Sustainability, ESG and Circular Economy
sub_domain:         Sustainability Reporting
name_full:          GRI 1: Foundation 2021
name_short:         GRI 1
standard_id:        GRI 1
issuer:             Global Reporting Initiative (GRI)
issuer_type:        NGO
governance_layer:   International
geographic_scope:   Global; mandatory for EU companies via CSRD/ESRS alignment
year_published:     2021
year_first:         2021
status:             Active
mandate:            Voluntary-with-regulatory-adoption
sector_applicability: All organisations reporting on environmental, social, and governance impacts; mandatory for large EU-based companies under CSRD
why_it_matters:     The foundation of GRI's sustainability reporting system, used by 73% of the world's 250 largest companies. It establishes the concepts, requirements, and guidance that underpin all GRI Standards. Any organisation beginning to report on ESG impact must start here. The EU's Corporate Sustainability Reporting Directive (CSRD) is aligned with GRI, making GRI 1 effectively mandatory for thousands of European companies.
key_outputs:        GRI 1 Foundation; links to GRI 2 (General Disclosures 2021), GRI 3 (Material Topics 2021), and all topic and sector standards
official_url:       https://www.globalreporting.org/standards/standards-development/universal-standards/
data_source:        GRI website, manual curation, 2026-05-01
notes:              Replaced GRI Sustainability Reporting Standards 2016 (GRI 101). The GRI Standards Download Center provides all standards free of charge.
```

---

## 10. Cross-references

- → **004-zs-index/001-zarish-index-overview.md** — Project charter, scope, and entry-count targets
- → **004-zs-index/002-domain-taxonomy-40.md** — Canonical 40-domain taxonomy that the domain field must match
- → **004-zs-index/004-harvesting-policy.md** — Harvesting policy with a condensed schema summary; this document governs where they differ
- → **004-zs-index/005-zarish-index-to-platform.md** — The fields the G2A Engine consumes from this schema
- → **005-zs-standards/002-transformation-model.md** — INDEX-to-STANDARDS pipeline that maps ZARISH-INDEX entries into executable definitions

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
