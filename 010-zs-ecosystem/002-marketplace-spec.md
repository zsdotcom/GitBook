# 002-marketplace-spec.md
## ZarishSphere Marketplace specification
### Component discovery and deployment hub

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [What the Marketplace contains](#2-what-the-marketplace-contains)
3. [Listing requirements](#3-listing-requirements)
4. [Deployment flow](#4-deployment-flow)
5. [Licensing](#5-licensing)
6. [Cross-references](#6-cross-references)

---

## 1. Purpose

The Marketplace is the central discovery and deployment hub for all ZarishSphere ecosystem components. Users browse, search, compare, and install components — all from the Console, all without writing code.

## 2. What the Marketplace contains

| Category | Examples |
|---|---|
| Domain modules | Health, Education, Logistics, WASH, Protection |
| Pre-built apps | Patient registry, supply chain tracker, case management |
| Form templates | Intake forms, assessment tools, survey instruments |
| Workflow templates | Approval chains, referral pathways, notification rules |
| Report templates | Donor reports, compliance reports, dashboards |
| Distributions | Air-gapped clinic, district health office, national program |

## 3. Listing requirements

Every Marketplace listing includes:
- Name, description, version, license
- Screenshots and demo link
- Supported planes (0-4)
- Dependencies (declared in each component's manifest; the Marketplace itself imposes no dependency requirements)
- Installation instructions
- Maintenance status (stable, beta, deprecated)

Components may declare runtime dependencies in their manifests — for example, apps list the modules and services they require. The Marketplace verifies these declarations at install time, but the Marketplace itself imposes no dependency requirements of its own.

## 4. Deployment flow

```
User finds component → Clicks "Install" → Console downloads manifest →
Verifies dependencies → Pulls from GitHub → Deploys to selected plane →
Confirmation + status dashboard
```

All deployments are one-click from the Console. No terminal commands.

## 5. Licensing

All Marketplace items are open-source:
- Code: Apache 2.0
- Documentation/templates: CC BY 4.0
- No paid listings
- No premium tier
- No commission or listing fees

## 6. Cross-references

→ **010-zs-ecosystem/001-console-spec.md** — Console specification: the Marketplace is operated from the Console
→ **010-zs-ecosystem/003-builder-spec.md** — Builder specification: creates the templates and apps listed in the Marketplace
→ **010-zs-ecosystem/004-apps-spec.md** — Apps specification: pre-built apps whose manifests declare dependencies the Marketplace verifies
→ **010-zs-ecosystem/011-distributions-spec.md** — Distributions specification: pre-packaged deployment bundles listed in the Marketplace
→ **002-zs-foundation/003-licensing-policy.md** — Dual-license policy governing every Marketplace item
→ **008-zs-adrs/008-adr-apache-cc-dual-license.md** — ADR-008: the Apache 2.0 / CC BY 4.0 dual-license rule behind the Marketplace licensing section

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
