# 011-adr-privacy-by-architecture.md
## ADR-011: Individual Surveillance Is Structurally Impossible
### All data architectures enforce privacy at the infrastructure layer, not by policy alone

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

All ZarishSphere platform data architectures enforce privacy at the infrastructure layer. No individual health, administrative, or personal data may flow to any server outside the deploying entity's infrastructure without explicit, documented, revocable consent from both the individual and the deploying organization. Passive individual tracking at the network, application, or database layer is structurally impossible.

This is enforced by:

1. **Local-first data sovereignty** — All data is created, stored, and processed on the deployer's infrastructure by default. No outbound data transmission occurs without explicit consent.
2. **Consent-gated outbound flow** — Any data that leaves the deploying entity's infrastructure must pass through a documented consent mechanism that records individual consent, organizational consent, purpose, scope, and expiration.
3. **Module code isolation** — No module may access data owned by another module without explicit, auditable cross-module data sharing enabled by the deploying organization.
4. **Emergency key destruction** — The ability to render individual data permanently unreadable must be technically implementable within 60 seconds on any deployment plane, including Plane 0 (air-gapped).
5. **No telemetry by default** — No ZarishSphere deployment transmits telemetry, usage data, error reports, analytics, or any information to zarishsphere.com or any ZarishSphere-operated server without explicit opt-in.

---

## 2. Context

The ZarishSphere ecosystem is designed for deployment in contexts where privacy is not optional but a matter of safety:

- **Humanitarian settings** — Refugee camps (Cox's Bazar), conflict zones, disaster response — where individual health data in the wrong hands can lead to persecution, discrimination, or violence
- **Health facilities** — Patient health records protected by medical confidentiality, data protection regulations (GDPR, HIPAA, local laws)
- **Government institutions** — Citizen data subject to national data protection frameworks
- **Resource-constrained environments** — Where privacy cannot rely on legal enforcement alone

Constitution Law 4 (Identity without surveillance) mandates:

> Every person served by the ZarishSphere Platform is protected by architecture, not by policy. The technical design makes individual surveillance structurally impossible — not merely prohibited by policy or terms of service. Individual health, administrative, and personal data may not flow to any server outside the deploying entity's infrastructure without explicit, documented, revocable consent from both the individual and the deploying organization.

Target deployments explicitly require this architecture:

- Plane 0 (air-gapped) deployments must have zero external data flow by default
- Plane 1 (Raspberry Pi / local network) deployments must have opt-in-only outbound data
- Module isolation prevents cross-module data access without organizational consent
- Consent mechanism must work offline and sync when reconnected

The single-founder context makes architecture-level enforcement essential. With no team to review every data access pattern, privacy guarantees must be structural — impossible to violate even unintentionally.

---

## 3. Alternatives Considered

| Alternative | Pros | Cons |
|---|---|---|
| **Policy-based privacy** (privacy enforced by written terms, not architecture) | Simpler to implement; faster initial development; no architectural constraints | Violates Law 4 structurally; unenforceable in practice; single founder cannot audit every data path; relies on human compliance |
| **Opt-out architecture** (data shared by default, users must opt out) | Easier data aggregation; simpler analytics pipeline; common SaaS pattern | Violates informed consent; creates surveillance architecture by default; unacceptable in humanitarian contexts; violates Law 4 |
| **Federated architecture with local sovereignty (chosen)** | Law 4 compliant; Plane 0 compatible; audit-friendly; consent mechanism provides verifiable trail; module isolation prevents cross-module leaks; emergency key destruction works on all planes | More complex sync mechanism; no central data warehouse for unified analytics; consent management adds UI and storage overhead; offline consent requires local cryptographic storage |
| **Centralized architecture** (all data in one server) | Simplest to build; easy analytics; single backup/restore point; conventional architecture pattern | Single point of surveillance; violates Law 4; unsuitable for refugee, health, and government deployments; creates unacceptable risk profile |
| **Client-side encryption only** (data encrypted at rest on device, server never sees plaintext) | Strong privacy guarantees; server cannot access data even if compromised | Cannot perform server-side processing (search, analytics, aggregation); complex key management; loss of device = loss of data; incompatible with multi-user workflows |

---

## 4. Reason for Decision

1. **Constitutional mandate:** Law 4 is a Tier II (Rights) law. It requires that individual surveillance be structurally impossible — not merely prohibited by policy. This ADR translates that constitutional requirement into specific architectural constraints that apply to every module, service, and deployment plane.

2. **Humanitarian deployment context:** The primary deployment contexts (refugee camps, conflict zones, health facilities) serve populations for whom privacy is a safety issue, not a convenience. A data breach in Cox's Bazar could lead to real-world harm. Policy-based privacy is insufficient protection.

3. **Single-founder constraint:** With a single developer building the platform, there is no team to review every data flow for privacy compliance. The architecture must make privacy violations structurally impossible — not just detect them after they occur.

4. **Local-first data sovereignty:** Plane 0 (air-gapped) deployments are a constitutional requirement (Law 8 — Deployment plane sovereignty). The platform must be fully functional with zero network connectivity. Privacy-by-architecture is the natural consequence: if data never leaves, surveillance is structurally impossible.

5. **Emergency preparedness:** The 60-second emergency key destruction requirement (derived from Law 4) ensures that even in worst-case scenarios (device confiscation, facility takeover, natural disaster), individual data can be rendered permanently unreadable. This is a design requirement that must be built in from the start.

---

## 5. Consequences

**Positive:**

- Individual surveillance is structurally impossible — not just contractually prohibited
- Plane 0 (air-gapped) deployments have zero external data flow by default, satisfying Law 8
- All outbound data has a consent audit trail (who consented, when, for what purpose, until when)
- Module code isolation prevents cross-module data access without explicit organizational approval
- Emergency key destruction is implementable on every deployment plane within 60 seconds
- Deploying organizations can demonstrate verifiable privacy compliance to regulators and communities
- Single founder cannot accidentally create surveillance architecture — the constraints prevent it

**Negative:**

- No central analytics warehouse: aggregate analytics require local aggregation at each deployment with periodic opt-in sharing
- Consent management adds architectural complexity: consent records, expiration tracking, revocation handling, offline consent storage
- Cross-module data sharing requires explicit organizational approval flow (cannot be assumed)
- Data portability across deployments requires explicit consent and purpose specification
- Sync mechanisms are more complex because they must respect consent gates on every data field
- Some users familiar with centralized SaaS platforms may find the local-first model unfamiliar

---

## 6. Status

Accepted. This ADR implements Constitution Law 4 (Identity without surveillance) and applies to all ZarishSphere Platform data architectures, modules, services, and deployment planes. It is a Tier II-consistent engineering standard and may not be overridden by any lower-tier decision. This ADR applies retroactively to all existing module and service specifications and prospectively to all new component designs.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 4 (identity without surveillance)
→ **003-zs-platform/003-deployment-planes.md** — Deployment plane model (Plane 0/1 offline constraints)
→ **006-zs-infrastructure/007-security-architecture.md** — Security architecture (encryption, key management)
→ **006-zs-infrastructure/008-security-policies.md** — Security policies and controls
→ **006-zs-infrastructure/010-threat-models.md** — Threat models covering data flow risks

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
