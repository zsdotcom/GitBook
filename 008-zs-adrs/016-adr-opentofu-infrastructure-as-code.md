# 016-adr-opentofu-infrastructure-as-code.md
## ADR-016: OpenTofu 1.12.5 for Infrastructure as Code
### MPL-2.0 Licensed, Linux Foundation Governance, Terraform-Compatible

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

Use OpenTofu 1.12.5 as the Infrastructure as Code (IaC) tool for defining and managing all ZarishSphere Platform infrastructure. All Kubernetes clusters, PostgreSQL deployments, NATS instances, Cloudflare DNS records, email relay configurations, and Keycloak 26.2.7 identity services are defined as OpenTofu configurations. OpenTofu is the Linux Foundation's community-maintained fork of Terraform under MPL-2.0 license, adopted after HashiCorp migrated Terraform to the BSL (Business Source License) in 2023.

---

## 2. Context

The ZarishSphere Platform infrastructure must be defined as code and reproducible from a clean environment. The IaC tool must satisfy several constraints:

- **Zero cost forever:** The tool must be free for all users including organizations that build commercial services on ZarishSphere. Constitution Law 5 (zero cost) and ADR-006 require unconditional free use with no licensing restrictions.
- **Constitution Law 9 (vendor freedom):** The tool must not create vendor lock-in. Infrastructure definitions must be portable to other IaC tools or deployment methods.
- **Community governance:** The IaC tool should be community-governed under a neutral foundation to prevent single-vendor license changes that could affect the ecosystem. HashiCorp's 2023 BSL license change for Terraform demonstrated this risk.
- **Terraform module compatibility:** The ecosystem may need to consume existing Terraform modules for common infrastructure patterns (Cloudflare DNS, Kubernetes manifests, PostgreSQL provisioning). The chosen tool must support HCL (HashiCorp Configuration Language) and existing Terraform providers.
- **Single-founder maintainable:** The tool must have a manageable learning curve — the single founder must be able to write, review, and maintain infrastructure configurations without specialized DevOps expertise.
- **Plane 0 and Plane 1 compatible:** IaC configuration must work in air-gapped and offline environments where the infrastructure tool cannot download providers from the internet.

---

## 3. Alternatives Considered

- **OpenTofu 1.12.5:** MPL-2.0 licensed under Linux Foundation governance. 1:1 fork of Terraform — all existing Terraform providers, modules, and HCL configurations work without modification. Active community development with features beyond Terraform OSS (including provider caching for air-gapped deployments, early variable/locals evaluation, and state file encryption). No licensing restrictions — MPL-2.0 permits free use by any organization for any purpose, including commercial managed services. Linux Foundation governance ensures no single vendor controls the project's license or direction.

- **Terraform (BSL):** Largest IaC ecosystem, most providers and modules, best documentation, most community knowledge. However: HashiCorp's 2023 BSL license change restricts use in competitive offerings, creates legal uncertainty for organizations providing infrastructure services, and violates the precautionary principle of ADR-006. The BSL conversion to MPL-2.0 after 4 years does not solve the immediate licensing concern for a platform designed to outlive its creators (Constitution Law 11). Terraform OSS (pre-BSL) is stuck on version 1.5.x with no future updates.

