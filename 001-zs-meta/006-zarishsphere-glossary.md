# 006-zarishsphere-glossary.md
## ZarishSphere Ecosystem Glossary
### All terms, identifiers, and concepts defined — V1

**Document type:** Reference
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative. Any term used in any ZarishSphere document must be defined here.

---

## Table of contents

1. [How to use this glossary](#1-how-to-use-this-glossary)
2. [Entities and projects](#2-entities-and-projects)
3. [Core concepts](#3-core-concepts)
4. [Technical terms](#4-technical-terms)
5. [Identifier patterns](#5-identifier-patterns)
6. [Document types](#6-document-types)
7. [Infrastructure terms](#7-infrastructure-terms)
8. [Deployment planes](#8-deployment-planes)
9. [Acronyms and abbreviations](#9-acronyms-and-abbreviations)

---

## 1. How to use this glossary

Every term listed here is the canonical definition for that term across all ZarishSphere documents, repositories, ADRs, SOPs, and communications. When a document uses a term defined here, it uses it in exactly this sense. No document may redefine a term from this glossary without an amendment committed to this file.

Terms are listed in plain language first, then technical detail where applicable.

---

## 2. Entities and projects

**ZarishSphere Foundation**
The governing institution of the ZarishSphere ecosystem. The Foundation is not a legal entity (V1). It is the named maintainer of all ZarishSphere repositories and the author of all governance documents. The Foundation's "office" is the `zarishsphere` repository.

**ZarishSphere Platform**
The deployable software platform — a collection of independent domain modules, a shared FHIR R5 engine, and a Guideline-to-Action (G2A) Engine. The platform serves as SaaS, PaaS, BaaS, and IaC depending on the deployment plane. It covers all 40 domains, not only health.

**ZarishIndex**
The world's first free, open-source, machine-readable, human-navigable unified index of every global standard, framework, treaty, guideline, classification system, code of practice, and technical specification that governs human civilisation. Covers 40 domains. Primary output: `ZI-[DOMAIN]-[NNNNN]` identifier records with metadata and source references.

**ZarishStandards**
The transformation layer that converts ZARISH-INDEX metadata into structured, deployable standards resources. ZarishStandards does not re-index — it transforms. Input: ZARISH-INDEX records. Output: structured standard definitions consumable by ZarishSphere Platform modules.

**ZarishSphere Organization**
The GitHub organization at `https://github.com/zsdotcom`. All repositories for the entire ecosystem live here. This is the "nervous system" of the ecosystem per Law 1 of the constitution.

**ZarishSphere Console**
The browser-based management center for the ZarishSphere ecosystem. Every function — browsing standards, deploying modules, managing users, configuring integrations — is achievable through the Console without terminal access or programming knowledge. The Console is the primary interface for all ecosystem operations (Law 6).

**ZarishSphere Marketplace**
The discovery and deployment hub for all ecosystem components: domain modules, pre-built apps, form templates, workflow templates, report templates, standard definitions, and deployment distributions. All Marketplace items are open-source and free. No paid listings, no premium tier.

**ZarishSphere Builder**
A GUI-based no-code creation tool for building forms, workflows, modules, and apps. Uses drag-and-drop, visual flow design, and template assembly — no programming required. Everything built in the Builder can be exported as code and committed to GitHub.

**ZarishSphere Apps**
Pre-built, ready-to-use domain applications — patient registry, supply chain tracker, case management, and equivalents across all domains. Deployable immediately without configuration. Customizable through the Builder.

**ZarishSphere Forms**
A dynamic form engine that generates browser-based forms directly from ZARISH-STANDARDS definitions. Forms work offline (Plane 0-compatible), support all data types, and can be customized through the Builder.

**ZarishSphere SDK**
Software development kits (Go, JavaScript, Python) for developers building custom integrations, modules, or applications. The SDK is optional — all ecosystem functions are accessible through the Console without it.

**ZarishSphere CLI**
Command-line interface for terminal-based ecosystem management. Always secondary to the Console (Law 6). Used for CI/CD automation, bulk operations, and headless environments.

**ZarishSphere API**
RESTful and GraphQL APIs for all ecosystem components. API-first design. OpenAPI 3.2 specifications. Free to use — no paid API tiers.

**ZarishSphere Services**
Backend microservices powering the ecosystem: identity and access management, immutable audit logging, offline data synchronization, notification delivery, data export, and external system integration.

**ZarishSphere Modules**
Independently deployable domain packages — each module covers one domain (health, education, logistics, protection, etc.) and contains all forms, workflows, data models, and services for that domain. No module requires another module to function (Law 7).

**ZarishSphere Distributions**
Pre-packaged, pre-configured, pre-tested deployment bundles for specific use cases: air-gapped clinic, district health office, national program, humanitarian response, global SaaS. One-click deploy for users who want a complete solution without assembly.

**ZarishSphere Engine**
The core runtime that executes G2A transformations, runs domain modules, renders forms, orchestrates workflows, synchronizes offline data, and generates reports. The Engine is the execution layer beneath all other components.

**ZarishSphere System**
The base operating environment — identity and access management, encryption and key management, configuration management, health monitoring and alerting, backup and recovery, and audit infrastructure. Every other component depends on the System layer.

**Platform-of-platforms**
An architectural pattern where a platform is designed to host and integrate multiple sub-platforms, each independently deployable and independently governed. ZarishSphere is a platform-of-platforms: the base platform hosts domain modules, each of which is itself a platform for its domain.

---

## 3. Core concepts

**Single source of truth (SSoT)**
The principle that every fact, decision, configuration, and specification exists in exactly one authoritative location. In ZarishSphere, `zs-docs` is the SSoT for the platform and foundation. `zs-zarish-index/docs/` is the SSoT for ZARISH-INDEX. `zs-zarish-standards/docs/` is the SSoT for ZARISH-STANDARDS. No fact is stored in two places. All references point to the authoritative location.

**Documentation-first**
The practice of writing the complete specification for a system before building it. In ZarishSphere, this is Law 2: documentation precedes existence. A feature that is not documented does not exist.

**Guideline-as-Code (G2C)**
The principle that every international standard, WHO protocol, UNHCR framework, or ISO specification is represented as a machine-readable, executable artifact — not merely as a PDF. The G2A Engine is the implementation of Guideline-as-Code.

**GitHub as Government**
Law 1 of the constitution. The practice of using git commits, pull requests, and repository history as the mechanism for all governance, policy, and administrative decisions. Every policy is a markdown file. Every decision is a commit.

**Zero-cost guarantee**
Law 5 of the constitution. The unconditional commitment that humanitarian, public health, public sector, and resource-constrained deployments of ZarishSphere are permanently free — encoded in Apache 2.0.

**G2A Engine**
Guideline-to-Action Engine. The core innovation of ZarishSphere. An open-source system that automatically converts any international standard (from ZARISH-STANDARDS) into deployable digital assets: forms, decision rules, indicators, SOPs, and donor reports. The G2A Engine is the mechanism that makes Law 3 real.

**Module sovereignty**
Law 7 of the constitution. The architectural principle that every module in ZarishSphere is independently deployable — it requires no other module to function.

**Deployment plane sovereignty**
Law 8 of the constitution. The architectural principle that every deployer owns their data completely, with no telemetry or reporting to ZarishSphere servers by default.

**Vendor freedom**
Law 9 of the constitution. The architectural principle that ZarishSphere has no required dependency on any proprietary, paid, or single-vendor service.

**FDMN**
Forcibly Displaced Myanmar Nationals. The Rohingya refugee population in Cox's Bazar, Bangladesh — one of the primary deployment contexts for ZarishSphere V1.

**DPI**
Digital Public Infrastructure. The category of technology ZarishSphere belongs to — open, interoperable, foundational digital infrastructure for public benefit.

---

## 4. Technical terms

**FHIR R5**
Fast Healthcare Interoperability Resources, Release 5. The WHO-endorsed international standard for health data exchange. ZarishSphere uses FHIR R5 as the data model for all health-domain entities. FHIR R4 is not used (ADR-005).

**HAPI FHIR**
A Java-based open-source FHIR server implementation. Not used in ZarishSphere because it requires 2–3 GB RAM at minimum. The ZarishSphere hardware constraint (8 GB RAM, i3) makes HAPI FHIR incompatible. A Go-native FHIR R5 server is used instead (ADR-004).

**Go-native**
Refers to software built in the Go programming language that compiles to a single binary with no runtime dependencies. Go-native tools typically use 50–150 MB RAM. ZarishSphere requires Go-native tools for all server-side components due to the RAM constraint.

**ZUSS**
ZarishSphere Universal Serialization Standard. The complete rule set governing how every file, folder, repository, identifier, and document is named, structured, and written within the ZarishSphere ecosystem. Defined in `004-zarishsphere-writing-rules.md`.

**ADR**
Architecture Decision Record. A document recording one architectural, governance, or platform decision. Each ADR has six required sections: Decision, Context, Alternatives Considered, Reason for Decision, Consequences, Status. All ADRs live in `zs-docs/008-zs-adrs/`.

**MCP**
Model Context Protocol. A protocol that allows AI agents to access external tools and data sources. ZarishSphere documentation is structured to be MCP-consumable — AI agents can use `zs-docs` as context to build, navigate, and operate within the ecosystem.

**IaC**
Infrastructure as Code. The practice of defining and managing infrastructure (DNS records, cloud configurations, deployment settings) as committed, versioned files rather than through manual GUI operations. ZarishSphere uses Cloudflare and GitHub Actions as its IaC targets.

**XaaS**
Anything as a Service. The umbrella term for SaaS, PaaS, BaaS, IaaS, and related models. ZarishSphere uses the XaaS mental model to communicate its deployment options across five planes.

**SaaS**
Software as a Service. The deployer uses a fully hosted application. Data configuration only. Corresponds to ZarishSphere Deployment Plane 4 (Global SaaS).

**PaaS**
Platform as a Service. The deployer manages applications and data. Corresponds to ZarishSphere Deployment Plane 2 (District server, PaaS-like).

**BaaS**
Backend as a Service. Managed backend services, typically API + database. ZarishSphere modules can be deployed as BaaS components.

**CLI**
Command-line interface. A text-based interface for operating software via typed commands. In ZarishSphere, CLI is always secondary to GUI (Law 6). All CLI commands in all documentation are exact, copy-paste ready, and numbered.

**GUI**
Graphical user interface. A visual, browser-based interface. The primary implementation target for all ZarishSphere functions (Law 6).

---

## 5. Identifier patterns

| Identifier | Pattern | Example | Used for |
|---|---|---|---|
| ZARISH-INDEX entry | `ZI-[DOMAIN_CODE]-[NNNNN]` | `ZI-HEALTH-00001` | Every indexed standard |
| ZarishSphere patient | `ZS-[MODULE]-[YEAR]-[NNNNNN]` | `ZS-NCD-2025-000001` | Every person registered |
| Form ID | `zs-form-[domain]-[name]-v1` | `zs-form-ncd-intake-v1` | Every digital form |
| FHIR profile | `https://zarishsphere.com/fhir/StructureDefinition/zs-[resource]` | `zs-patient` | Every FHIR profile |
| ADR | `ADR-[NNN]-[title-kebab-case]` | `ADR-001-go-as-primary-language` | Every architecture decision |
| Service URL | `https://[service].zarishsphere.com/[path]` | `https://api.zarishsphere.com/fhir/R5/` | All service endpoints |
| Repository | `zs-{layer}-{module}[-{submodule}]` | `zs-fhir-server`, `zs-modules-health` | All GitHub repos |
| Docker image | `zarishsphere/[service-name]:[vN.N.N]` | `zarishsphere/zs-fhir-server:v1.0.0` | All container images |
| Document file | `nnn-descriptive-name.ext` | `001-zarishsphere-constitution.md` | All documentation files |
| Workflow file | `[id]--[trigger]--[process].yml` | `101--on-push--validate-markdown.yml` | All GitHub Actions workflows |

---

## 6. Document types

| Type | When to use | Required sections |
|---|---|---|
| Constitution | Supreme governing document | Preamble, Laws, Rights, Obligations, Supremacy, Amendment, Scope |
| Specification | Defining what something is and how it works | Purpose, Scope, Requirements, Architecture, Constraints |
| ADR | Recording one architectural decision | Decision, Context, Alternatives Considered, Reason, Consequences, Status |
| SOP | Step-by-step procedure for a repeatable process | Purpose, Scope, Roles, Preconditions, Steps (GUI-first), Expected outcome, Escalation |
| PRD | Product requirements for a feature | Problem, Goals, User stories, Functional requirements, Non-functional requirements, Acceptance criteria |
| Charter | Defining a project's mission and boundaries | Mission, Scope, Vision, Non-goals, Constraints, Relationships |
| Reference | Lookup document — glossary, registry, index | Defined per document; must be scannable by tables |
| Context profile | Background context for AI agents | Identity, Working environment, Preferences, Constraints |

---

## 7. Infrastructure terms

**Cloudflare**
The edge infrastructure provider used for DNS, CDN, static hosting (Pages), serverless functions (Workers), and email routing. Used because the free tier is permanently free, globally distributed, and sufficient for V1.

**Cloudflare Pages**
Static site hosting. Free tier: unlimited sites, 500 builds/month, custom domains. Used for all ZarishSphere public-facing documentation and platform interfaces.

**Cloudflare Workers**
Serverless JavaScript/WebAssembly functions at the edge. Free tier: 100,000 requests/day. Used for API routing and lightweight serverless functions.

**Cloudflare Email Routing**
Free email alias service. Routes incoming mail from `*@zarishsphere.com` to a destination inbox. No mail server operated. Unlimited aliases.

**Cloudflare DNS**
DNS management. Free, unlimited DNS records including unlimited subdomains. All `zarishsphere.com` DNS is managed here.

**GitHub Actions**
CI/CD automation built into GitHub. Free tier: 2,000 minutes/month for public repositories (unlimited for public repos). All ZarishSphere automation uses GitHub Actions. Workflow files follow ZUSS naming: `[id]--[trigger]--[process].yml`.

**GitHub Pages**
Alternative to Cloudflare Pages for static documentation hosting. Free for public repositories. ZarishSphere uses Cloudflare Pages as primary but GitHub Pages is an acceptable fallback.

---

## 8. Deployment planes

ZarishSphere defines five deployment planes, from fully offline to fully hosted. Each plane is independently viable — a deployer may use any plane without requiring any other.

| Plane | Name | Context | XaaS equivalent | Data ownership |
|---|---|---|---|---|
| Plane 0 | Air-gapped | No network, single device, offline-only | On-premises | Absolute — no transmission possible |
| Plane 1 | Raspberry Pi | Local network, single facility | IaaS + self-managed | Absolute — no outbound by default |
| Plane 2 | District server | Shared facility, intermittent internet | PaaS-like | Complete — deployer controls all |
| Plane 3 | National cloud | Self-hosted cloud infrastructure | SaaS + config | Complete — deployer controls all |
| Plane 4 | Global SaaS | ZarishSphere-hosted | Full SaaS | Data residency, export, and deletion rights guaranteed |

---

## 9. Acronyms and abbreviations

| Acronym | Expansion |
|---|---|
| ADR | Architecture Decision Record |
| BaaS | Backend as a Service |
| CLI | Command-line interface |
| DPI | Digital Public Infrastructure |
| FDMN | Forcibly Displaced Myanmar Nationals |
| FHIR | Fast Healthcare Interoperability Resources |
| G2A | Guideline-to-Action |
| G2C | Guideline-as-Code |
| GUI | Graphical user interface |
| IaC | Infrastructure as Code |
| IaaS | Infrastructure as a Service |
| IAM | Identity and Access Management |
| MCP | Model Context Protocol |
| MEAL | Monitoring, Evaluation, Accountability, and Learning |
| PaaS | Platform as a Service |
| PoP | Platform-of-Platforms |
| PRD | Product Requirements Document |
| REST | Representational State Transfer |
| SaaS | Software as a Service |
| SDK | Software Development Kit |
| SOP | Standard Operating Procedure |
| SSoT | Single Source of Truth |
| WASH | Water, Sanitation, and Hygiene |
| XaaS | Anything as a Service |
| ZI | ZARISH-INDEX (identifier prefix) |
| ZS | ZarishSphere (identifier prefix) |
| ZS-UID | ZarishSphere Universal Identifier |
| ZUSS | ZarishSphere Universal Serialization Standard |

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
