# 012-adr-no-single-person-dependency.md
## ADR-012: The Platform Outlives Its Creators
### Every credential, key, and operational function has a documented succession path

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

No feature, workflow, critical function, API key, credential, or operational process in the ZarishSphere ecosystem may depend on a single person's access, knowledge, or continued availability.

This is enforced by:

1. **Documented succession for every credential** — Every API key, service credential, SSH key, cloud access token, GitHub token, email credential, and deployment secret must have a documented rotation and succession process committed to the repository. The process must specify who can rotate it, how it is rotated, and what happens if the current holder is unavailable.

2. **Knowledge committed as documentation** — Every function that requires human knowledge (deployment procedures, configuration details, troubleshooting steps, recovery processes, domain management, DNS configuration, CI/CD pipeline maintenance) must have that knowledge committed as a document. No undocumented knowledge may be required to operate any production system.

3. **Encrypted CI/CD secret store** — All base secrets, credentials, and configuration are stored as encrypted CI/CD variables (GitHub Actions secrets or equivalent), not in personal accounts, personal password managers, or undocumented local files. Credentials stored only in the founder's personal accounts are a violation of this ADR.

4. **No undocumented single-owner resources** — Every cloud resource, domain registration, DNS zone, API integration, email relay, and third-party service account must have documented ownership and succession information in the repository. No resource may be owned by a single individual's personal account without a documented fallback path.

5. **Obituary/transition document** — A documented transition procedure describes how the ecosystem continues operation if the founder becomes permanently unavailable. This includes:
   - How repositories are transferred or forked
   - How domain names and DNS are managed
   - How CI/CD pipelines continue to operate
   - How the community maintains the projects
   - How deployments in progress are completed

---

## 2. Context

The ZarishSphere Foundation is a single-founder project. Mohammad Ariful Islam is the sole builder, architect, and operator of the entire ecosystem. Constitution Law 11 (The platform outlives its creators) mandates:

> No feature, workflow, critical function, API key, credential, or operational process may depend on a single person's access, knowledge, or continued availability. Every function that requires human knowledge must have that knowledge committed as a document. Every credential must have a documented rotation and succession process. Every access right must have a documented succession path.

The constraint on this law explicitly states:

> Any system component that requires the founder's GitHub account, personal API key, personal credential, or undocumented knowledge to function in production violates this law. Succession documentation is a constitutional requirement, not an administrative nicety.

Key realities that drive this decision:

- **Bus-factor of 1:** The founder is the only person who knows how all components fit together, where credentials are stored, how the CI/CD pipeline is configured, and how deployments work
- **Humanitarian deployments demand continuity:** Target deployments (refugee camps, health facilities, government institutions) require multi-year operational continuity. A platform that fails because its creator is unavailable is unacceptable
- **No team exists to absorb knowledge:** Unlike an organization with multiple engineers, there is no shared institutional knowledge, no code review, no peer backup
- **Credentials exist across multiple services:** GitHub, Cloudflare, domain registrars, email services, cloud providers — each is a potential single point of failure if only one person has access
- **V1 development phase is the riskiest time:** During early development, it is tempting to take shortcuts and store credentials only locally "until launch." This ADR requires succession design from Day 1

The founder profile (§5.3) confirms the context: "V1 development — starting from zero. No team — Ariful is the sole builder."

---

## 3. Alternatives Considered

| Alternative | Pros | Cons |
|---|---|---|
| **Single-person ops** (founder manages everything personally, credentials in personal accounts, undocumented knowledge) | Simplest approach; no process overhead; fastest development velocity | Violates Law 11; bus-factor of 1; ecosystem becomes non-functional if founder is unavailable; no path for community continuation; unacceptable for humanitarian deployments |
| **Shared team access** (multiple trusted people have all credentials) | Shared responsibility; knowledge distributed across team members; multiple people can operate deployment | No team exists yet; solo founder project for V1; recruiting keyholders is premature and introduces security risks before the project has governance structure; credentials would still need documentation for the original holder |
| **Automated succession with documented secrets (chosen)** | Fully Law 11 compliant; zero knowledge lost if founder is unavailable; automated credential rotation possible; CI/CD variables provide encrypted storage; documented succession path for every resource; transition document enables ecosystem continuation | Documentation overhead; requires discipline to maintain current runbooks; founder must document work as it is done (cannot defer); encrypted secrets must be rotated to prevent single-holder access |
| **Open-source community keyholders** (trusted community members hold backup credentials, split-key crypto) | Distributes trust across multiple people; no single point of compromise; community governance model | Premature for V1; no community exists yet; keyholder selection requires governance the project does not yet have; increases security surface area; founder must train keyholders |
| **Hardware security module / dedicated secrets vault** (physical HSM or cloud vault with MFA) | Strongest security for credentials; audit trail for all access; automated rotation | Expensive (HSMs); requires cloud service (vault) that may not be zero-cost; adds operational complexity for single-founder project; vault itself becomes a single point of failure if improperly configured |

