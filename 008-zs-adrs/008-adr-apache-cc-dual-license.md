# 008-adr-apache-cc-dual-license.md
## ADR-008: Dual-Licensing Rules for Open-Source Modularity
### Apache 2.0 for all code — CC BY 4.0 for all documentation

**Document type:** ADR
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Accepted

---

## Table of Contents

1. [Decision](#1-decision)
2. [Context](#2-context)
3. [Alternatives Considered](#3-alternatives-considered)
4. [Reason for Decision](#4-reason-for-decision)
5. [Consequences](#5-consequences)
6. [Status](#6-status)
7. [Cross-references](#7-cross-references)

---

## 1. Decision

All ZarishSphere ecosystem outputs are dual-licensed:

- **Code** (Go, TypeScript/JavaScript, YAML configuration, SQL, Dockerfiles, CI/CD scripts, and all executable artifacts): Apache License 2.0
- **Documentation** (Markdown files, specifications, ADRs, SOPs, standards descriptions, architecture documents, educational materials, and all non-executable artifacts): Creative Commons Attribution 4.0 International (CC BY 4.0)

Every repository, file, and artifact must clearly indicate which license applies. Mixed repositories (containing both code and documentation) carry both licenses with clear demarcation.

---

## 2. Context

The ZarishSphere ecosystem produces two fundamentally different types of intellectual property: executable code (the Platform, SDK, CLI, console, services) and non-executable documentation (governance documents, architecture records, standards indexes, user guides, API references). These two categories serve different purposes and require different legal frameworks.

The license choice must satisfy several constitutional constraints:

- **Constitution Law 5** (zero-cost structural guarantee): The license must ensure permanent, unconditional free access for humanitarian, public health, and resource-constrained deployments.
- **Constitution Law 9** (vendor freedom): The license must not restrict deployers' ability to use, modify, and redistribute the platform.
- **Constitution Law 12** (borderless contribution): The license must not create legal barriers to contribution.
- **Constitution Law 1** (documentation precedes existence): Documentation is the primary artifact of the ecosystem and must be freely shareable and reusable.

Three licensing approaches were evaluated: single license (Apache 2.0 or MIT or GPL), dual license (Apache 2.0 + CC BY 4.0), and a custom license.

---

## 3. Alternatives Considered

| Alternative | Pros | Cons |
|---|---|---|
| **Apache 2.0 (code) + CC BY 4.0 (docs)** | Code: business-friendly, patent grant, no copyleft concerns, compatible with commercial use; Docs: most permissive Creative Commons license, allows sharing/remix/translation, ideal for standards documentation; clear separation of concerns; widely adopted and understood | Requires license labeling on every artifact; users must check which license applies to a given file; CC BY 4.0 requires attribution which can be cumbersome for derivative works |
| **MIT License (all artifacts)** | Simplest license, maximum permissiveness, one license for everything, widely adopted | No patent protection (unlike Apache 2.0); does not address patent grants from contributors; less protective of the ecosystem's open nature; CC BY is better suited for documentation (explicitly designed for non-software creative works) |
| **GNU GPL v3 (code) + CC BY 4.0 (docs)** | Strong copyleft ensures modifications remain open; prevents proprietary forks from competing | GPL is too restrictive for health system adoption — many hospitals and health ministries have policies against GPL software; incompatible with some open-source projects; may prevent commercial ecosystem growth; founders who want to build on ZarishSphere may avoid it; would limit adoption in the very sectors ZarishSphere aims to serve |
| **Custom license (Foundation-specific)** | Perfectly tailored to ZarishSphere's goals | Legally risky — custom licenses are untested in court; creates confusion ("yet another license"); violates open-source definition (OSI approval unlikely); increases barriers for corporate adoption (legal teams must review novel licenses) |
| **Unlicense / Public Domain** | Maximum freedom, no restrictions | No patent grant; no attribution requirement (could harm reputation); legally uncertain in some jurisdictions; unattractive for corporate contribution (no legal framework); does not protect the ecosystem's identity |

---

## 4. Reason for Decision

1. **Apache 2.0 for code:** Apache 2.0 is the standard choice for health IT infrastructure. It provides:
   - **Patent grant:** Contributors explicitly grant patent rights for their contributions — critical in health technology where software patents are common.
   - **No copyleft restriction:** Organizations can use, modify, and integrate ZarishSphere code into proprietary systems without the GPL's "viral" requirement. This is essential for adoption in hospital systems, government health IT, and commercial health software.
   - **Trademark protection:** Apache 2.0 does not grant trademark rights, protecting the ZarishSphere brand.
   - **Zero-cost guarantee:** The license permanently permits free use, modification, and distribution — directly implementing Law 5.
   - **Adoption-friendly:** Apache 2.0 is the most popular license for open-source infrastructure projects (Kubernetes, Apache projects, Ethereum, TensorFlow, Android).

2. **CC BY 4.0 for documentation:** Creative Commons licenses are designed specifically for creative and educational works, not software. CC BY 4.0 provides:
   - **Remix and translation permission:** Documentation can be translated into local languages, reformatted for local contexts, and adapted for different deployment environments — essential for global adoption.
   - **Attribution:** Derivative works must credit ZarishSphere Foundation, ensuring the ecosystem's provenance is maintained.
   - **Share-alike optional:** CC BY 4.0 does not require derivative works to use the same license (unlike CC BY-SA), allowing maximal reuse.
   - **Legal clarity:** CC BY 4.0 is internationally recognized and enforced, with ported versions in over 30 jurisdictions.

3. **Dual license clarity:** The Constitution itself carries both licenses in its footer: "Apache 2.0 (code) · CC BY 4.0 (documentation)." This pattern is applied consistently across the ecosystem. Every ADR, SOP, and specification in `zs-docs` is CC BY 4.0. Every Go file, TypeScript component, and YAML deployment config is Apache 2.0.

---

## 5. Consequences

**Positive:**

- Code can be freely adopted by commercial and government entities without legal friction — driving adoption
- Documentation can be translated, adapted, and redistributed freely — enabling global reach
- Patent protection for contributors and users (via Apache 2.0)
- Clear legal framework that is familiar to most developers and organizations
- Attribution ensures the Foundation's role is recognized even in derivative works
- No risk of GPL-related adoption barriers in government health IT procurement

**Negative:**

- Must clearly label every artifact with its applicable license (increases metadata overhead)
- Some files may blur the line between code and documentation (e.g., YAML configs are code, but README-like comments are docs) — need explicit guidance
- CC BY 4.0 attribution requirement can be burdensome for downstream projects that use large amounts of documentation
- Third-party tools that scan licenses may not handle dual-license repos correctly
- If the Foundation ever wanted to change the license, all contributors' consent would be needed (Apache 2.0 is irrevocable)

---

## 6. Status

Accepted. All existing and future repositories under the `zarishsphere` GitHub organization use this dual-license model. The license footer in every file (3-line block: Foundation name, licenses, GitHub link) implements this ADR.

---

## 7. Cross-references

→ **002-zs-foundation/003-licensing-policy.md** — Licensing policy for all ZarishSphere projects
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 5 (zero cost), Law 9 (vendor freedom), Law 12 (borderless contribution)
→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS §10 mandatory license footer

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
