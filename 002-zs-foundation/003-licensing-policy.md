# 003-licensing-policy.md
## ZarishSphere Foundation licensing policy
### Dual licensing: Apache 2.0 (code) and CC BY 4.0 (documentation)

**Document type:** Normative standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable
**GitHub:** https://github.com/zsdotcom
**Depends on:** `002-zs-foundation/001-foundation-charter.md`

---

## Table of contents

1. [Licensing philosophy](#1-licensing-philosophy)
2. [What license applies to what](#2-what-license-applies-to-what)
3. [Dual-license boundary rules](#3-dual-license-boundary-rules)
4. [Contributor licensing](#4-contributor-licensing)
5. [Third-party content](#5-third-party-content)
6. [Enforcement](#6-enforcement)
7. [Edge cases](#7-edge-cases)
8. [Cross-references](#8-cross-references)

---

## 1. Licensing philosophy

The ZarishSphere ecosystem uses two open licenses:

| License | Applies to | Why this license |
|---|---|---|
| Apache 2.0 | Code, configurations, deployment templates | Patent protection, commercial-friendly, standard for open-source infrastructure |
| CC BY 4.0 | Documentation, standards definitions, specifications | Most permissive for knowledge work, requires only attribution |

The Foundation never uses:
- GPL or AGPL (too restrictive for government and humanitarian deployers)
- Custom licenses (too confusing for contributors)
- Commons Clause or similar (defeats open-source purpose)
- Any license that creates a paid tier (violates Law 5)

## 2. What license applies to what

### 2.1 Apache 2.0 applies to

- Go source code and binaries
- JavaScript/TypeScript source code
- Python source code
- YAML and JSON configuration files
- Dockerfiles and container definitions
- Deployment scripts and workflow files
- Form definitions (XLSForm, FHIR Questionnaire)
- API specifications (OpenAPI 3.1)
- Any file executed by a runtime

### 2.2 CC BY 4.0 applies to

- All markdown documentation in `zs-docs` and all project docs repos
- ZARISH-INDEX entries (standard citations and metadata)
- ZARISH-STANDARDS transformation definitions
- Architecture Decision Records
- SOPs, runbooks, and operational guides
- README files, CHANGELOG files, CONTRIBUTING files
- Any file whose primary purpose is communication, not execution

### 2.3 Files in both categories

Some files contain both executable and documentary content. In these cases, the executable portions are Apache 2.0 and the documentary portions are CC BY 4.0:
- Markdown files with embedded YAML front matter (front matter = CC BY 4.0, YAML = Apache 2.0 if machine-read)
- YAML files with extensive comments (code = Apache 2.0, comments = CC BY 4.0)

## 3. Dual-license boundary rules

| Scenario | License | Reasoning |
|---|---|---|
| A user copies code from a ZarishSphere repository | Apache 2.0 | Code is always Apache 2.0 |
| A user copies documentation text | CC BY 4.0 | Documentation is always CC BY 4.0 |
| A user runs a ZarishSphere form in their own deployment | Apache 2.0 | Forms are executable assets |
| A user references a ZARISH-INDEX entry in their research | CC BY 4.0 | Index entries are metadata |
| A user combines ZarishSphere code with proprietary code | Apache 2.0 | Apache 2.0 permits this |
| A user modifies and redistributes documentation | CC BY 4.0 | CC BY 4.0 requires attribution |

## 4. Contributor licensing

### 4.1 Inbound = outbound

All contributions to ZarishSphere projects are licensed under the same license as the project they are contributed to. No contributor license agreement (CLA) is required. By submitting a PR, the contributor agrees that their contribution is licensed under the project's applicable license.

### 4.2 Attribution

Contributors are attributed in the repository's contributor list or git log. Documentation contributors are attributed in the document front matter where practical. Attribution must not become a barrier to contribution.

### 4.3 Copyright assignment

The Foundation does not require copyright assignment. Contributors retain copyright over their contributions and grant the Foundation a perpetual, irrevocable license to distribute them under the project's license.

## 5. Third-party content

### 5.1 Standards content

Standards indexed in ZARISH-INDEX are not owned or re-licensed by the Foundation. The index entry (citation, metadata, classification) is CC BY 4.0. The underlying standard document is governed by its original publisher's license.

### 5.2 Dependencies

All ecosystem dependencies must use OSI-approved open-source licenses. No dependency with a restrictive license (AGPL, SSPL, BUSL) may be used.

### 5.3 Templates and examples

Example code, configuration templates, and form templates are Apache 2.0 unless they include substantial documentation text, in which case the documentation portions are CC BY 4.0.

## 6. Enforcement

### 6.1 How license violations are handled

1. A violation is reported via a GitHub issue
2. The Foundation reviews and confirms the violation
3. The violating party is contacted with a 30-day cure period
4. If unresolved, the Foundation may pursue remedies under the applicable license

### 6.2 What is not a violation

- Using ZarishSphere components for any purpose (including commercial)
- Modifying and redistributing ZarishSphere components
- Combining ZarishSphere components with proprietary systems
- Forking any repository

## 7. Edge cases

| Edge case | Resolution |
|---|---|
| AI-generated code contributions | Treated as public-domain-equivalent; contributor licenses as usual |
| Translation of documentation | Translation is a derivative work under CC BY 4.0 |
| Benchmarking and comparisons | Permitted under Apache 2.0; attribution required for CC BY 4.0 portions |
| Export control | No ZarishSphere component is subject to export control; deployers are responsible for their own compliance |

## 8. Cross-references

→ **001-zs-meta/001-zarishsphere-constitution.md** — Law 5 (zero cost is a structural guarantee) and Law 12 (contribution is borderless), both encoded in this dual-license policy
→ **002-zs-foundation/001-foundation-charter.md** — Foundation obligations, including permanent zero cost
→ **002-zs-foundation/004-contributor-guidelines.md** — Inbound-equals-outbound contribution licensing in practice
→ **008-zs-adrs/008-adr-apache-cc-dual-license.md** — ADR-008: dual-licensing rules for open-source modularity
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: structural requirement for an open-source, zero-cost toolchain
→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS §10 license footer rules applied to every document

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