- **Pulumi:** Modern IaC tool that allows infrastructure definition in general-purpose programming languages (TypeScript, Python, Go, C#, Java). Strong type safety, real programming language features (loops, conditionals, functions), and excellent testing support. However: requires expertise in TypeScript or Python (additional skill requirement for the single founder), not HCL-compatible (existing Terraform modules cannot be consumed directly), BSL-licensed for the core product with enterprise features requiring a license, and younger ecosystem with fewer providers. The TypeScript/Python dependency conflicts with ADR-001's Go-primary decision for backend tooling.

- **AWS CDK / CloudFormation:** Deep AWS integration with infrastructure defined in TypeScript/Python/Java. However: AWS-specific (violates ADR-009 no vendor lock-in — infrastructure would be tied to AWS), not portable to other cloud providers or on-premise deployments, CloudFormation is JSON/YAML-only with limited abstraction, and both conflict with Plane 0/1 air-gapped deployments that may not use AWS at all.

- **Ansible:** Agentless configuration management and automation tool. Simple YAML syntax, large community, extensive module library. However: Ansible is a configuration management tool, not an IaC state management tool — it does not maintain desired state or detect drift in the way Terraform/OpenTofu do. No built-in state file, no resource graph, no plan/apply workflow. Suitable for server configuration but not for infrastructure resource lifecycle management.

- **Manual kubectl / Helm / Cloud Console:** No IaC tooling — infrastructure created and modified manually through CLI or web console. Simplest initial approach but violates Constitution Law 10 (Every decision is auditable forever) — manual changes are not recorded in git, not reproducible, and prone to drift. Inconsistent with the GitHub-as-government model (Constitution Law 1) where every change should be a git commit.

---

## 4. Reason for Decision

1. **License certainty after HashiCorp BSL change:** HashiCorp's August 2023 license change from MPL-2.0 to BSL demonstrated that commercially-owned open-source projects can change licensing terms in ways that restrict downstream users. OpenTofu, under Linux Foundation governance, provides permanent license certainty. This is essential for Constitution Law 11 (The platform outlives its creators) — infrastructure tooling must remain free for any deployer, forever.

2. **Terraform ecosystem compatibility:** OpenTofu is a 1:1 compatible fork. All existing Terraform providers (Cloudflare, Kubernetes, Helm, PostgreSQL, Keycloak) and modules can be used without modification. This preserves access to the largest IaC ecosystem while providing better licensing terms. Migration is a `terraform` → `tofu` command alias change.

3. **Air-gapped deployment support:** OpenTofu 1.12.5 adds provider caching and offline plugin management that are essential for Plane 0 and Plane 1 deployments where internet access may be unavailable or unreliable. Infrastructure must be provisionable in environments where provider registries cannot be reached.

4. **Single-founder maintainable:** OpenTofu/Terraform HCL is a declarative language with a relatively gentle learning curve. The single founder can write, review, and maintain infrastructure definitions without specialized DevOps training. This contrasts with Pulumi, which requires proficiency in TypeScript or Python, and Ansible, which uses a different conceptual model (configuration management vs. state management).

5. **GitOps compatibility:** OpenTofu configurations are plain HCL files in git repositories, which integrates naturally with the GitHub-as-government model (Constitution Law 1). Combined with Argo CD (ADR-017) for Kubernetes GitOps, the entire infrastructure pipeline flows from git commits.

---

## 5. Consequences

**Positive:**

- Permanent MPL-2.0 license with Linux Foundation governance — no license uncertainty
- Full compatibility with existing Terraform providers, modules, and HCL configurations
- Provider caching enables air-gapped infrastructure provisioning (Plane 0/1)
- HCL is declarative and maintainable by a single founder
- Infrastructure changes flow through git — auditable, reviewable, revertable (Constitution Law 10)
- State file encryption support for sensitive infrastructure
- Active community with regular releases and growing feature set
- All infrastructure is reproducible from scratch from committed configurations

**Negative:**

- Smaller ecosystem than Terraform — fewer community modules, less Stack Overflow knowledge
- Some bleeding-edge Terraform providers may lag in OpenTofu compatibility
- Team behind OpenTofu is smaller than HashiCorp's Terraform team — feature development pace may differ
- Migration from any existing Terraform configurations requires `s/terraform/tofu/g` and state migration
- Cloud providers optimize for Terraform — some new provider features may arrive on Terraform first
- HCL is less expressive than general-purpose languages for complex infrastructure logic

---

## 6. Status

Accepted. OpenTofu 1.12.5 is the Infrastructure as Code tool for all ZarishSphere Platform infrastructure. All infrastructure resources (Kubernetes, databases, messaging, DNS, email, identity) must be defined as OpenTofu configurations committed to git. Manual infrastructure changes are permitted only for emergency break-glass scenarios and must be followed by an OpenTofu configuration update within 24 hours.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **008-zs-adrs/001-adr-go-as-primary-language.md** — ADR-001: Go as the primary backend language (Pulumi conflicts)
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: zero-cost toolchain structural requirement
→ **008-zs-adrs/009-adr-no-vendor-lock-in.md** — ADR-009: no vendor lock-in (rules out AWS CDK/CloudFormation)
→ **008-zs-adrs/017-adr-argocd-gitops.md** — ADR-017: Argo CD GitOps deployment engine (GitOps compatibility)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
