# 009-repository-catalog.md
## ZarishSphere repository catalog
### Complete reference: 212+ repositories across 11 layers (Layer 0 through Layer 10)

**Document type:** Reference
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Naming convention](#2-naming-convention)
3. [Layer 0 — Governance and meta](#3-layer-0--governance-and-meta)
4. [Layer 1 — Core platform kernel](#4-layer-1--core-platform-kernel)
5. [Layer 2 — Backend microservices](#5-layer-2--backend-microservices)
6. [Layer 3 — Frontend microfrontends](#6-layer-3--frontend-microfrontends)
7. [Layer 4 — Mobile applications](#7-layer-4--mobile-applications)
8. [Layer 5 — Desktop applications](#8-layer-5--desktop-applications)
9. [Layer 6 — Data as Code](#9-layer-6--data-as-code)
10. [Layer 7 — Infrastructure as Code](#10-layer-7--infrastructure-as-code)
11. [Layer 8 — Distribution layer](#11-layer-8--distribution-layer)
12. [Layer 9 — CI/CD, agents and automation](#12-layer-9--cicd-agents-and-automation)
13. [Layer 10 — Integration and interoperability](#13-layer-10--integration-and-interoperability)
14. [Summary statistics](#14-summary-statistics)
15. [Cross-references](#15-cross-references)

---

## 1. Purpose

### 1.1 Why this catalog exists

The ZarishSphere ecosystem spans 212+ repositories across GitHub. This catalog provides the authoritative reference — a single source of truth for discovering, understanding, and navigating every repository in the organisation.

### 1.2 Who it serves

| Audience | How they use this catalog |
|---|---|
| **Architects** | Understand system boundaries, dependency chains, and layer responsibilities |
| **Developers** | Find the correct repository for a given task, understand naming patterns |
| **Deployers** | Identify which repositories must be deployed for a given use case |
| **Contributors** | Locate the right repos for code contributions and issue tracking |
| **Integrators** | Find integration and bridge repositories for partner systems |

### 1.3 How to use

Repositories are organised into 11 functional layers (Layer 0 through Layer 10). Within each layer, repositories are grouped by sub-domain. Each entry includes the repository name, primary language or technology, and a description of its function.

New repositories must follow the naming convention defined in section 2 and be added to this catalog before creation.

---

## 2. Naming convention

All repositories follow the canonical `zs-<layer>-<module>[-<submodule>]` pattern:

```
zs-{layer}-{module}[-{submodule}]
```

### 2.1 Layer prefixes

| Prefix | Layer |
|---|---|
| `zs-docs-` | Documentation, RFCs, ADRs, standards |
| `zs-core-` | Platform kernel, FHIR engine, shared libraries |
| `zs-pkg-` (Go) | Shared Go libraries |
| `zs-pkg-ui-` | Shared frontend TypeScript packages |
| `zs-pkg-flutter-` | Shared Flutter/Dart packages |
| `zs-svc-` | Backend microservices (Go) |
| `zs-ui-` | Frontend microfrontends (React/Next.js) |
| `zs-mobile-` | Mobile applications (Flutter) |
| `zs-desktop-` | Desktop applications (Tauri) |
| `zs-content-` | Clinical content (forms, workflows, protocols) |
| `zs-data-` | Data as Code (terminologies, concepts, FHIR IGs) |
| `zs-iac-` | Infrastructure as Code (OpenTofu, Helm, K8s) |
| `zs-infra-` | Live infrastructure state (per country) |
| `zs-distro-` | Country/program distribution layers |
| `zs-agent-` | GitHub automation agents and bots |
| `zs-ops-` | Operational tooling, shared Actions |
| `zs-int-` | Integration and interoperability bridges |

### 2.2 Module naming rules

- Lowercase only
- Hyphens between words
- Descriptive but concise (avoid acronyms unless universally understood)
- Submodules separated by second hyphen: `zs-svc-patient` (no submodule), `zs-svc-dhis2-bridge` (submodule)

---

## 3. Layer 0 — Governance and meta

Documentation, RFCs, ADRs, governance, community health, and org-level configuration.

| # | Repository | Description | Primary content |
|---|---|---|---|
| 1 | `zs-docs` | Master documentation repository — this repo. Governance, architecture, ADRs, ZARISH-INDEX, ZARISH-STANDARDS, infrastructure, tech stack, operations, ecosystem specs | Markdown |
| 2 | `.github` | GitHub org-level community health files, PR templates, issue templates, CODEOWNERS | Markdown/YAML |
| 3 | `zs-docs-adr` | Architecture Decision Records — all significant technical and governance decisions | Markdown |
| 4 | `zs-docs-rfc` | Request for Comments — open proposals, drafts, community input | Markdown |
| 5 | `zs-docs-runbooks` | Operational runbooks — deployments, incidents, on-call, backup, recovery | Markdown |
| 6 | `zs-docs-security` | Security policies, HIPAA/GDPR compliance documentation, threat models | Markdown |
| 7 | `zs-docs-camm` | Country Adoption Maturity Model (CAMM) — onboarding frameworks and assessment tools | Markdown |

**Layer 0 total: 7 repositories**

> **Constraint:** All Layer 0 repositories are documentation-only (`doc-type: reference`). No production code lives here.

---

## 4. Layer 1 — Core platform kernel

### 4.1 FHIR engine and runtime

| # | Repository | Language | Description |
|---|---|---|---|
| 8 | `zs-core-fhir-engine` | Go | Primary FHIR R5 server — REST, search, history, validation, FHIRPath. Runs on Raspberry Pi 5 (8 GB). |
| 9 | `zs-core-fhir-r4-bridge` | Go | R4↔R5 translation layer — bridges partner systems still on FHIR R4 |
| 10 | `zs-core-fhir-validator` | Go | Standalone FHIR resource validator with profile and IG support |
| 11 | `zs-core-fhir-subscriptions` | Go | FHIR R5 topic-based subscription engine via NATS JetStream |
| 12 | `zs-core-fhirpath` | Go | FHIRPath 2.0 evaluator — used in validation, CDS, and search |
| 13 | `zs-core-cds-hooks` | Go | CDS Hooks 2.0 service — clinical decision support integration |

### 4.2 Shared Go libraries (packages)

| # | Repository | Description |
|---|---|---|
| 14 | `zs-pkg-go-fhir` | Shared Go FHIR types, builders, and utility functions |
| 15 | `zs-pkg-go-auth` | OIDC/SMART on FHIR token validation, JWT parsing, scope enforcement |
| 16 | `zs-pkg-go-db` | PostgreSQL connection pool, pgx v5.7.2 wrapper, JSONB helpers |
| 17 | `zs-pkg-go-cache` | Valkey/Redis client wrapper with FHIR-aware TTL patterns |
| 18 | `zs-pkg-go-messaging` | NATS JetStream client — pub/sub, durable consumers, FHIR events |
| 19 | `zs-pkg-go-audit` | FHIR AuditEvent generation, structured zerolog, compliance logging |
| 20 | `zs-pkg-go-telemetry` | OpenTelemetry 1.40 — traces, metrics, Prometheus exporter |
| 21 | `zs-pkg-go-config` | Viper-based config loader — YAML, env vars, Vault secrets |
| 22 | `zs-pkg-go-migration` | golang-migrate wrapper — idempotent PostgreSQL schema migrations |
| 23 | `zs-pkg-go-testing` | testcontainers-go fixtures, FHIR test helpers, integration test utilities |
| 24 | `zs-pkg-go-i18n` | Multi-language support — EN, BN, MY, UR, HI, TH (server-side) |
| 25 | `zs-pkg-go-crypto` | Encryption utilities — field-level encryption, pseudonymization |

### 4.3 Shared frontend libraries (TypeScript packages)

| # | Repository | Description |
|---|---|---|
| 26 | `zs-pkg-ui-design-system` | ZarishSphere design system — Carbon DS extensions, WCAG 2.2 AA |
| 27 | `zs-pkg-ui-fhir-hooks` | React hooks for FHIR resources — usePatient, useEncounter, etc. |
| 28 | `zs-pkg-ui-form-engine` | JSON-schema driven clinical form renderer (React Hook Form + Zod) |
| 29 | `zs-pkg-ui-offline-store` | Dexie.js offline FHIR resource store — IndexedDB with sync queue |
| 30 | `zs-pkg-ui-i18n` | i18next configuration — lazy-load language files for all supported locales |
| 31 | `zs-pkg-ui-charts` | Apache ECharts wrappers — clinical charts, vitals trends, BI dashboards |
| 32 | `zs-pkg-ui-maps` | OpenLayers wrappers — geo health data, facility maps, outbreak mapping |
| 33 | `zs-pkg-ui-auth` | SMART App Launch 2.2.0 launch, OIDC session management for SPA |

**Layer 1 total: 26 repositories** (6 FHIR engine + 12 Go libraries + 8 TypeScript packages)

---

## 5. Layer 2 — Backend microservices

### 5.1 Clinical services

| # | Repository | Port | Description |
|---|---|---|---|
| 34 | `zs-svc-patient` | 8001 | Patient registration, demographics, Master Patient Index (MPI) |
| 35 | `zs-svc-encounter` | 8002 | Encounter management — visits, episodes, care settings |
| 36 | `zs-svc-observation` | 8003 | Vitals, lab results, measurements (LOINC-coded FHIR Observations) |
| 37 | `zs-svc-medication` | 8004 | Medication requests, dispensing, administration records |
| 38 | `zs-svc-immunization` | 8005 | Immunization records, CVX coding, EPI program support |
| 39 | `zs-svc-condition` | 8006 | Diagnoses, problem lists, ICD-11/SNOMED CT coding |
| 40 | `zs-svc-procedure` | 8007 | Procedures, surgical notes, clinical procedures |
| 41 | `zs-svc-allergy` | 8008 | AllergyIntolerance — allergens, reactions, severity |
| 42 | `zs-svc-appointment` | 8009 | Scheduling, slot management, referrals, waitlists |
| 43 | `zs-svc-document` | 8010 | Clinical documents, discharge summaries, FHIR DocumentReference |
| 44 | `zs-svc-diagnostic` | 8011 | Diagnostic reports, imaging orders, lab orders (DICOM integration) |
| 45 | `zs-svc-care-plan` | 8012 | CarePlan, CareTeam, Goals — chronic disease management |
| 46 | `zs-svc-consent` | 8013 | FHIR Consent resources — patient consent, data sharing agreements |
| 47 | `zs-svc-nutrition` | 8014 | NutritionOrder, nutritional assessments, MUAC, MIYCN tracking |
| 48 | `zs-svc-maternity` | 8015 | ANC/PNC workflows, delivery records, postnatal care |
| 49 | `zs-svc-child-health` | 8016 | Pediatric growth monitoring, EPI, IMCI, development milestones |
| 50 | `zs-svc-mental-health` | 8017 | Mental health assessments, MHPSS programs, PHQ-9, GAD-7 |
| 51 | `zs-svc-communicable-disease` | 8018 | Communicable disease surveillance, outbreak detection, EWARS |
| 52 | `zs-svc-laboratory` | 8019 | Lab order management, result ingestion, HL7 v2 Lab ADT |
| 53 | `zs-svc-pharmacy` | 8020 | Formulary management, dispensing, stock integration |
| 54 | `zs-svc-radiology` | 8021 | DICOM worklist, imaging orders, PACS integration |

### 5.2 Public health and surveillance services

| # | Repository | Port | Description |
|---|---|---|---|
| 55 | `zs-svc-ewars` | 8030 | Early Warning and Response System — outbreak alerts, threshold monitoring |
| 56 | `zs-svc-dhis2-bridge` | 8031 | DHIS2 data exchange — aggregate export, tracker import, metadata sync |
| 57 | `zs-svc-godata-bridge` | 8032 | Go.Data integration — contact tracing, case investigation |
| 58 | `zs-svc-surveillance` | 8033 | Disease surveillance, case counts, trend analysis, EWARS feeds |
| 59 | `zs-svc-population` | 8034 | Population registry, catchment areas, denominators for indicators |
| 60 | `zs-svc-indicator` | 8035 | Health indicator calculation engine — WHO/UNICEF KPI definitions |
| 61 | `zs-svc-reporting` | 8036 | Report generation engine — MOH reports, donor reports, HMIS exports |
| 62 | `zs-svc-analytics` | 8037 | Aggregate analytics, BI data pipelines, Grafana data source |

### 5.3 Platform and operational services

| # | Repository | Port | Description |
|---|---|---|---|
| 63 | `zs-svc-auth` | 8050 | Auth service wrapping Keycloak — SMART on FHIR, user management |
| 64 | `zs-svc-notification` | 8051 | Multi-channel notifications — SMS, email, push, WhatsApp |
| 65 | `zs-svc-audit` | 8052 | Audit log aggregator — FHIR AuditEvent, HIPAA/GDPR event stream |
| 66 | `zs-svc-media` | 8053 | File/media management — photos, documents, Cloudflare R2 backed |
| 67 | `zs-svc-translation` | 8054 | Dynamic content translation — UI strings, form labels |
| 68 | `zs-svc-terminology` | 8055 | Terminology server — ICD-11, SNOMED CT, LOINC, CIEL, CVX, RxNorm |
| 69 | `zs-svc-search` | 8056 | Typesense-backed search — patient names, clinical notes, facilities |
| 70 | `zs-svc-geo` | 8057 | Geolocation — facility registry, GPS tracking, health zones |
| 71 | `zs-svc-scheduler` | 8058 | Background job scheduler — report generation, sync, backups |
| 72 | `zs-svc-webhook` | 8059 | Webhook dispatcher — outbound FHIR R5 subscriptions, partner feeds |
| 73 | `zs-svc-import` | 8060 | Data import — CSV, HL7 v2, legacy system migration |
| 74 | `zs-svc-export` | 8061 | Data export — FHIR bulk export, CSV, PDF reports |
| 75 | `zs-svc-gateway` | 8080 | API Gateway (Traefik) — routing, rate limiting, CORS |

### 5.4 Business and ERP services

| # | Repository | Port | Description |
|---|---|---|---|
| 76 | `zs-svc-hrm` | 8070 | Human Resource Management — staff registry, contracts, schedules |
| 77 | `zs-svc-finance` | 8071 | Financial management — budgets, expenditure, donor fund tracking |
| 78 | `zs-svc-accounting` | 8072 | Accounting — general ledger, cost centers, grant accounting |
| 79 | `zs-svc-inventory` | 8073 | Supply chain — medical stock, procurement, expiry tracking |
| 80 | `zs-svc-logistics` | 8074 | Logistics — cold chain, distribution, last-mile delivery tracking |
| 81 | `zs-svc-procurement` | 8075 | Procurement workflows — purchase orders, vendor management, contracts |
| 82 | `zs-svc-crm` | 8076 | Community relations — beneficiary registry, referral management |
| 83 | `zs-svc-asset` | 8077 | Asset management — equipment, vehicles, maintenance logs |
| 84 | `zs-svc-project` | 8078 | Project management — grants, deliverables, M&E frameworks |

**Layer 2 total: 51 repositories** (21 clinical + 8 public health + 13 operational + 9 ERP)

---

## 6. Layer 3 — Frontend microfrontends

### 6.1 Clinical microfrontends

| # | Repository | Domain | Description |
|---|---|---|---|
| 85 | `zs-ui-patient-registration` | apps | Patient registration shell — demographics, ID, consent |
| 86 | `zs-ui-patient-profile` | apps | Patient profile — timeline, conditions, encounters |
| 87 | `zs-ui-encounter-form` | apps | Encounter form renderer — JSON-schema forms, clinical workflow |
| 88 | `zs-ui-vitals-entry` | apps | Vitals capture — BP, weight, height, MUAC, temperature |
| 89 | `zs-ui-medication-management` | apps | Medication orders, dispensing, medication administration record |
| 90 | `zs-ui-appointment-scheduler` | apps | Appointment booking — calendar, slots, reminders |
| 91 | `zs-ui-lab-results` | apps | Lab result viewer — trend charts, reference ranges |
| 92 | `zs-ui-clinical-notes` | apps | Rich clinical notes — Tiptap editor, SOAP/POMR templates |
| 93 | `zs-ui-care-plan` | apps | Care plan builder — goals, tasks, team assignments |
| 94 | `zs-ui-maternity` | apps | Antenatal/postnatal care — partograph, birth registry |
| 95 | `zs-ui-child-health` | apps | Child health — growth charts, EPI schedule, IMCI |
| 96 | `zs-ui-immunization` | apps | Vaccination records, EPI tracking, campaign management |
| 97 | `zs-ui-mental-health` | apps | Mental health — PHQ-9, GAD-7, MHPSS workflows |
| 98 | `zs-ui-communicable-disease` | apps | Communicable disease case investigation forms, outbreak response |
| 99 | `zs-ui-pharmacy` | apps | Pharmacy dispensing, formulary, stock alerts |
| 100 | `zs-ui-radiology` | apps | Imaging orders, DICOM viewer integration |

### 6.2 Public health and surveillance microfrontends

| # | Repository | Domain | Description |
|---|---|---|---|
| 101 | `zs-ui-dashboard-clinical` | health | Clinical dashboard — facility-level KPIs, daily statistics |
| 102 | `zs-ui-dashboard-public-health` | health | Public health dashboard — population indicators, program coverage |
| 103 | `zs-ui-dashboard-surveillance` | health | Disease surveillance — EWARS alerts, outbreak maps |
| 104 | `zs-ui-dashboard-analytics` | health | BI analytics — Grafana-embedded, customizable reports |
| 105 | `zs-ui-geo-mapping` | health | Geospatial health — facility maps, catchment zones, outbreak mapping |
| 106 | `zs-ui-indicator-tracker` | health | WHO/UNICEF indicator tracking, M&E dashboards |
| 107 | `zs-ui-report-builder` | health | Drag-and-drop report builder — MOH, donor, HMIS |

### 6.3 Operational microfrontends

| # | Repository | Domain | Description |
|---|---|---|---|
| 108 | `zs-ui-admin-console` | ops | Platform admin — user management, org config, system settings |
| 109 | `zs-ui-form-builder` | forms | No-code form builder — drag-and-drop, JSON schema output |
| 110 | `zs-ui-workflow-builder` | ops | Clinical workflow designer — BPMN-lite, state machines |
| 111 | `zs-ui-terminology-browser` | ops | Terminology browser — ICD-11, SNOMED CT, LOINC, CIEL |
| 112 | `zs-ui-facility-registry` | ops | Facility management — health posts, hospitals, clinics |
| 113 | `zs-ui-user-management` | ops | User/role management — RBAC, team assignments |
| 114 | `zs-ui-audit-viewer` | ops | Audit log explorer — HIPAA-compliant access history |
| 115 | `zs-ui-notification-center` | ops | Notification management — channels, templates, schedules |
| 116 | `zs-ui-import-wizard` | ops | Data import wizard — CSV upload, field mapping, validation |

### 6.4 Business and ERP microfrontends

| # | Repository | Domain | Description |
|---|---|---|---|
| 117 | `zs-ui-hrm` | ops | HR dashboard — staff profiles, leave, payroll, rosters |
| 118 | `zs-ui-inventory` | ops | Stock management — medical supplies, procurement, expiry |
| 119 | `zs-ui-finance` | ops | Finance — budgets, expenditure, donor reporting |
| 120 | `zs-ui-project-management` | ops | Grant/project management — milestones, M&E, donor reports |
| 121 | `zs-ui-crm` | ops | Community CRM — beneficiary lists, referrals, field visits |
| 122 | `zs-ui-logistics` | ops | Logistics and supply chain — cold chain, distribution, tracking |

### 6.5 Shell applications (composition layer)

| # | Repository | Domain | Description |
|---|---|---|---|
| 123 | `zs-ui-shell-clinical` | apps | Clinical app shell — composes clinical microfrontends, routing |
| 124 | `zs-ui-shell-ops` | ops | Operations app shell — HR, finance, logistics, project shells |
| 125 | `zs-ui-shell-public` | health | Public health shell — surveillance, dashboards, maps |
| 126 | `zs-ui-portal-patient` | health | Patient-facing portal — records, appointments, messages |
| 127 | `zs-ui-landing` | — | Public website — landing pages, docs site |

**Layer 3 total: 43 repositories** (16 clinical + 7 public health + 9 operational + 6 ERP + 5 shells)

---

## 7. Layer 4 — Mobile applications

All mobile applications are built with Flutter, targeting Android and iOS with offline-first capability.

| # | Repository | Platform | Description |
|---|---|---|---|
| 128 | `zs-mobile-clinic` | Android/iOS | Primary clinical mobile app — offline-first EMR, PowerSync |
| 129 | `zs-mobile-community` | Android/iOS | Community health worker app — household visits, surveys, referrals |
| 130 | `zs-mobile-ewars` | Android/iOS | EWARS field reporting — outbreak alerts, tally sheets |
| 131 | `zs-mobile-supervisor` | Android/iOS | Supervisor dashboard — staff performance, facility monitoring |
| 132 | `zs-mobile-patient` | Android/iOS | Patient-facing app — appointments, records, health education |
| 133 | `zs-pkg-flutter-fhir` | Dart | Shared Flutter FHIR client, models, offline sync (PowerSync) |
| 134 | `zs-pkg-flutter-ui` | Dart | Shared Flutter component library — design system for mobile |

**Layer 4 total: 7 repositories** (5 apps + 2 shared packages)

---

## 8. Layer 5 — Desktop applications

Desktop applications use Tauri (Rust-based) for minimal binary footprint and cross-platform support.

| # | Repository | Platform | Description |
|---|---|---|---|
| 135 | `zs-desktop-clinic` | Win/Mac/Linux | Desktop clinical app — embeds Go FHIR binary, fully offline |
| 136 | `zs-desktop-admin` | Win/Mac/Linux | Platform admin desktop — org setup, bulk import, system config |

**Layer 5 total: 2 repositories**

---

## 9. Layer 6 — Data as Code

All content including clinical forms, terminologies, concept maps, FHIR profiles, and translations stored as version-controlled data.

### 9.1 Clinical content (forms and workflows)

| # | Repository | Description |
|---|---|---|
| 137 | `zs-content-forms-core` | Core clinical forms — registration, triage, vitals (JSON Schema) |
| 138 | `zs-content-forms-maternity` | ANC, PNC, delivery, postnatal forms (JSON Schema) |
| 139 | `zs-content-forms-child-health` | Pediatric forms — growth, EPI, IMCI, development (JSON Schema) |
| 140 | `zs-content-forms-nutrition` | MUAC, MIYCN, SAM/MAM assessment forms (JSON Schema) |
| 141 | `zs-content-forms-mental-health` | MHPSS forms — PHQ-9, GAD-7, psychosocial screening (JSON Schema) |
| 142 | `zs-content-forms-cd` | Communicable disease forms — TB, malaria, cholera, COVID (JSON Schema) |
| 143 | `zs-content-forms-ncd` | Non-communicable disease forms — HTN, DM, asthma, cancer screening |
| 144 | `zs-content-forms-emergency` | Emergency/trauma forms — triage, resuscitation, mass casualty |
| 145 | `zs-content-forms-laboratory` | Lab order and result forms — chemistry, hematology, microbiology |
| 146 | `zs-content-forms-pharmacy` | Medication order, dispensing, counseling forms |
| 147 | `zs-content-forms-community` | CHW forms — household surveys, WASH, GBV screening |
| 148 | `zs-content-forms-ops` | Operational forms — HR, logistics, procurement, finance |
| 149 | `zs-content-workflows` | Clinical workflow definitions — state machines, decision trees |
| 150 | `zs-content-protocols` | Clinical protocols — treatment algorithms, clinical decision support |

### 9.2 Terminology and concepts

| # | Repository | Description |
|---|---|---|
| 151 | `zs-data-icd11` | ICD-11 (2026 edition) — PostgreSQL seed, Go client, annual update pipeline |
| 152 | `zs-data-snomed` | SNOMED CT — RF2 loader, GIN-indexed PostgreSQL |
| 153 | `zs-data-loinc` | LOINC — CSV loader, lab code mappings, panel definitions |
| 154 | `zs-data-ciel` | OpenMRS CIEL — concept dictionary, mappings, update pipeline |
| 155 | `zs-data-rxnorm` | RxNorm — NLM API client, medication name normalization |
| 156 | `zs-data-cvx` | CVX vaccine codes — CDC flat file loader, EPI program mappings |
| 157 | `zs-data-facility-registry` | Global health facility registry — FHIR Location resources by country |
| 158 | `zs-data-concept-maps` | Cross-terminology concept maps — ICD-10↔ICD-11, SNOMED↔CIEL |
| 159 | `zs-data-value-sets` | FHIR ValueSets — curated sets for ZarishSphere clinical domains |
| 160 | `zs-data-fhir-profiles` | FHIR Implementation Guide — ZarishSphere IG, StructureDefinitions |
| 161 | `zs-data-indicators` | WHO/UNICEF indicator definitions — FHIR Measure resources |
| 162 | `zs-data-translations` | Multilingual translations — EN, BN, MY, UR, HI, TH (JSON/CSV) |

**Layer 6 total: 26 repositories** (14 clinical content + 12 terminology and concepts)

> **Constraint:** Layer 6 repositories are data-only — no runtime code. All data is versioned and deployed via CI/CD pipelines.

---

## 10. Layer 7 — Infrastructure as Code

### 10.1 Core infrastructure

| # | Repository | Description |
|---|---|---|
| 163 | `zs-iac-platform` | Core platform IaC — shared OpenTofu modules, Kubernetes base configs |
| 164 | `zs-iac-networking` | Networking — Traefik, Cilium, Cloudflare, DNS, TLS, CDN |
| 165 | `zs-iac-storage` | Storage — PostgreSQL, TimescaleDB, Valkey, NATS, Typesense configs |
| 166 | `zs-iac-observability` | Observability stack — Prometheus, Grafana, Loki, Tempo, Alertmanager |
| 167 | `zs-iac-security` | Security — HashiCorp Vault, Keycloak, network policies, RBAC |
| 168 | `zs-iac-registry` | Container registry — GHCR, image signing, Trivy scanning |
| 169 | `zs-iac-gitops` | Argo CD configuration — ApplicationSets, app-of-apps pattern |
| 170 | `zs-iac-helm-charts` | Helm charts — all ZarishSphere services, shared chart library |
| 171 | `zs-iac-backup` | Backup automation — PostgreSQL, Valkey, R2 object storage |

### 10.2 Country infrastructure (per-environment state)

| # | Repository | Country | Description |
|---|---|---|---|
| 172 | `zs-infra-bgd` | Bangladesh | BGD environment — OpenTofu state, K8s manifests, site configs |
| 173 | `zs-infra-ind` | India | IND environment — OpenTofu state, K8s manifests, site configs |
| 174 | `zs-infra-mmr` | Myanmar | MMR environment — OpenTofu state, K8s manifests, site configs |
| 175 | `zs-infra-pak` | Pakistan | PAK environment — OpenTofu state, K8s manifests, site configs |
| 176 | `zs-infra-tha` | Thailand | THA environment — OpenTofu state, K8s manifests, site configs |

### 10.3 Environment templates

| # | Repository | Description |
|---|---|---|
| 177 | `zs-iac-template-country` | One-click country environment template — fork to onboard new country |
| 178 | `zs-iac-template-site` | Site-level deployment template — fork to add a health facility |
| 179 | `zs-iac-dev-environment` | Local development environment — Docker Compose, devcontainer configs |

**Layer 7 total: 17 repositories** (9 core IaC + 5 country infrastructure + 3 templates)

---

## 11. Layer 8 — Distribution layer

Pre-packaged, pre-configured deployment bundles per country, program, or use case.

| # | Repository | Description |
|---|---|---|
| 180 | `zs-distro-core` | Core distro — global defaults, base FHIR profiles, core forms |
| 181 | `zs-distro-bgd` | Bangladesh distro — BGD forms, DHIS2 config, MOH terminology |
| 182 | `zs-distro-bgd-cxb` | Cox's Bazar distro — Rohingya refugee response, camp health programs |
| 183 | `zs-distro-ind` | India distro — national health ID, ABDM integration, IN forms |
| 184 | `zs-distro-mmr` | Myanmar distro — MMR forms, cross-border health, conflict settings |
| 185 | `zs-distro-pak` | Pakistan distro — PKI integration, Urdu forms, national codes |
| 186 | `zs-distro-tha` | Thailand distro — THA forms, national code sets, MoPH integration |
| 187 | `zs-distro-humanitarian` | Humanitarian distro — UNHCR, SPHERE standards, camp settings |
| 188 | `zs-distro-refugee-response` | Refugee response distro — biometric ID, camp registration, mVAM |

**Layer 8 total: 9 repositories**

---

## 12. Layer 9 — CI/CD, agents and automation

GitHub bots, shared Actions, and automation agents for the entire ecosystem.

| # | Repository | Description |
|---|---|---|
| 189 | `zs-agent-platform-bot` | Main GitHub App — org-wide automation, RFC state machine, labeler |
| 190 | `zs-agent-code-review` | AI code review bot — FHIR compliance, Go lint, security scan |
| 191 | `zs-agent-content-validator` | Clinical content validator — JSON form validation, concept lookup |
| 192 | `zs-agent-dependency-updater` | Renovate bot config — dependency updates across all repositories |
| 193 | `zs-agent-docs-generator` | Auto-generates API docs from OpenAPI specs, publishes to docs site |
| 194 | `zs-agent-release-manager` | Automated release notes, changelog, versioning across repos |
| 195 | `zs-agent-security-scanner` | Trivy + CodeQL + GitGuardian — org-wide security scanning |
| 196 | `zs-agent-smoke-tester` | Post-deploy smoke tests — FHIR endpoint health checks, critical paths |
| 197 | `zs-ops-github-actions` | Shared GitHub Actions — reusable workflows for all repositories |

**Layer 9 total: 9 repositories**

---

## 13. Layer 10 — Integration and interoperability

Bridges to partner systems, legacy platforms, and interoperability standards.

| # | Repository | Description |
|---|---|---|
| 198 | `zs-int-dhis2` | DHIS2 integration — metadata sync, aggregate reporting, tracker |
| 199 | `zs-int-openmrs` | OpenMRS integration — patient sync, concept mapping, forms bridge |
| 200 | `zs-int-openimis` | OpenIMIS integration — health insurance, claims, beneficiary registry |
| 201 | `zs-int-godata` | Go.Data integration — contact tracing, case investigation export |
| 202 | `zs-int-hl7v2` | HL7 v2.9 adapter — ADT, ORU, ORM messages from legacy systems |
| 203 | `zs-int-dicom` | DICOM integration — PACS worklist, imaging order bridge |
| 204 | `zs-int-lims` | LIMS integration — lab order exchange, result ingestion |
| 205 | `zs-int-abdm` | ABDM (India) — Ayushman Bharat Digital Mission integration |
| 206 | `zs-int-sms-gateway` | SMS gateway adapters — Twilio, Africa's Talking, local operators |
| 207 | `zs-int-who-icd11` | WHO ICD-11 API client — code lookup, hierarchy traversal |

**Layer 10 total: 10 repositories**

---

## 14. Summary statistics

| Layer | Count | Description |
|---|---|---|
| 0 — Governance and meta | 7 | Docs, RFCs, ADRs, org-level files |
| 1 — Core platform kernel | 26 | FHIR engine + Go/TypeScript shared libraries |
| 2 — Backend microservices | 51 | Go microservices (clinical, PH, ops, ERP) |
| 3 — Frontend microfrontends | 43 | React microfrontends + shell apps |
| 4 — Mobile applications | 7 | Flutter apps + shared packages |
| 5 — Desktop applications | 2 | Tauri apps |
| 6 — Data as Code | 26 | Forms, terminologies, FHIR IGs, translations |
| 7 — Infrastructure as Code | 17 | OpenTofu, Helm, GitOps, country infra |
| 8 — Distribution layer | 9 | Country and program distribution layers |
| 9 — CI/CD, agents and automation | 9 | GitHub bots, actions, automation |
| 10 — Integration and interoperability | 10 | DHIS2, HL7v2, DICOM, partner systems |
| **Total** | **207** | **Listed repositories** |

> **Note:** Total count is 207 explicitly listed repositories. The remaining capacity to 212+ accounts for planned expansion in Layers 2, 6, 7, and 10 as new country deployments and integration partners are added.

---

## 15. Cross-references

→ **001-zs-meta/005-zarishsphere-ecosystem-architecture.md** — Ecosystem master map with the repository inventory and creation sequence
→ **003-zs-platform/008-domain-registry.md** — The 40-domain registry behind `zs-modules-*` and distribution repositories
→ **007-zs-tech-stack/001-tech-stack-master.md** — Version-pinned technology inventory these repositories implement
→ **008-zs-adrs/001-adr-go-as-primary-language.md** — ADR-001: Go as the primary language for backend services
→ **010-zs-ecosystem/010-modules-spec.md** — Modules component specification served by the module repositories
→ **003-zs-platform/010-free-tier-resource-map.md** — Free-tier constraints (GitHub Actions, ARM64) shaping repository CI/CD design

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
