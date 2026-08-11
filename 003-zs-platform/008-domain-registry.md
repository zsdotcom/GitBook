# 008-domain-registry.md
## Domain registry
### 40-domain classification taxonomy

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose of the domain registry](#1-purpose-of-the-domain-registry)
2. [The 40 domains](#2-the-40-domains)
3. [Domain prefix conventions](#3-domain-prefix-conventions)
4. [Domain expansion](#4-domain-expansion)
5. [Cross-references](#5-cross-references)

---

## 1. Purpose of the domain registry

The domain registry is the master classification system for the entire ecosystem. Every ZARISH-INDEX entry, every domain module, and every ecosystem component maps to exactly one domain. The registry ensures consistent naming, routing, and cross-referencing across all projects.

## 2. The 40 domains

| # | Code | Domain | ZI prefix | Module code | Examples |
|---|---|---|---|---|---|
| 1 | HEALTH | Health | ZI-HEALTH- | ZS-MOD-HEALTH | Clinical guidelines, FHIR, WHO protocols |
| 2 | NUTRITION | Nutrition | ZI-NUTRITION- | ZS-MOD-NUTRITION | Food security, supplementation, growth monitoring |
| 3 | WASH | Water, sanitation, hygiene | ZI-WASH- | ZS-MOD-WASH | Water quality, sanitation facilities, hygiene promotion |
| 4 | PROTECTION | Protection | ZI-PROTECTION- | ZS-MOD-PROTECTION | Child protection, GBV, human rights |
| 5 | EDUCATION | Education | ZI-EDUCATION- | ZS-MOD-EDUCATION | Formal, non-formal, early childhood, vocational |
| 6 | LOGISTICS | Logistics | ZI-LOGISTICS- | ZS-MOD-LOGISTICS | Supply chain, fleet, warehousing |
| 7 | SHELTER | Shelter | ZI-SHELTER- | ZS-MOD-SHELTER | Emergency shelter, housing, infrastructure |
| 8 | CAMP | Camp management | ZI-CAMP- | ZS-MOD-CAMP | Site planning, population tracking |
| 9 | FINANCE | Finance | ZI-FINANCE- | ZS-MOD-FINANCE | Budgeting, accounting, CVA |
| 10 | HR | Human resources | ZI-HR- | ZS-MOD-HR | Personnel, training, capacity building |
| 11 | GOVERNANCE | Governance | ZI-GOVERNANCE- | ZS-MOD-GOVERNANCE | Civil administration, public services |
| 12 | JUSTICE | Justice | ZI-JUSTICE- | ZS-MOD-JUSTICE | Courts, legal aid, legal identity |
| 13 | ENVIRONMENT | Environment | ZI-ENVIRONMENT- | ZS-MOD-ENVIRONMENT | Climate, biodiversity, pollution, natural resources |
| 14 | ENERGY | Energy | ZI-ENERGY- | ZS-MOD-ENERGY | Electricity, renewables, fuel |
| 15 | AGRICULTURE | Agriculture | ZI-AGRICULTURE- | ZS-MOD-AGRICULTURE | Farming, livestock, food production |
| 16 | WATER | Water resources | ZI-WATER- | ZS-MOD-WATER | Water resource management, irrigation |
| 17 | TELECOM | Telecommunications | ZI-TELECOM- | ZS-MOD-TELECOM | Connectivity, telecom regulation |
| 18 | TRANSPORT | Transport | ZI-TRANSPORT- | ZS-MOD-TRANSPORT | Roads, transit, mobility |
| 19 | TRADE | Trade | ZI-TRADE- | ZS-MOD-TRADE | International trade, customs |
| 20 | LABOUR | Labour | ZI-LABOUR- | ZS-MOD-LABOUR | Employment, workers' rights |
| 21 | SOCIAL | Social protection | ZI-SOCIAL- | ZS-MOD-SOCIAL | Social safety nets, welfare |
| 22 | HOUSING | Housing | ZI-HOUSING- | ZS-MOD-HOUSING | Urban development, land rights |
| 23 | CULTURE | Culture | ZI-CULTURE- | ZS-MOD-CULTURE | Heritage, arts, cultural preservation |
| 24 | SPORT | Sport | ZI-SPORT- | ZS-MOD-SPORT | Physical education, sports governance |
| 25 | SCIENCE | Science | ZI-SCIENCE- | ZS-MOD-SCIENCE | Research, scientific standards |
| 26 | TECHNOLOGY | Technology | ZI-TECHNOLOGY- | ZS-MOD-TECHNOLOGY | ICT, AI, cybersecurity, standards |
| 27 | CONSTRUCTION | Construction | ZI-CONSTRUCTION- | ZS-MOD-CONSTRUCTION | Building codes, engineering |
| 28 | MANUFACTURING | Manufacturing | ZI-MANUFACTURING- | ZS-MOD-MANUFACTURING | Industrial standards, production |
| 29 | HUMANITARIAN | Humanitarian | ZI-HUMANITARIAN- | ZS-MOD-HUMANITARIAN | SPRHERE, humanitarian principles, cluster system |
| 30 | HUMAN_RIGHTS | Human rights | ZI-HUMAN_RIGHTS- | ZS-MOD-HUMAN_RIGHTS | Treaties, declarations, monitoring |
| 31 | GENDER | Gender | ZI-GENDER- | ZS-MOD-GENDER | Gender equality, GBV prevention |
| 32 | DISABILITY | Disability | ZI-DISABILITY- | ZS-MOD-DISABILITY | Inclusion, accessibility |
| 33 | MIGRATION | Migration | ZI-MIGRATION- | ZS-MOD-MIGRATION | Refugee protection, IDP rights |
| 34 | INDIGENOUS | Indigenous peoples | ZI-INDIGENOUS- | ZS-MOD-INDIGENOUS | Indigenous rights, traditional knowledge |
| 35 | CHILDREN | Children | ZI-CHILDREN- | ZS-MOD-CHILDREN | Child rights, child protection |
| 36 | YOUTH | Youth | ZI-YOUTH- | ZS-MOD-YOUTH | Youth development, participation |
| 37 | ELDERLY | Elderly | ZI-ELDERLY- | ZS-MOD-ELDERLY | Ageing, elder care |
| 38 | FOOD | Food | ZI-FOOD- | ZS-MOD-FOOD | Food safety, food standards |
| 39 | ANIMAL | Animal health | ZI-ANIMAL- | ZS-MOD-ANIMAL | Veterinary, animal welfare, zoonotic diseases |
| 40 | DATA | Data | ZI-DATA- | ZS-MOD-DATA | Data standards, metadata, statistics |

## 3. Domain prefix conventions

| Prefix type | Format | Example |
|---|---|---|
| ZARISH-INDEX | `ZI-[CODE]-` | `ZI-HEALTH-00001` |
| Module code | `ZS-MOD-[CODE]` | `ZS-MOD-HEALTH` |
| Domain code | Uppercase, underscore for spaces | `HUMAN_RIGHTS` |
| File path | Lowercase, hyphens | `human-rights/` |

## 4. Domain expansion

The initial 40 domains serve as the foundation. As the ecosystem grows, domains may be:

- **Split** — a domain becomes too large and is divided (e.g., HEALTH → CLINICAL, PUBLIC_HEALTH)
- **Merged** — two closely related domains are combined
- **Added** — new domains are added as the ecosystem covers more of human activity

All domain registry changes require an ADR in `008-zs-adrs/`.

The architecture supports 100+ domains. The 40-domain startup is the minimum viable scope for a globally representative index.

## 5. Cross-references

→ **004-zs-index/002-domain-taxonomy-40.md** — The same 40-domain taxonomy from the ZARISH-INDEX project side
→ **003-zs-platform/002-module-architecture.md** — Module structure and lifecycle every domain module follows
→ **003-zs-platform/007-data-model.md** — ZS-UID MODULE codes derived from this registry
→ **003-zs-platform/009-repository-catalog.md** — Domain module repositories (`zs-modules-*`) organised by these domains
→ **001-zs-meta/005-zarishsphere-ecosystem-architecture.md** — Ecosystem master map and navigation rules
→ **008-zs-adrs/001-adr-go-as-primary-language.md** — ADR-001: Go as the primary language for domain modules

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
