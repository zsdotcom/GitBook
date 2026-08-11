---
id: "ZS-SOP-016"
title: "standard-operating-procedure-deployment-workflow"
domain: "009-zs-operations"
doc-type: "sop"
entity-type: "procedure"
description: >-
  Comprehensive Standard Operating Procedure for the ZarishSphere deployment workflow, covering GitOps pipelines, Argo CD synchronization, OpenTofu infrastructure changes, Cloudflare edge routing, and zero-downtime rollback protocols.
version: "1.0.0"
status: "stable"
tags:
  - "sop"
  - "deployment"
  - "gitops"
  - "argocd"
  - "opentofu"
isolation_tier: "global"
canonical: true
audience:
  - "contributors"
  - "ai-agents"
  - "operators"
last_updated: "2026-08-11"
---

# Standard Operating Procedure: Deployment Workflow (ZS-SOP-016)

## 1. Purpose and Scope
This Standard Operating Procedure defines the end-to-end deployment workflow for all services, microfrontends, and infrastructure within the ZarishSphere ecosystem. Per **ADR-003** (GitHub as Government) [1] and **ADR-017** (Argo CD GitOps) [2], all state changes—whether infrastructure provisioning via OpenTofu or container deployments via Argo CD—must originate from version-controlled Git repositories and be applied declaratively.

This document applies to all system operators, automated CI/CD agents, and contributors managing production and staging environments.

## 2. Prerequisites and Tooling
Before initiating any deployment or infrastructure change, operators must ensure access to the following approved toolchain:
- **Git & GitHub CLI (`gh`)**: For code review, PR merging, and tagging releases.
- **OpenTofu (v1.12.5)**: For infrastructure provisioning and state management (**ADR-016**) [3].
- **Argo CD (v3.4.6)**: For continuous delivery and synchronization of Kubernetes manifests (**ADR-017**) [2].
- **Cilium (v1.20.0)**: For service mesh traffic routing and network security policies (**ADR-018**) [4].

## 3. Deployment Pipeline Phases

### Phase 1: Branching and Code Review
1. All changes must be committed to feature branches and submitted via Pull Request to the main branch of the respective repository (e.g., `zsdotcom/GitBook` or core service repositories).
2. Automated checks (linting, type checking, unit tests) must pass successfully.
3. At least one human review or approved AI agent verification is required per ZUSS compliance rules.

### Phase 2: Infrastructure Changes (OpenTofu)
For any changes affecting cloud resources, databases, or networking topology:
1. Navigate to the infrastructure repository.
2. Initialize and validate the workspace:
   ```bash
   tofu init
   tofu plan -out=tfplan
   ```
3. Review the execution plan to ensure zero unintended resource destruction.
4. Apply the plan upon approval:
   ```bash
   tofu apply tfplan
   ```

### Phase 3: Application Synchronization (Argo CD)
For containerized workloads and microfrontends:
1. Merge approved code changes into the `main` release branch.
2. Argo CD continuously monitors the Git repository and detects manifest updates in the deployment configuration path.
3. Trigger manual synchronization if automated sync is disabled:
   ```bash
   argocd app sync zs-core-application
   ```
4. Verify application health status and endpoint responsiveness:
   ```bash
   argocd app get zs-core-application
   ```

## 4. Rollback and Incident Response
If a deployment introduces critical regressions or health check failures:
1. **Immediate Reversion**: Revert the offending commit in Git or rollback the Argo CD application deployment to the previous healthy revision:
   ```bash
   argocd app rollback zs-core-application <PREVIOUS_REVISION_ID>
   ```
2. **Infrastructure Rollback**: If OpenTofu state was incorrectly applied, revert the infrastructure commit and re-apply the prior state configuration.
3. **Post-Incident Review**: File an incident report per **ZS-SOP-010** (Incident Response) within 24 hours.

## 5. References
[1] [003-adr-github-as-government.md](../008-zs-adrs/003-adr-github-as-government.md) - GitHub as the sole operational control plane.  
[2] [017-adr-argocd-gitops.md](../008-zs-adrs/017-adr-argocd-gitops.md) - Argo CD as the GitOps deployment engine.  
[3] [016-adr-opentofu-infrastructure-as-code.md](../008-zs-adrs/016-adr-opentofu-infrastructure-as-code.md) - OpenTofu infrastructure as code.  
[4] [018-adr-cilium-service-mesh.md](../008-zs-adrs/018-adr-cilium-service-mesh.md) - Cilium service mesh and network security.
