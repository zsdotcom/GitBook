# 004-contributor-guidelines.md
## ZarishSphere Foundation contributor guidelines
### How anyone can contribute to any project

**Document type:** Normative standard
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable
**GitHub:** https://github.com/zsdotcom
**Depends on:** `002-zs-foundation/001-foundation-charter.md`

---

## Table of contents

1. [Who can contribute](#1-who-can-contribute)
2. [Types of contribution](#2-types-of-contribution)
3. [Contribution process](#3-contribution-process)
4. [Communication channels](#4-communication-channels)
5. [Expectations](#5-expectations)
6. [Recognition](#6-recognition)
7. [Code of conduct](#7-code-of-conduct)
8. [Cross-references](#8-cross-references)

---

## 1. Who can contribute

Anyone can contribute to any ZarishSphere project. No special status, membership, or approval is required to open an issue, submit a PR, or participate in discussions.

Contributors fall into three categories:

| Category | Description | Rights |
|---|---|---|
| First-time contributor | Anyone submitting their first PR | Can open issues and PRs; PRs reviewed by maintainers |
| Regular contributor | Multiple accepted PRs | Can be added to `CONTRIBUTORS.md`; trusted for self-reviewed doc fixes |
| Maintainer | Consistent, high-quality contributions over time | Can review and merge PRs; may be invited to governance bodies |

## 2. Types of contribution

### 2.1 Technical contributions

- Code: Go, JavaScript, Python, or any language in the ecosystem
- Documentation: Markdown files in any repository
- Standards: ZARISH-INDEX entries, ZARISH-STANDARDS transformations
- Forms: XLSForm, FHIR Questionnaire, or form templates
- Modules: Domain module implementations
- Apps: Pre-built application templates
- Distributions: Deployment bundle definitions
- Tests: Validation scripts, test cases, compliance checks

### 2.2 Non-technical contributions

- Issue reporting: Bug reports, feature requests, documentation gaps
- Review: Reading and commenting on PRs, ADRs, RFCs
- Translation: Translating documentation or interfaces
- Domain expertise: Reviewing standards entries for accuracy
- Design: Interface mockups, user experience feedback
- Outreach: Spreading awareness of the ecosystem

### 2.3 What is not a contribution

- Spam, self-promotion, or unrelated content
- Offensive or discriminatory communication
- Violation of any project license

## 3. Contribution process

| Step | Technical contribution | Non-technical contribution |
|---|---|---|
| 1 | Fork the repository | Open an issue or join discussion |
| 2 | Create a branch (`fix/`, `feat/`, `docs/`) | Provide detailed information |
| 3 | Make changes following ZUSS | Do not require a PR |
| 4 | Submit a PR with clear description | Community responds in thread |
| 5 | Address reviewer feedback | |
| 6 | PR is merged | |

### 3.1 PR requirements

- PR title must describe the change (not `Update file.md`)
- PR description must explain why the change is needed
- Documentation PRs must include before/after if changing content
- Code PRs must include tests or explain why tests are not needed
- Cross-reference changes must validate that references resolve

### 3.2 Review standards

| Contribution type | Review required | Review timeframe |
|---|---|---|
| Typo fix | Single maintainer | 24 hours |
| Documentation update | Single maintainer | 72 hours |
| New specification | Two reviewers or 7-day public comment | 7 days |
| Code change | Single maintainer + automated checks | 72 hours |
| ADR / RFC | 30-day public comment period | 30 days |

## 4. Communication channels

| Channel | Purpose |
|---|---|
| GitHub Issues | Bug reports, feature requests, questions |
| GitHub Discussions | General discussion, design conversations |
| GitHub PRs | Code and documentation review |

All communication is public and persistent. No private discussions about project direction, decisions, or disputes.

## 5. Expectations

### 5.1 For contributors

- Follow ZUSS naming and formatting rules (see → **001-zs-meta/004-zarishsphere-writing-rules.md**)
- Respect the constitution's 12 laws
- Be respectful and constructive in all interactions
- Accept that the founder has final decision authority during single-founder phase

### 5.2 For maintainers

- Respond to PRs and issues within the review timeframe
- Provide constructive feedback, not just rejection
- Explain why changes are accepted or rejected
- Keep the contributor informed about their contribution's status

### 5.3 What contributors should not expect

- Immediate response (Foundation is single-founder with limited bandwidth)
- Guaranteed acceptance of every PR
- Payment for contributions (all contributions are voluntary)

## 6. Recognition

| Contribution level | Recognition |
|---|---|
| First PR merged | Listed in `CONTRIBUTORS.md` |
| 5+ PRs merged | Listed with contribution area |
| 20+ PRs merged | Invited to maintainer discussions |
| Consistent quality over 6+ months | Considered for governance body nomination |

## 7. Code of conduct

### 7.1 Expected behavior

- Be respectful and inclusive
- Assume good faith
- Provide constructive feedback
- Accept responsibility for mistakes
- Focus on what is best for the ecosystem

### 7.2 Unacceptable behavior

- Harassment, discrimination, or personal attacks
- Trolling, insults, or derogatory comments
- Publishing others' private information
- Other conduct that could reasonably be considered inappropriate

### 7.3 Enforcement

Violations should be reported via GitHub issue to the repository maintainer. During the single-founder phase, the founder is the final authority on conduct enforcement. Consequences range from a warning to a permanent ban from all Foundation repositories.

## 8. Cross-references

→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS naming and formatting rules every contribution must follow
→ **001-zs-meta/001-zarishsphere-constitution.md** — The 12 laws that constrain all contributions
→ **002-zs-foundation/001-foundation-charter.md** — Foundation mission and obligations
→ **002-zs-foundation/002-governance-model.md** — Decision processes and review standards that govern PR acceptance
→ **002-zs-foundation/003-licensing-policy.md** — Inbound-equals-outbound licensing for every contribution
→ **009-zs-operations/003-sop-contribution-process.md** — Review pipeline and external patch merge processing

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
