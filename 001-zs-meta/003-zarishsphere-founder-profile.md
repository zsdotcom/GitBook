# 003-zarishsphere-founder-profile.md
## Mohammad Ariful Islam — complete context profile
### ZarishSphere Foundation, founder and sole builder

**Document type:** Reference
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative profile context for all AI-assisted work

---

## Table of contents

1. [Identity and contact architecture](#1-identity-and-contact-architecture)
2. [Professional background](#2-professional-background)
3. [Education and certifications](#3-education-and-certifications)
4. [Current working environment](#4-current-working-environment)
5. [ZarishSphere Foundation project context](#5-zarishsphere-foundation-project-context)
6. [Working style and AI interaction preferences](#6-working-style-and-ai-interaction-preferences)

---

## 1. Identity and contact architecture

| Field | Value |
|---|---|
| Full name | Mohammad Ariful Islam |
| Preferred alias | Ariful / Md Ariful Islam |
| Nationality | Bangladeshi |
| Current location | Cox's Bazar — 4700, Bangladesh |
| Role (Primary) | Founder, ZarishSphere Foundation |
| Educational Background | Public Health Professional |

### 1.1 Digital identity

| Platform | Handle / URL |
|---|---|
| LinkedIn | https://www.linkedin.com/in/arifulislambgd |
| GitHub personal | https://github.com/codeandbraain |
| GitHub Org | https://github.com/zsdotcom |
| Website | https://zarishsphere.com |
| GitHub Personal2 | https://github.com/arwazarish |

---

## 2. Professional background

### 2.1 Career summary

Mohammad Ariful Islam is a humanitarian leader and public health professional with over 15 years of experience bridging the gap between clinical excellence, academic rigor, and large-scale humanitarian response. Track record in orchestrating multi-sectoral interventions — spanning Health, Nutrition, WASH, Education, and Protection — in highly volatile, resource-constrained environments. Transitioned from foundational clinical practice and medical education to strategic leadership, culminating in the management of complex global health clusters. Adept at navigating diverse socio-political landscapes, managing multidisciplinary teams, and driving evidence-based, sustainable development outcomes.

Recently, Ariful has transitioned from a field practitioner to a systems architect. His approach is rooted in the reality that the distance between high-level policy (e.g., WHO guidelines) and the working tools available to health workers is a structural failure that causes real-world harm. His work is driven by the conviction that humanitarian digital infrastructure should be "not just software," but foundational technology for civilization.

He is a **practitioner-turned-builder**: his platform design decisions are not academic — they come from years of watching systems fail in the field.

### 2.2 Career evaluation

- Global Humanitarian Leadership: Directing large-scale health clusters, coordinating humanitarian response strategies across multiple crisis zones.
- Public Health Implementation: Program leadership in resource-constrained environments, focusing on infectious disease control, maternal/child health, and health system resilience.
- Clinical Practice: Delivering frontline care in diverse clinical settings, managing acute and chronic health conditions.
- Academic Tenure: Served as a Lecturer/Professor in Medical Schools, responsible for clinical training, research, and fostering professional standards in clinical practice.

### 2.3 Core professional expertise

- Strategic Planning & Program Management: End-to-end management from needs assessment to monitoring and evaluation (M&E).
- Data-Driven Decision Making: Expert in leveraging epidemiological data for rapid response and resource allocation.
- Crisis Management & Security: Navigating high-risk environments with strict adherence to humanitarian principles and security protocols.
- Interpersonal & Soft Skills: Diplomacy, conflict resolution, cultural intelligence, and high-stakes negotiation.
- Program design, cycle management, and quality-focused implementation
- MEAL (Monitoring, Evaluation, Accountability, and Learning)
- Grant management, financial oversight, budget preparation, and donor compliance
- Partnership management: due diligence, capacity assessment, training
- Government liaison and stakeholder coordination (MoH, NGO Affairs Bureau)
- Procurement management and medical warehouse operations
- Staff supervision, training needs assessment, performance appraisal
- SOP design, quality assurance, and field oversight
- Multilingual leadership (English, Bangla, Rohingya context awareness)

### 2.4 Technical profile stack

| Domain | Knowledge Level |
|---|---|
| Public health program management | Expert |
| FHIR R5 / HL7 health standards | Intermediate (building toward advanced) |
| GitHub / git workflows | Intermediate |
| Docker / containerisation | Beginner (learning via ZarishSphere) |
| Markdown documentation | Proficient |
| No-code / GUI tools | Preferred primary interface |
| Terminal / CLI | Basic — always needs copy-paste exact commands |
| Go / Python / YAML | Reading-level, not writing-level |

---

## 3. Education and certifications

| Qualification | Institution | Year |
|---|---|---|
| Masters of Public Health (MPH) | Bangladesh Open University (University of South Asia listed on CV) | 2023 |
| Post Graduation Diploma in Management (PGDM) — HR | Bangladesh Open University / School of Business | 2023 |
| Health Financing | SSPH+ Summer School, Lugano | 2023 |
| Wealth Inequalities | SSPH+ Summer School, Lugano | 2023 |
| Project Management in Global Health | University of Washington | 2022 |
| Global Mental Health | University of Washington | 2021 |
| Bachelor of Public Health Nutrition | Primeasia University | 2018 |
| Diploma in Medical Faculty | State Medical Faculty | 2013 |

### 3.1 Key professional training

- Leadership & Management in Health — University of Washington (2023)
- Health Finance & Wealth Inequalities — Swiss School of Public Health (2023)
- International Humanitarian Law (IHL) — Humanitarian Leadership Academy (2021)
- Health Cluster Coordination — WHO (2020)
- BLS with CPR Certification — First Aid Centre of Sweden (2020)
- Health Facility Design — WHO (2020)
- Building a Better Response — Humanitarian Academy at Harvard (2020)
- Gender Equality in Humanitarian Action — IASC (2020)
- Public Health Engineering — MSF (2019)
- Field Management Course — MSF (2018)
- Mass Casualty Management — MSF (2018)

---

## 4. Current working environment

### 4.1 Hardware

| Item | Specification |
|---|---|
| Machine | Lenovo laptop |
| CPU | Intel Core i3 |
| RAM | 8 GB |
| OS | Ubuntu 26.04 LTS |
| Context | Primary development machine — sole build environment |

> **Critical constraint:** No Java-based tools (HAPI FHIR, etc.) — RAM constraint is absolute. Go-native, lightweight binaries only.

### 4.2 Network context

- Primary internet: Bangladeshi mobile broadband (variable quality)
- Must account for intermittent connectivity
- All tools must work offline or degrade gracefully

### 4.3 Infrastructure snapshot — August 2026

| Asset | Status |
|---|---|
| Domain: zarishsphere.com | Active — Cloudflare DNS/CDN/Pages |
| GitHub Org: zarishsphere | Active |
| GitHub Personal: arwazarish | Active |
| Platform email routing | Cloudflare Email Routing (all → zarishsphere@gmail.com) |
| Deployment status | V1 development — not yet live |

---

## 5. ZarishSphere Foundation project context

### 5.1 What ZarishSphere is

ZarishSphere is an **open-source, omni-domain Platform of Platforms** — a universal Digital Public Infrastructure (DPI) designed to bridge the gap between international policy (WHO protocols, UNHCR standards, ISO frameworks) and field execution. Its primary innovation is the **Guideline-to-Action (G2A) Engine**: the world's first open-source system that automatically converts any international standard into deployable digital assets.

**Primary deployment context:** Cox's Bazar, Bangladesh — Rohingya FDMN population + Bangladeshi host community.

**Scale intent:** Global from Day 1. Architecture, documentation, and all configurations are built for multi-country deployment from the first commit.

### 5.2 Three defining principles

| Principle | Plain statement |
|---|---|
| Guideline-as-Code | Every WHO protocol, SPHERE standard, ISO framework is machine-readable and auto-executable. No standard lives only as a PDF. |
| Identity Without Surveillance | Serve individuals for clinical continuity while making their surveillance technically impossible. Identical data has emergency key-destruction protection. |
| GitHub as Government | Every form, protocol, deployment, and governance decision is a git commit. Clinical governance happens through PR review. |

### 5.3 Current stage

- V1 development — starting from zero
- No team — Ariful is the sole builder
- No budget — all tools must be zero-cost with free tiers (no trials, no expiring freemium)
- Not yet live — every action is building toward first real deployment

### 5.4 Three active projects

| Project | Codename | Type | License |
|---|---|---|---|
| Standards index (40+ domains, 88K+ entries) | ZARISH-INDEX | Autonomous open research | CC BY 4.0 |
| 40-domain standards registry & hierarchy | ZARISH-STANDARDS | Standards classification system | CC BY 4.0 |
| Digital Public Infrastructure platform | ZarishSphere | Platform of Platforms | Apache 2.0 (code) · CC BY 4.0 (docs) |

### 5.5 Non-negotiables

1. **Zero cost** — No trial. No freemium that expires. Zero budget is absolute.
2. **No-code first** — Every setup must be achievable via GUI. CLI is secondary, always copy-paste.
3. **Vendor lock-in freedom** — No single vendor dependency. Data exportable in open standards at all times.
4. **V1 until launch** — No versioning until complete platform launch.
5. **Global from Day 1** — All architecture and documentation designed for global deployment from the first day.
6. **No HAPI FHIR (Java)** — RAM constraint. Go-native FHIR server only (50–150MB vs 2–3GB for HAPI).

---

## 6. Working style and AI interaction preferences

### 6.1 Tone

- Semi-formal to direct: knowledgeable but not condescending
- No excessive pleasantries, no apologies, no padding
- Treat Ariful as a domain expert in public health, not a beginner

### 6.2 Technical instructions

- Always GUI-first, then CLI as fallback
- CLI commands: exact, copy-paste ready, step-by-step, numbered
- Assume zero prior knowledge for each new technical task
- Never assume a development team exists

### 6.3 Output format

- Primary outputs: Markdown files (downloadable)
- Technical configs: YAML / JSON / Go code (exact, version-pinned, never `latest` tags)
- Documentation: structured headers, numbered sections, tables for comparisons
- Naming: ZUSS standard (see → **004-zarishsphere-writing-rules.md**)

### 6.4 What agents must never assume

- That Ariful has a development team
- That Ariful has any budget
- That previous references to i5-10400 / 16GB desktop are current (current machine: Lenovo i3 / 8GB / Ubuntu 26.04)
- That ZarishSphere is already live
- That HAPI FHIR is acceptable (it is not — RAM constraint)
- That any tool requires payment

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
