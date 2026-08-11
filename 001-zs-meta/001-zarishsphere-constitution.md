# 001-zarishsphere-constitution.md
## ZarishSphere Ecosystem Constitution
### Twelve laws, four tiers, one supreme governing document — V1

**Document type:** Constitution — Supreme Governing Document
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Authoritative. All projects, repositories, components, and decisions are subordinate to this document.

---

## Table of contents

1. [Preamble](#1-preamble)
2. [The twelve laws](#2-the-twelve-laws)
3. [Tier I — Foundation laws](#3-tier-i--foundation-laws)
4. [Tier II — Rights laws](#4-tier-ii--rights-laws)
5. [Tier III — Architecture laws](#5-tier-iii--architecture-laws)
6. [Tier IV — Governance laws](#6-tier-iv--governance-laws)
7. [The five rights of every person served](#7-the-five-rights-of-every-person-served)
8. [The four obligations of the foundation](#8-the-four-obligations-of-the-foundation)
9. [Supremacy and conflict resolution](#9-supremacy-and-conflict-resolution)
10. [Amendment process](#10-amendment-process)
11. [Ecosystem scope](#11-ecosystem-scope)
12. [Document cross-references](#12-document-cross-references)

---

## 1. Preamble

The distance between a published standard and the person it is meant to serve is not a technical problem. It is a governance failure.

A WHO clinical guideline written in Geneva does not arrive in the hands of a community health worker in Cox's Bazar. An ISO technical specification published behind a paywall does not reach the engineer in Nairobi who needs it. A UNHCR protection framework that exists only as a PDF cannot be verified, queried, adapted, or deployed by the organization in the field that it governs. An ILO labor convention, a W3C web standard, an IETF protocol, an IFRS accounting rule, a building safety code, a human rights treaty — every one of them faces the same gap.

ZarishSphere exists to close this distance permanently — for every domain of human civilization, not only health.

The vision is total and uncompromising: a single, open, zero-cost ecosystem that indexes every meaningful standard, framework, treaty, guideline, specification, code, regulation, and protocol across every domain of human activity. Health. Education. Labor. Trade. Environment. Technology. Finance. Construction. Human rights. Humanitarian response. Governance. Science. Culture. Everything.

This is not a software project. It is a new kind of infrastructure for human civilization — a sovereign operating system for how standards become action, accessible to anyone with a browser, deployable by anyone with a GitHub account, buildable by a single person with zero budget using only free tools.

The ZarishSphere ecosystem comprises everything required to make this real:

- **ZarishSphere Foundation** — the governing institution that maintains the ecosystem
- **ZarishIndex** — a machine-readable unified index of every global standard across all domains
- **ZarishStandards** — the transformation layer that converts indexed metadata into structured, deployable digital assets
- **ZarishSphere Platform** — a platform-of-platforms: modular, layered, deployable at any scale from air-gapped single device to global SaaS
- **ZarishSphere Apps** — pre-built, ready-to-use applications for every domain
- **ZarishSphere Console** — the browser-based control center for managing the entire ecosystem
- **ZarishSphere Marketplace** — where modules, apps, templates, and standards are discovered and deployed
- **ZarishSphere Builder** — GUI-based tool for creating forms, workflows, and modules without code
- **ZarishSphere SDK and CLI** — programmatic interfaces for those who want them (always secondary to GUI)
- **ZarishSphere Services, Modules, Forms, Engines, and Distributions** — the complete catalog of deployable components
- **ZarishNote** — a free, open-source, ultra-lightweight WYSIWYG Markdown editor with a sandboxed private AI assistant. It is a clean, elegant Markdown editor that — when you need it — becomes a fully sandboxed private AI agent running entirely on your device, with no data leaving without your explicit permission. ZarishNote begins as a great writing tool. Before the AI, before the sandbox, before the MCP — the core is an editor that respects every flow. Open it and you see text. Not panels, not icons, not banners — type `#` and see a heading instantly. Type `- ` and a list begins. Split the view to see rendered Markdown live, or drop into raw source when you need precision. The sidebar shows your files. Search finds what you wrote. Tags group what belongs together. Tables, tasks, math (KaTeX), code blocks, images, and Mermaid diagrams are all native.

Every line of code, every document, every deployment, every organizational decision is subordinate to this constitution. No individual, organization, funder, government, or technology may override the laws stated here.

This document is a living instrument. It can be amended. It cannot be abandoned.

---

## 2. The twelve laws

The ZarishSphere ecosystem is governed by twelve laws, organized across four tiers. Laws in lower tiers may not contradict laws in higher tiers.

| Law | Title | Tier |
|---|---|---|
| 1 | GitHub is the government | I — Foundation |
| 2 | Documentation precedes existence | I — Foundation |
| 3 | Every standard is executable | I — Foundation |
| 4 | Identity without surveillance | II — Rights |
| 5 | Zero cost is a structural guarantee | II — Rights |
| 6 | No-code first | II — Rights |
| 7 | Module sovereignty | III — Architecture |
| 8 | Deployment plane sovereignty | III — Architecture |
| 9 | Vendor freedom | III — Architecture |
| 10 | Every decision is auditable forever | IV — Governance |
| 11 | The platform outlives its creators | IV — Governance |
| 12 | Contribution is borderless | IV — Governance |

**Tier precedence:** Tier I laws are inviolable. Tier II laws may be qualified only by Tier I. Tier III laws may be qualified by Tiers I and II. Tier IV laws may be qualified by any higher tier but never suspended.

---

## 3. Tier I — Foundation laws

These three laws may never be amended. They define what ZarishSphere fundamentally is. A change to any of these three laws constitutes the creation of a new, forked project — not an amendment to ZarishSphere.

### Law 1 — GitHub is the government

Every form, protocol, configuration, deployment, and governance decision in the ZarishSphere ecosystem is a git commit. Administrative, clinical, and platform governance happen through pull request. Audit trails are repository histories. Policy is a markdown file. A decision that has not been committed to a repository is not a ZarishSphere decision.

The `zarishsphere` GitHub organization is the nervous system of the entire ecosystem. Every entity — the Foundation, the Platform, ZarishIndex, and ZarishStandards — is represented by repositories within it. The organization does not exist independently of its repository history.

> **Constraint:** No verbal, email, messaging-platform, or informal decision is binding on the ZarishSphere ecosystem unless it has been committed as a document or configuration to a repository.

### Law 2 — Documentation precedes existence

No system, module, service, workflow, integration, or operational process may be built before a complete specification for it has been committed to `zs-docs` or the relevant project documentation repository. Markdown is the primary language of the ZarishSphere ecosystem. Code is the secondary language. A system that exists without committed documentation does not exist in ZarishSphere terms.

This law exists because documentation is the only mechanism by which a single founder, a future contributor, and an AI agent can all work on the same ecosystem with shared understanding. Code without documentation produces a system that only its author understands. Documentation without code produces a system that anyone can build.

> **Constraint:** No repository may receive its first code commit before its documentation repository contains a complete, committed project charter. The sequence is always: specification first, then code.

### Law 3 — Every standard is executable

No standard, protocol, guideline, classification, code of practice, specification, treaty, regulation, or framework that enters the ZarishSphere ecosystem may remain only in a format that a human must read to use. Every such artifact must become machine-readable, queryable, and deployable — regardless of domain.

ZarishIndex provides the unified index across all domains of human civilization. ZarishStandards provides the transformation layer. The Guideline-to-Action (G2A) Engine is the execution mechanism. The `ZI-[DOMAIN_CODE]-[NNNNN]` identifier format is the universal identifier that links every standard across the entire ecosystem.

A standard that cannot be queried by an API, consumed by a form engine, evaluated as a decision rule, compiled into an app, or deployed as an operational workflow is not a ZarishSphere-compatible standard. This applies equally to a WHO clinical guideline, an ISO technical specification, an ILO labor convention, a W3C web standard, an IETF protocol, an IFRS accounting rule, and a UN human rights treaty.

> **Constraint:** Every standard ingested into ZarishIndex must have, at minimum, a machine-readable metadata record with a `ZI-[DOMAIN]-[NNNNN]` identifier before it is considered indexed.

---

## 4. Tier II — Rights laws

These three laws define what ZarishSphere guarantees to every person, organization, and community it serves. They may not be suspended, qualified, or traded away for convenience, funding, or partnership.

### Law 4 — Identity without surveillance

Every person served by the ZarishSphere Platform is protected by architecture, not by policy. The technical design makes individual surveillance structurally impossible — not merely prohibited by policy or terms of service. Individual health, administrative, and personal data may not flow to any server outside the deploying entity's infrastructure without explicit, documented, revocable consent from both the individual and the deploying organization.

Emergency key destruction — the ability to render individual data permanently unreadable — must be technically implementable within 60 seconds on any deployment plane, including the lowest-resource deployment (Plane 0, air-gapped).

This law applies to ZarishSphere-operated infrastructure as much as to any deployer. The Foundation may not surveil users of the platform it governs.

> **Constraint:** No module may be merged into any ZarishSphere repository if its data architecture enables passive individual tracking at the network, application, or database layer without a documented, auditable consent mechanism.

### Law 5 — Zero cost is a structural guarantee

The ZarishSphere Platform, ZarishIndex, and ZarishStandards must remain permanently, unconditionally zero-cost for all humanitarian, public health, public sector, civil society, and resource-constrained deployments. Commercial pricing, freemium tiers, feature-gating, and usage limits may never apply to organizations or individuals who cannot pay.

This guarantee is not a business decision. It is encoded in the Apache 2.0 license, which permits free use, modification, and deployment by anyone, without restriction. No governance decision may override the license. No partnership agreement may create a carveout. No funder condition may create a paid tier for the populations this platform is designed to serve.

Commercial use by organizations with the capacity to pay is permitted and welcomed under Apache 2.0. Commercial revenue, if generated, returns to the Foundation to fund the zero-cost guarantee for those who cannot.

> **Constraint:** No component, service, integration, or dependency may be introduced into any ZarishSphere repository that makes a previously free, open function require payment to use.

### Law 6 — No-code first

Every function, configuration, workflow, and deployment process in the ZarishSphere ecosystem must be achievable through a graphical, browser-based interface before a command-line or programmatic equivalent is built. The GUI implementation is the primary implementation. CLI tools, API access, and programmatic interfaces are secondary — always optional, never required.

This law exists because the people who most need this platform — community health workers, district health officers, NGO program managers, government administrators in resource-constrained settings — are the least likely to have terminal access, programming knowledge, or reliable command-line environments. Building for them is not a constraint. It is the design intent.

> **Constraint:** A feature that exists only as a CLI command, API call, or code configuration is an incomplete feature. A feature that exists only as a documented, tested GUI workflow is a complete feature.

---

## 5. Tier III — Architecture laws

These three laws define how every component of ZarishSphere must be built. They are engineering constraints with the force of constitutional law. ADRs may document implementation choices within these constraints but may never contradict them.

### Law 7 — Module sovereignty

Every module, service, application, and domain of ZarishSphere is independently deployable. A health worker using the health module must never require the logistics module to be present. A government deploying only the education domain must never require the FHIR server to be running. A community using the nutrition module must never require a network connection to the health module.

Module independence is structural, not configurable. It cannot be achieved through a feature flag or deployment option. The architecture must make co-dependency between modules technically impossible.

This law enables the most critical deployment context: a single health worker with a Raspberry Pi and no internet connection, running one module in an air-gapped camp, serving people who have no other option.

> **Constraint:** Any two modules must be able to operate simultaneously, on different hardware, in different geographies, with no shared runtime dependency. A module that requires another module as a precondition violates this law.

### Law 8 — Deployment plane sovereignty

Any organization, health facility, government, individual, or community that deploys ZarishSphere owns their data completely, absolutely, and without condition. No ZarishSphere deployment may, by default, transmit telemetry, usage data, error reports, analytics, or any information of any kind to zarishsphere.com or any ZarishSphere-operated server.

This law applies equally across all five deployment planes:

| Plane | Context | Data ownership |
|---|---|---|
| Plane 0 — Air-gapped | No network, single device | Absolute — no transmission possible |
| Plane 1 — Raspberry Pi | Local network, single facility | Absolute — opt-in only for any outbound data |
| Plane 2 — District server | Local + intermittent internet | Absolute — no automatic sync without consent |
| Plane 3 — National cloud | Self-hosted cloud | Complete — deployer controls all infrastructure |
| Plane 4 — Global SaaS | ZarishSphere-operated | Clear data residency, export rights, deletion rights |

> **Constraint:** The platform must be fully functional, including all core modules, with zero network connection to any ZarishSphere-operated server. Any feature that requires a call to a ZarishSphere endpoint to function is a violation of this law.

### Law 9 — Vendor freedom

No ZarishSphere component, module, service, or workflow may depend on a proprietary API, a paid service, or a single-vendor infrastructure component as a required dependency. Every dependency must have an open-source, zero-cost alternative path that a deployer can use without the original vendor.

All data stored by ZarishSphere, at any deployment plane, must be exportable in internationally recognized open standards (FHIR R5, CSV, JSON, YAML, Parquet, or equivalent open formats) at any time, without restriction, without a fee, and without the consent of the Foundation.

Vendor lock-in is classified as a structural failure equivalent to a security vulnerability. It is not a trade-off. It is not a temporary pragmatic choice. It is a violation of this constitution.

> **Constraint:** Any component that requires a vendor account, vendor API key, or vendor-operated service to function in production violates this law. Cloudflare is used as an edge layer because it can be replaced; its use is architectural convenience, not dependency.

---

## 6. Tier IV — Governance laws

These three laws define how decisions are made, who holds the platform, and who can contribute to it. They govern the people and processes of ZarishSphere, not just its technology.

### Law 10 — Every decision is auditable forever

All architectural, platform, standards, governance, and operational decisions affecting the ZarishSphere ecosystem are recorded as Architecture Decision Records (ADRs) committed to the relevant repository before implementation begins. No undocumented decision may be implemented.

An ADR that is superseded is retained in the repository with its full history and a supersession record. The ADR is not deleted. The decision history of ZarishSphere — including every path not taken and every reason a different path was chosen — is as important as the current state of its code.

This law exists because a single founder building a complex ecosystem creates institutional memory risk. If the reasoning behind every decision is not committed, the ecosystem becomes opaque to future contributors, to AI agents working within it, and to the communities it serves who have a right to understand why it was built the way it was.

> **Constraint:** Any pull request that implements an architectural change to any ZarishSphere repository without a linked, committed ADR may be rejected. The ADR precedes the implementation, not the other way around.

### Law 11 — The platform outlives its creators

ZarishSphere must remain functional, forkable, and fully deployable by any person on earth, without requiring any individual's participation — including the founder — and without requiring any organization's continued operation — including the ZarishSphere Foundation.

No feature, workflow, critical function, API key, credential, or operational process may depend on a single person's access, knowledge, or continued availability. Every function that requires human knowledge must have that knowledge committed as a document. Every credential must have a documented rotation and succession process. Every access right must have a documented succession path.

This law exists because the people served by this platform live in conditions where systems fail regularly. The platform they depend on cannot be one of those systems. It must be structurally incapable of failing because a person left, an organization dissolved, or a funder withdrew.

> **Constraint:** Any system component that requires the founder's GitHub account, personal API key, personal credential, or undocumented knowledge to function in production violates this law. Succession documentation is a constitutional requirement, not an administrative nicety.

### Law 12 — Contribution is borderless

No feature, standard, module, documentation, or operational process in the ZarishSphere ecosystem may be locked to any single contributor, organization, language, geography, technical skill level, or sector. Contribution pathways must exist for every domain, every language, every deployment context, and every skill level.

A community health worker in Cox's Bazar who identifies a gap in the nutrition module and a data scientist in Geneva who can encode a WHO guideline must both have a viable, documented, supported path to contribute to the same ecosystem. Neither path may require the other. Neither contributor may be privileged over the other by default.

The ecosystem grows through participation, not permission. The Foundation's role is to maintain the standards of contribution, not to control what is contributed.

> **Constraint:** Any module, documentation folder, or operational process that cannot accept an external contribution through a documented, accessible process violates this law.

---

## 7. The five rights of every person served

Every person, organization, or community that uses, deploys, or is served by any ZarishSphere component has these five rights, regardless of deployment plane, geography, or funding status.

| Right | Statement |
|---|---|
| Right to access | Free access to the platform, its standards index, and its documentation — without payment, registration requirement, or identity disclosure |
| Right to understand | The right to know, in plain language and in their own language, what data the platform holds about them, how it is used, and how to request deletion |
| Right to sovereignty | The right to deploy the platform on their own infrastructure, export all their data at any time, and operate without connection to any ZarishSphere server |
| Right to contribute | The right to contribute to any public ZarishSphere repository through the documented contribution process, without requiring prior permission |
| Right to fork | The right to fork any ZarishSphere repository, modify it, and deploy it under the terms of the Apache 2.0 license — including commercially — without restriction |

---

## 8. The four obligations of the foundation

The ZarishSphere Foundation, as the governing institution of this ecosystem, has four unconditional obligations.

| Obligation | Statement |
|---|---|
| Maintain the standard | The Foundation maintains ZUSS, the ADR registry, and the constitutional framework. These do not degrade. |
| Preserve the history | The Foundation never deletes repository history, ADRs, or committed documentation — including unflattering decisions. |
| Enable succession | The Foundation maintains documented succession pathways for every critical system function. No single person is a single point of failure. |
| Serve the unreachable | The Foundation designs every process, tool, and interface for the person with the least access — the least connectivity, the least technical skill, the fewest resources. All other users are served as a consequence. |

---

## 9. Supremacy and conflict resolution

When any document, decision, code contribution, ADR, SOP, or configuration in the ZarishSphere ecosystem conflicts with this constitution, the constitution prevails.

When two laws within this constitution appear to conflict, resolution follows tier precedence: Tier I supersedes all. Tier II supersedes Tiers III and IV. Tier III supersedes Tier IV. Within a tier, no law supersedes another — the conflict is escalated to the amendment process.

When an ADR contradicts a law, the ADR is invalid. The implementation must be revised to comply with the law.

---

## 10. Amendment process

**Tier I laws (Laws 1–3):** Cannot be amended. Amendment of any Tier I law constitutes the creation of a fork.

**Tier II laws (Laws 4–6):** May be amended only by a committed document that:
1. States the specific law text being amended
2. States the specific replacement text
3. States the reason the amendment is required
4. References at least one ADR that explains the consequences
5. Is committed to `zs-docs/001-zs-meta/001-zarishsphere-constitution.md` with a documented reason in the commit message

**Tier III laws (Laws 7–9):** May be amended following the same process as Tier II, with the additional requirement that a ZUSS-compliant ADR is committed to `008-zs-adrs/` before the amendment is committed.

**Tier IV laws (Laws 10–12):** May be amended following the Tier III process.

**In V1 (pre-launch):** All amendments require commit by the founder. Post-launch: the governance model defined in `002-zs-foundation/002-governance-model.md` applies.

---

## 11. Ecosystem scope

This constitution governs the entire ZarishSphere ecosystem: all entities, components, repositories, documentation, deployments, apps, modules, services, and governance processes under the `zarishsphere` GitHub organization.

The ecosystem comprises **13 ecosystem components** — Console, Marketplace, Builder, Apps, Forms, SDK, CLI, API, Services, Modules, Distributions, Engine, and System — together with the **19 published product specifications** across the ecosystem catalog (the 13 component specifications plus the content, Home, and FHIR Hub specifications). All are subordinate to this constitution.

### 11.1 Governing entity

| Entity | Type | Role |
|---|---|---|
| ZarishSphere Foundation | Governing institution | Maintains the ecosystem, enforces the constitution, stewards all components |

### 11.2 Core infrastructure components

| Component | Type | Role |
|---|---|---|
| ZarishIndex | Open standards index | Machine-readable unified index of every global standard across all domains |
| ZarishStandards | Standards transformation layer | Converts indexed metadata into structured, deployable digital assets |
| ZarishSphere Platform | Platform-of-platforms | Modular, layered deployable infrastructure — SaaS, PaaS, BaaS, IaC |
| G2A Engine | Guideline-to-Action engine | Core automation that transforms standards into deployable assets |

### 11.3 User-facing ecosystem

| Component | Type | Role |
|---|---|---|
| ZarishSphere Apps | Pre-built applications | Ready-to-use domain applications for end users |
| ZarishSphere Console | Management console | Browser-based control center for the entire ecosystem |
| ZarishSphere Marketplace | Component marketplace | Discovery and deployment of modules, apps, templates, standards |
| ZarishSphere Forms | Form engine | Dynamic form generation from ZarishStandards definitions |
| ZarishSphere Builder | GUI builder | No-code tool for creating forms, workflows, modules |

### 11.4 Developer and integration layer

| Component | Type | Role |
|---|---|---|
| ZarishSphere SDK | Software development kit | Libraries and tools for building on the platform |
| ZarishSphere CLI | Command-line interface | Terminal access (always secondary to GUI per Law 6) |
| ZarishSphere API | Programmatic interface | REST and GraphQL APIs for all ecosystem components |
| ZarishSphere Services | Backend services | Domain-specific microservices and serverless functions |
| ZarishSphere Modules | Domain modules | Independently deployable module packages |
| ZarishSphere Distributions | Pre-packaged deployments | Ready-to-deploy bundles for specific use cases |

### 11.5 Platform infrastructure

| Component | Type | Role |
|---|---|---|
| ZarishSphere Engine | Core runtime | Execution environment for G2A and all modules |
| ZarishSphere System | Operating environment | Base system layer, identity, security, audit |
| ZarishSphere CI/CD | Automation pipeline | GitHub Actions workflows for build, test, deploy |

### 11.6 Scope principles

- Every component listed above is **open-source, zero-cost, and no-coder-friendly** by design
- Every component is **independently deployable** — no component requires another to function
- Every component is **documented in `zarishsphere` before** it receives its first code commit (Law 2)
- New components may be added by amendment to this section
- All repositories are under the `zarishsphere` GitHub organization: `https://github.com/zsdotcom`

---

## 12. Document cross-references

→ **002-zarishsphere-profile.md** — Foundation identity, mandate, and institutional profile
→ **003-zarishsphere-founder-profile.md** — Founder context, working environment, and AI interaction preferences
→ **004-zarishsphere-writing-rules.md** — ZUSS: naming, formatting, and documentation rules for all ecosystem files
→ **005-zarishsphere-ecosystem-architecture.md** — Complete ecosystem master map: components, repositories, navigation
→ **006-zarishsphere-glossary.md** — Canonical definitions for every ecosystem term
→ **007-zarishsphere-agent-ecosystem-strategy.md** — AI agent, MCP, and skills strategy
→ **008-zarishsphere-monorepo-blueprint.md** — Monorepo architecture blueprint (vision; see 005 for the current architecture)
→ **002-zs-foundation/002-governance-model.md** — Post-launch governance and decision-making model
→ **008-zs-adrs/001-adr-go-as-primary-language.md** — First ADR: Go as the primary language
→ **004-zs-index/004-harvesting-policy.md** — Strategic direction for the ZarishIndex autonomous research project

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
