---
id: "ZS-SOP-017"
title: "standard-operating-procedure-incident-response-cilium-valkey"
domain: "009-zs-operations"
doc-type: "sop"
entity-type: "procedure"
description: >-
  Standard Operating Procedure for diagnosing and resolving production incidents related to Cilium service mesh routing and Valkey caching clusters, referencing ADR-015 and ADR-018.
version: "1.0.0"
status: "stable"
tags:
  - "sop"
  - "incident-response"
  - "cilium"
  - "valkey"
  - "caching"
  - "service-mesh"
isolation_tier: "global"
canonical: true
audience:
  - "contributors"
  - "ai-agents"
  - "operators"
last_updated: "2026-08-11"
---

# Standard Operating Procedure: Incident Response for Cilium Service Mesh & Valkey Caching (ZS-SOP-017)

## 1. Purpose and Scope
This Standard Operating Procedure establishes rapid diagnosis and remediation workflows for critical incidents affecting the **Cilium Service Mesh** (**ADR-018**) [1] and **Valkey Caching Layer** (**ADR-015**) [2]. Because these components govern inter-service communication, security policies, and high-performance data caching across the ZarishSphere ecosystem, prompt escalation and structured troubleshooting are mandatory.

## 2. Cilium Service Mesh Incidents

### 2.1 Common Failure Modes
- **Network Policy Drops**: Egress/ingress connectivity blocked due to overly restrictive NetworkPolicies.
- **eBPF Map Pressure**: BPF map entry exhaustion causing dropped packet flows.
- **Hubble Relay Disconnections**: Loss of observability across microservices.

### 2.2 Diagnostic Commands
1. Check Cilium agent health across nodes:
   ```bash
   kubectl -n kube-system rollout status daemonset/cilium
   ```
2. Inspect dropped packet flows using Hubble CLI:
   ```bash
   hubble observe --verdict DROPPED
   ```
3. Verify BPF map limits and utilization:
   ```bash
   cilium bpf metrics list
   ```

### 2.3 Remediation Protocol
1. If a newly applied NetworkPolicy causes service isolation, temporarily rollback the policy commit in Git and sync via Argo CD (**ZS-SOP-016**) [3].
2. If BPF map exhaustion occurs, flush stale entries or adjust map sizing parameters in the OpenTofu cluster configuration (**ADR-016**) [4].

---

## 3. Valkey Caching Layer Incidents

### 3.1 Common Failure Modes
- **Memory Eviction Storms**: High write volume exceeding maxmemory limits leading to heavy cache misses.
- **Replication Lag / Split-Brain**: Primary-replica synchronization failure in clustered deployments.
- **Connection Pool Exhaustion**: Microservice clients exhausting maxclients limits.

### 3.2 Diagnostic Commands
1. Check Valkey server health and memory statistics:
   ```bash
   valkey-cli -h valkey.internal ping
   valkey-cli -h valkey.internal info memory
   ```
2. Monitor connected clients and eviction rates:
   ```bash
   valkey-cli -h valkey.internal info stats
   ```

### 3.3 Remediation Protocol
1. If memory limits are breached, trigger safe key eviction or scale memory allocations via OpenTofu state update.
2. If primary node failure occurs, verify automatic failover and promote replica nodes if replication timeout exceeds 30 seconds.

## 4. References
[1] [018-adr-cilium-service-mesh.md](../008-zs-adrs/018-adr-cilium-service-mesh.md) - Cilium service mesh and network security.  
[2] [015-adr-valkey-for-caching.md](../008-zs-adrs/015-adr-valkey-for-caching.md) - Valkey caching layer replacement.  
[3] [016-sop-deployment-workflow.md](016-sop-deployment-workflow.md) - Deployment workflow SOP.  
[4] [016-adr-opentofu-infrastructure-as-code.md](../008-zs-adrs/016-adr-opentofu-infrastructure-as-code.md) - OpenTofu infrastructure as code.
