# 005-country-adoption-model.md
## Country Adoption Maturity Model (CAMM)
### Structured pathway from first contact to full digital health sovereignty

**Document type:** Normative standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Draft
**GitHub:** https://github.com/zsdotcom
**Depends on:** `002-zs-foundation/001-foundation-charter.md`, `002-zs-foundation/002-governance-model.md`

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Model overview](#2-model-overview)
3. [Level 0 — Interest](#3-level-0--interest)
4. [Level 1 — Foundation](#4-level-1--foundation)
5. [Level 2 — Adoption](#5-level-2--adoption)
6. [Level 3 — Integration](#6-level-3--integration)
7. [Level 4 — Maturity](#7-level-4--maturity)
8. [Level 5 — Sovereignty](#8-level-5--sovereignty)
9. [Assessment process](#9-assessment-process)
10. [Prerequisites for entry](#10-prerequisites-for-entry)
11. [Templates and tools](#11-templates-and-tools)
12. [Cross-references](#12-cross-references)

---

## 1. Purpose

The Country Adoption Maturity Model (CAMM) defines the structured pathway that guides partner countries, programmes, and organisations from first contact with the ZarishSphere ecosystem through to full digital health sovereignty. CAMM prevents the common failure mode of "big bang" health IT implementation by breaking adoption into six incremental, verifiable levels.

### 1.1 Why CAMM exists

Without a structured adoption model:

- Countries become overwhelmed by the breadth of the platform and abandon implementation
- Resources are wasted training users on features the programme is not ready for
- The Foundation cannot prioritise support effectively across multiple partners
- Donor reporting lacks a common framework for measuring progress

CAMM provides all parties — country teams, donors, implementing partners, and the Foundation — with a shared language for describing and measuring progress.

### 1.2 How CAMM relates to the Foundation

CAMM is a governance instrument of the ZarishSphere Foundation. It flows from the Foundation's mission to make global standards executable at zero cost, and it supports the no-vendor-lock-in principle (ADR-009) by ensuring every country builds the internal capability to operate independently.

> **Constraint:** No country may be required to use CAMM to adopt ZarishSphere technology. CAMM is a guidance framework, not a gatekeeping mechanism. Any country may deploy the platform at any level without Foundation approval.

---

## 2. Model overview

CAMM defines six levels. Each level has:

- **Entry gates** — conditions that must be true before work at this level begins
- **Activities** — the work to be completed during the level
- **Exit criteria** — verifiable conditions that must be met before advancing
- **Deliverables** — concrete artifacts produced during the level

### 2.1 Level progression diagram

```
L0: Interest ──► L1: Foundation ──► L2: Adoption ──► L3: Integration ──► L4: Maturity ──► L5: Sovereignty
    MOU signed       1 facility live   10+ facilities   MOH bridges live    National rollout   Self-governing
    1–3 months       3–6 months        6–12 months      12–18 months        18–36 months       Ongoing
```

### 2.2 Level summary table

| Level | Name | Duration | Scope | Key outcome |
|---|---|---|---|---|
| L0 | Interest | 1–3 months | Assessment | Signed MOU, country repos created |
| L1 | Foundation | 3–6 months | 1 facility | Pilot live with real patient data |
| L2 | Adoption | 6–12 months | 10+ facilities | DHIS2 reporting, local capacity |
| L3 | Integration | 12–18 months | National bridges | MOH systems connected, data residency |
| L4 | Maturity | 18–36 months | National rollout | All modules active, EWARS, automated reports |
| L5 | Sovereignty | Ongoing | Self-governance | Independent operations, community contribution |

### 2.3 Deployment plane mapping

Each CAMM level maps onto the Platform deployment planes defined in → **003-zs-platform/003-deployment-planes.md** — five infrastructure planes, from air-gapped to global SaaS:

| CAMM level | Typical deployment plane | Infrastructure |
|---|---|---|
| L0–L1 | Plane 1 (Edge) | Raspberry Pi 5, 8 GB RAM, Docker Compose |
| L2 | Plane 2 (District) | Single server, 16 GB RAM, K3s |
| L3 | Plane 3 (National) | Multi-node cluster, 64+ GB RAM |
| L4–L5 | Plane 3/4 (National/SaaS) | Full K8s, CI/CD, monitoring |

---

## 3. Level 0 — Interest

**Typical duration:** 1–3 months

### 3.1 What this level is about

At L0, a country, programme, or organisation has discovered ZarishSphere and is evaluating it. The goal is to move from initial awareness to a signed commitment with a clear plan.

**Who this is for:** Ministries of Health, NGOs, UN agencies, or donor-funded programmes in the early evaluation phase.

### 3.2 Entry gates

No prerequisites — any organisation can start at L0. No financial commitment is required or requested.

### 3.3 Activities

1. **Initial outreach and discovery**
   - Introductory call with the Foundation
   - Share overview materials and the CAMM framework itself
   - Review the adoption journey and set expectations

2. **Needs assessment**
   - Complete a structured questionnaire covering:
     - Number of health facilities and workers in scope
     - Current health information systems in use
     - Connectivity situation (internet, power reliability)
     - Device types available (Android, tablets, computers)
     - Key clinical domains required
     - Staff capacity and IT capability
     - Data sovereignty requirements
   - See → **[Needs Assessment template](#11-templates-and-tools)**

3. **Technical readiness assessment**
   - Evaluate hosting options (cloud, on-premise, edge device)
   - Assess network infrastructure and device compatibility
   - Confirm minimum hardware requirements for chosen deployment plane
   - See → **[Technical Readiness template](#11-templates-and-tools)**

4. **Memorandum of Understanding (MOU) signing**
   - Both parties sign the MOU template
   - Foundation commits: platform access, documentation, community support
   - Country commits: designated focal point, participation in community, sharing back
   - **No financial obligation** is created by the MOU
   - See → **[MOU template](#11-templates-and-tools)**

### 3.4 Exit criteria

To advance from L0 to L1:

- [ ] Needs assessment completed and documented
- [ ] Technical readiness assessment completed
- [ ] MOU signed by authorised representative of both parties
- [ ] Country distribution repository created (e.g., `zs-distro-{cc}`)
- [ ] Country infrastructure repository created (e.g., `zs-infra-{cc}`)
- [ ] Designated country focal point added as GitHub organisation member
- [ ] Country-specific form requirements documented

### 3.5 Deliverables

| Deliverable | Location |
|---|---|
| Completed needs assessment | `zs-docs-camm/countries/{CC}-STATUS.md` |
| Signed MOU | `zs-docs-donor/grants/` (private copy) |
| Country distribution repo | `github.com/zarishsphere/zs-distro-{cc}` |
| Country infrastructure repo | `github.com/zarishsphere/zs-infra-{cc}` |

---

## 4. Level 1 — Foundation

**Typical duration:** 3–6 months

### 4.1 What this level is about

L1 is proof of concept with real patients. At least one real health facility runs ZarishSphere with real clinical data. The system works. Staff are trained. Core clinical modules are live.

**Who this is for:** Programmes that have completed initial assessment and have leadership buy-in.

### 4.2 Entry gates

- L0 exit criteria all met
- MOU signed
- Country distribution and infrastructure repos exist

### 4.3 Activities

1. **Platform setup**
   - Fork infrastructure template to `zs-infra-{cc}`
   - Fork distribution template to `zs-distro-{cc}`
   - Deploy ZarishSphere to first facility (Raspberry Pi 5, cloud VM, or hosted)
   - Configure identity provider with facility users and roles
   - Load core clinical forms

2. **Core clinical configuration**
   - Configure patient registration with local ID types
   - Add facility to the FHIR Location registry
   - Set up user accounts for all clinical staff
   - Configure offline-capable notification channel (SMS if available)

3. **Staff training**
   - Complete L1 training track (approximately 8 hours per staff member):
     - ZarishSphere introduction (30 min)
     - Patient registration (2 hours)
     - Encounter and vitals entry (2 hours)
     - Clinical forms (2 hours)
     - Data quality and audit (1 hour)
     - Troubleshooting basics (30 min)
   - See → **[Training Curriculum template](#11-templates-and-tools)**

4. **Pilot go-live**
   - Go live with first facility
   - Foundation provides weekly support for the first month
   - Collect structured feedback from clinical staff

5. **Form localisation** (if needed)
   - Identify gaps in core clinical forms
   - Create country-specific forms using the no-code form builder
   - Submit forms to the content repository via pull request
   - Add country-specific language translations

### 4.4 Exit criteria

To advance from L1 to L2:

- [ ] At least 1 facility live with real patient data for 30+ days
- [ ] Minimum 50 patient encounters recorded in production
- [ ] All clinical staff at pilot facility trained
- [ ] Data quality audit completed — >90% complete records
- [ ] Country focal point can create or modify forms without Foundation assistance
- [ ] Country-specific language translations added (if not English)
- [ ] Retrospective conducted with facility staff
- [ ] L1 report submitted to the Foundation

### 4.5 Deliverables

| Deliverable | Location |
|---|---|
| Pilot facility live in production | `zs-infra-{cc}` deployed |
| Country distribution with local forms | `zs-distro-{cc}` |
| Staff training completion records | `zs-docs-camm/countries/{CC}-STATUS.md` |
| L1 retrospective report | `zs-docs-donor/reports/` |

---

## 5. Level 2 — Adoption

**Typical duration:** 6–12 months

### 5.1 What this level is about

L2 is expansion. The pilot succeeded. The programme scales to 10+ facilities, connects to national reporting systems, and builds local capacity to manage the platform independently.

**Who this is for:** Programmes expanding beyond the pilot, typically with government co-ownership.

### 5.2 Entry gates

- L1 exit criteria all met
- Production system stable for 60 consecutive days

### 5.3 Activities

1. **Multi-facility rollout**
   - Scale from 1 to 10+ facilities using site deployment templates
   - Each new facility should take 1–2 days to onboard (versus weeks at L1)
   - Local IT staff manages facility onboarding using the template

2. **DHIS2 connection**
   - Configure DHIS2 bridge for national HMIS reporting
   - Map ZarishSphere indicators to DHIS2 data elements
   - Establish automated weekly aggregate reporting

3. **Local capacity building**
   - Train 2 country-level "super users" to manage the platform
   - Train facility IT staff on basic troubleshooting
   - Establish a local helpdesk process
   - Complete L2 training track (approximately 16 hours per super user)

4. **Country-specific form completion**
   - Complete all forms required for national reporting
   - All forms validated against ZS Form Schema v1
   - All required language translations complete and verified

5. **Monitoring and alerting**
   - Configure monitoring dashboards for the country programme
   - Set up threshold alerts for communicable disease surveillance
   - Establish weekly automated indicator reports to programme management

### 5.4 Exit criteria

To advance from L2 to L3:

- [ ] 10+ facilities live with regular clinical data entry
- [ ] DHIS2 aggregate reporting automated
- [ ] 2 trained country super users certified
- [ ] All required programme forms deployed and in active use
- [ ] Country can onboard a new facility without Foundation core team involvement
- [ ] 3 months of stable operations without critical incidents

### 5.5 Deliverables

| Deliverable | Location |
|---|---|
| Multi-facility infrastructure state | `zs-infra-{cc}` |
| DHIS2 bridge configuration | `zs-distro-{cc}/config/dhis2/` |
| Country super user certification records | `zs-docs-camm/countries/{CC}-STATUS.md` |
| Quarterly impact report (L2) | `zs-docs-donor/reports/` |

---

## 6. Level 3 — Integration

**Typical duration:** 12–18 months

### 6.1 What this level is about

L3 is national system integration. ZarishSphere connects to the national health information infrastructure: DHIS2 (bidirectional), laboratory systems, national registries. The Ministry of Health is an active partner. National codes and terminologies are loaded.

**Who this is for:** Programmes with full government endorsement and national system connectivity.

### 6.2 Entry gates

- L2 exit criteria all met
- Ministry of Health endorsement obtained

### 6.3 Activities

1. **National system bridges**
   - **DHIS2 bidirectional:** aggregate plus tracker sync
   - **HL7v2 lab results:** configure bridge for national laboratory systems
   - **National drug codes:** load into local terminology service with mappings
   - **Country-specific:** ABDM (India), SGDB (Bangladesh), or equivalent integration

2. **National terminology alignment**
   - Load national facility registry into the platform
   - Map local diagnosis codes to ICD-11 in concept maps
   - Add national laboratory panels to LOINC mappings
   - See → **005-zs-standards/016-terminology-governance.md** — Terminology code system governance

3. **Government data governance**
   - Establish data sharing agreement with Ministry of Health
   - Configure data residency — country data stays in country
   - Implement field-level encryption for protected health information at rest
   - See → **006-zs-infrastructure/009-compliance-controls.md** — HIPAA and GDPR compliance controls

4. **National indicator alignment**
   - Map ZarishSphere indicators to national KPIs
   - Automate Ministry of Health monthly and quarterly reports
   - Configure national threshold settings for early warning surveillance

### 6.4 Exit criteria

To advance from L3 to L4:

- [ ] DHIS2 bidirectional sync live and verified
- [ ] At least one national system integrated (lab, pharmacy, or registry)
- [ ] National facility registry loaded and mapped
- [ ] Country data stored in-country — data residency verified
- [ ] Ministry of Health signs off on data quality
- [ ] 6 months of stable multi-system operations without data loss incidents

### 6.5 Deliverables

| Deliverable | Location |
|---|---|
| Integration configurations | `zs-distro-{cc}/config/` |
| National system mapping documentation | `zs-docs-standards/` |
| Data residency verification report | `zs-docs-security/compliance/` |

---

## 7. Level 4 — Maturity

**Typical duration:** 18–36 months

### 7.1 What this level is about

L4 is national rollout. ZarishSphere is the national digital health platform for primary care. All clinical modules are active. Public health surveillance feeds the national early warning system. Donor reporting is fully automated.

**Who this is for:** National programmes with full deployment and active use across the health system.

### 7.2 Entry gates

- L3 exit criteria all met
- National budget allocated for platform operations

### 7.3 Activities

1. **Full clinical module deployment**
   All clinical modules active:
   - Patient management, encounter, observation, medication
   - Immunisation, condition, procedure, allergy
   - Appointment, document, care plan, consent
   - Nutrition, maternity, child health, mental health
   - Communicable disease, laboratory, pharmacy, radiology

2. **Public health surveillance**
   - 24/7 automated outbreak detection via early warning system
   - Real-time disease surveillance dashboard for national disease control
   - Automatic outbreak notification to Ministry of Health
   - Syndromic surveillance running on anonymised encounter data

3. **Administrative and ERP modules**
   - HRM: staff records, leave management, performance tracking
   - Inventory: medical supply chain management
   - Finance: donor fund tracking and expenditure reporting
   - Procurement: purchase orders and vendor management

4. **Quality improvement**
   - Monthly clinical audit workflows
   - Indicator performance dashboards for each facility
   - Anonymous peer facility comparison

5. **Business continuity**
   - Document and test the business continuity plan
   - Establish backup and disaster recovery procedures
   - See → **006-zs-infrastructure/007-security-architecture.md** — Defence-in-depth security architecture

### 7.4 Exit criteria

To advance from L4 to L5:

- [ ] >80% of target facilities live and submitting data
- [ ] All clinical domains active across the deployment
- [ ] Early warning system fully automated
- [ ] National donor reports fully automated
- [ ] Country team trained in basic platform operations
- [ ] Business continuity plan documented and tested
- [ ] No critical security incidents in the preceding 12 months

### 7.5 Deliverables

| Deliverable | Location |
|---|---|
| Full national deployment state | `zs-infra-{cc}` |
| Automated donor report configuration | `zs-docs-donor/reports/` |
| Business continuity plan | `zs-docs-runbooks/` |
| Security audit report | `zs-docs-security/audits/` |

---

## 8. Level 5 — Sovereignty

**Duration:** Ongoing

### 8.1 What this level is about

L5 is the ultimate goal. The country is fully self-sufficient. It manages its own infrastructure, trains its own staff, handles its own incidents, and contributes back to the ZarishSphere community.

**Who this is for:** Mature national programmes that have fully internalised the platform and operate independently.

### 8.2 Entry gates

- L4 exit criteria all met
- Country has in-house DevOps capability
- Country has been managing its own operations for 6+ consecutive months

### 8.3 What sovereignty means

#### 8.3.1 Technical sovereignty
- Country owns and manages its own Kubernetes cluster
- Country manages its own database, messaging, cache, and identity infrastructure
- Country runs its own CI/CD pipeline
- Country can deploy updates without Foundation assistance

#### 8.3.2 Governance sovereignty
- Country has permanent representation in ZarishSphere governance bodies
- Country votes on RFCs that affect their programme
- Country holds a GitHub organisation owner credential for their distribution repos

#### 8.3.3 Content sovereignty
- Country maintains its own clinical forms repository
- Country manages its own translation files independently
- Country contributes forms and protocols back to the global content repositories

#### 8.3.4 Community contribution
- Country staff present at global ZarishSphere community events
- Country contributes at least 1 clinical form or protocol per quarter to the shared community
- Country mentors new programmes reaching L0 and L1

### 8.4 Ongoing responsibilities

- Keep the platform updated — automated dependency management
- Respond to security advisories within 72 hours
- Participate in quarterly community coordination calls
- Submit annual impact report to the Foundation
- Vote on RFCs affecting the platform

### 8.5 Deliverables (ongoing)

| Deliverable | Frequency |
|---|---|
| Quarterly community contribution | Quarterly |
| Security advisory response log | Continuous |
| Annual impact report | Annually |
| Mentorship of L0/L1 programmes | Ongoing |

---

## 9. Assessment process

A country's current CAMM level is determined through a structured, evidence-based assessment. The assessment is a collaborative process, not an audit.

### 9.1 Assessment triggers

1. **Initial determination** — performed during L0 needs assessment
2. **Level advancement request** — initiated by the country team when they believe exit criteria are met
3. **Annual review** — re-assessment conducted annually for countries at L3+
4. **Incident-triggered review** — if a critical incident suggests capability gaps

### 9.2 Assessment steps

1. Country team reviews the exit criteria for their current level
2. Country team opens a GitHub issue in the CAMM repository with supporting evidence for each criterion
3. Foundation reviews and verifies the evidence within 10 business days
4. Foundation approves or returns the assessment with specific gaps identified
5. Country `STATUS.md` is updated to reflect the current level
6. Notification is sent to the country team

### 9.3 Evidence requirements

| Criterion type | Evidence format |
|---|---|
| Technical criteria | Link to GitHub workflow run, deployment state, or configuration file |
| Training criteria | Training completion records, certification badges |
| Volume criteria | Analytics dashboard export or database query result |
| Quality criteria | Data quality audit report |
| Governance criteria | Signed agreement, MoH letter, or meeting minutes |

### 9.4 Level regression

A country may be moved to a lower CAMM level if:

- The country requests regression
- Critical infrastructure has been down for 90+ consecutive days
- The country can no longer meet exit criteria of their current level
- Regression is documented in a GitHub issue with rationale

Level regression is a collaborative decision, not a punitive measure.

---

## 10. Prerequisites for entry

### 10.1 What the country must bring

| Prerequisite | Minimum requirement | Notes |
|---|---|---|
| Designated focal point | Named individual with decision authority | Must be added to GitHub organisation |
| Connectivity | One of: internet, occasional sync, or offline | Plane 0 works with zero connectivity |
| Hardware | Per deployment plane minimums | See → **003-zs-platform/003-deployment-planes.md** |
| Staff availability | Clinical staff for training | Training time: 8h L1, 16h L2, 24h L3 |
| Leadership buy-in | Signed MOU or equivalent | No financial commitment required |
| Data sovereignty policy | Statement of requirements | Determines hosting and encryption |

### 10.2 What the Foundation provides

| Resource | Detail |
|---|---|
| Platform software | All components under Apache 2.0 — zero cost, perpetually |
| Documentation | Full ZUSS-compliant documentation in English |
| Infrastructure templates | OpenTofu and Docker Compose templates for all planes |
| Training materials | Slide decks, video guides, practice environments |
| Community support | GitHub Issues response within 5 business days |
| Onboarding support | Direct support through L1 (Foundation stage) |
| Content library | Core clinical forms, translations, protocol definitions |

### 10.3 What is never required

- No financial payment of any kind
- No purchase of proprietary hardware or software
- No exclusive commitment to ZarishSphere over other systems
- No surrender of data ownership or sovereignty
- No signing of non-disclosure agreements

---

## 11. Templates and tools

The following templates support CAMM adoption. They are maintained in the CAMM repository and are freely reusable under CC BY 4.0.

### 11.1 CAMM templates

| Template | Purpose | CAMM level |
|---|---|---|
| MOU template | Formalises the partnership commitment | L0 |
| Needs assessment | Structured questionnaire for programme context | L0 |
| Technical readiness | Hardware, connectivity, and device checklist | L0 |
| Training curriculum | L1, L2, and L3 training tracks with durations | L1–L3 |
| Country STATUS.md | Tracks CAMM level, milestones, and contact info | All levels |

### 11.2 Infrastructure templates

Country deployment uses the following template repositories:

| Repository | Purpose | Created at |
|---|---|---|
| `zs-iac-template-country` | Infrastructure-as-Code starter for a country | L0 exit |
| `zs-distro-core` | Core distribution with default modules and forms | L0 exit |
| `zs-iac-template-site` | Per-facility deployment template | L2 |

See → **006-zs-infrastructure/001-infrastructure-overview.md** — Enterprise infrastructure and zero-touch deployment mapping.

### 11.3 Country distribution model

Each country programme operates its own fork of the core distribution:

```
zs-distro-core (upstream)
  └── zs-distro-{cc} (country fork)
       ├── config/
       │   ├── dhis2/
       │   ├── fhir/
       │   ├── keycloak/
       │   └── monitoring/
       ├── forms/
       ├── translations/
       ├── modules/
       └── README.md
```

Country forks are independent. They pull upstream updates at their own cadence. No country is required to accept upstream changes.

---

## 12. Cross-references

### 12.1 Foundation documents

→ **002-zs-foundation/001-foundation-charter.md** — Foundation mission, scope, and obligations to deployers
→ **002-zs-foundation/002-governance-model.md** — Decision-making model, including future governance bodies where L5 countries would have representation
→ **002-zs-foundation/003-licensing-policy.md** — Apache 2.0 and CC BY 4.0 dual-licensing rules governing all CAMM templates
→ **002-zs-foundation/004-contributor-guidelines.md** — How country teams contribute forms, translations, and code back to the community

### 12.2 Constitutional references

→ **001-zs-meta/001-zarishsphere-constitution.md** § Law 4 — Plane 0 compliance (all CAMM levels work offline)
→ **001-zs-meta/001-zarishsphere-constitution.md** § Law 5 — Permanent zero cost (no CAMM level requires payment)
→ **001-zs-meta/001-zarishsphere-constitution.md** § Law 6 — No-code accessibility (L1 training relies on GUI-only operations)
→ **001-zs-meta/001-zarishsphere-constitution.md** § Law 8 — Data portability (countries can export all data at any level)
→ **001-zs-meta/001-zarishsphere-constitution.md** § Law 9 — No vendor lock-in (L5 sovereignty is the designed end state)

### 12.3 Platform documents

→ **003-zs-platform/003-deployment-planes.md** — Infrastructure specifications for each plane that CAMM levels deploy on
→ **003-zs-platform/001-platform-overview.md** — Technical overview context for what countries are adopting

### 12.4 Standards documents

→ **005-zs-standards/016-terminology-governance.md** — Terminology code system governance for national system alignment at L3
→ **005-zs-standards/013-form-schema-specification.md** — ZS Form Schema v1 for country-specific form validation at L2

### 12.5 Infrastructure documents

→ **006-zs-infrastructure/001-infrastructure-overview.md** — Enterprise infrastructure design and zero-touch deployment
→ **006-zs-infrastructure/009-compliance-controls.md** — HIPAA and GDPR compliance for L3+ data governance

### 12.6 Architecture Decision Records

→ **008-zs-adrs/009-adr-no-vendor-lock-in.md** — Ensures no country is locked into any vendor; L5 sovereignty is the guaranteed outcome
→ **008-zs-adrs/012-adr-no-single-person-dependency.md** — Country operations must not depend on any single person at any CAMM level

### 12.7 Operational procedures

→ **009-zs-operations/013-sop-country-onboarding.md** — Operational procedure for onboarding a new country; implements the CAMM L0 entry and assessment process
→ **009-zs-operations/014-sop-facility-onboarding.md** — Operational procedure for onboarding a new health facility; implements the CAMM L1–L2 facility rollout

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