---

## 4. Reason for Decision

1. **Constitutional mandate:** Law 11 is a Tier IV (Governance) law with direct force over all operational processes. It requires that the platform remain functional, forkable, and fully deployable by any person on earth without requiring any individual's participation — including the founder. This ADR implements Law 11 as an operational engineering standard.

2. **Single-founder project must design for founder absence from Day 1:** With a bus-factor of 1, every undocumented credential, every locally stored API key, every unrecorded configuration detail is a potential project-ending failure. The only way to guarantee continuity is to document everything and store secrets where they can be accessed without the founder.

3. **Humanitarian deployments require continuity guarantees:** The target deployment contexts (refugee camps, health facilities, government institutions) serve populations that depend on the platform for essential services. These deployments must continue operating for years — beyond the availability of any single person. A health worker in Cox's Bazar cannot wait for the founder to recover availability.

4. **Automated processes do not have emergencies:** Documented credentials in CI/CD pipelines, documented procedures in runbooks, and documented succession paths do not get sick, take leave, disappear, or face personal emergencies. By moving operational knowledge from the founder's memory to the repository, the ecosystem achieves operational continuity that no human-dependent system can match.

5. **Community forkability:** If the Foundation ever ceases to operate, the ecosystem must be forkable by any person or organization. Documented credentials and procedures are what make a fork viable. Without this ADR, the ecosystem is a single-person artifact. With it, it is infrastructure.

---

## 5. Consequences

**Positive:**

- Continuity guaranteed — the ecosystem does not depend on any single person's availability
- No bus-factor vulnerability — every credential and procedure is documented
- Documented succession paths for every critical resource (domains, DNS, GitHub org, Cloudflare, email, CI/CD)
- Encrypted CI/CD secret store provides secure, documented credential management without personal account dependency
- Transition/obituary document enables community continuation if needed
- Law 11 compliance verified through documented procedures
- New contributors can understand and operate the system without the founder's direct involvement
- Automated credential rotation is possible because credentials are managed through documented processes

**Negative:**

- Documentation overhead: every procedure, credential location, and configuration detail must be documented as it is created — no deferring documentation "until later"
- Maintaining current runbooks requires discipline: every time a process changes, the corresponding documentation must be updated
- No single-person shortcuts allowed even during development: the founder cannot take the faster path of storing a credential locally "just for now" — it must go through the documented process from Day 1
- Encrypted CI/CD secrets management adds a small operational step for each new credential
- Self-discipline is the only enforcement mechanism in V1 (no team to review compliance)
- Transition document requires confronting the possibility of personal unavailability, which is psychologically uncomfortable but constitutionally required

---

## 6. Status

Accepted. This ADR implements Constitution Law 11 (The platform outlives its creators) and applies to all ZarishSphere ecosystem credentials, operational processes, API keys, deployment configurations, and third-party service accounts. Compliance is verified through documented succession paths committed to the repository. Any new service, credential, or operational process added to the ecosystem must include a documented succession plan before it is used in production.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 11 (platform outlives its creators)
→ **001-zs-meta/003-zarishsphere-founder-profile.md** — Founder profile (§5.3: single-founder context)
→ **006-zs-infrastructure/002-github-org-architecture.md** — GitHub org access and secret management
→ **006-zs-infrastructure/006-ci-cd-architecture.md** — CI/CD pipelines (secrets, deployment continuity)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
