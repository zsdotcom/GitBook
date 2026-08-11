# 018-adr-cilium-service-mesh.md
## ADR-018: Cilium 1.20.0 as Kubernetes Networking, Security, and Observability Layer
### eBPF-Native, ~60% Less Overhead Than Istio, CNCF Graduated, No Sidecar Required

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

Use Cilium 1.20.0 as the Kubernetes Container Network Interface (CNI), network policy enforcer, service mesh, and observability layer for all ZarishSphere Platform deployments. Cilium uses eBPF (extended Berkeley Packet Filter) to provide networking, load balancing, security policies, and observability without sidecar proxies. Hubble provides flow-level observability. This replaces both a traditional CNI (Calico, Flannel) and a separate service mesh (Istio, Linkerd).

---

## 2. Context

ZarishSphere Platform microservices require network-level capabilities across several dimensions:

- **Zero-trust network security:** All inter-service communication must be authenticated and encrypted regardless of network location. No service should implicitly trust another service based on network adjacency. This is essential for clinical data protection (Constitution Law 4 — Privacy by architecture).
- **Network policies:** Granular allow/deny rules at the pod level — which services can communicate with which other services, on which ports, over which protocols. Policies must be auditable and version-controlled (Constitution Law 10).
- **Observability:** Flow-level visibility into inter-service communication — which services talk to which other services, with what latency, error rates, and bandwidth. This is essential for debugging, capacity planning, and performance monitoring.
- **Low resource overhead:** Plane 1 (Raspberry Pi) deployments run on resource-constrained hardware (8 GB RAM shared across all services). The network layer must not consume resources that could otherwise serve clinical workloads.
- **Encryption in transit:** All clinical data must be encrypted between services — mTLS or equivalent. This is a baseline requirement for any health data platform.
- **Multi-cluster support:** Plane 3 (national cloud) and Plane 4 (global SaaS) may span multiple Kubernetes clusters. Service mesh must support cross-cluster communication.

---

## 3. Alternatives Considered

- **Cilium 1.20.0:** eBPF-native networking with no sidecar proxies. Uses eBPF for load balancing, network policy enforcement, and observability — operates at the kernel level rather than through a userspace proxy. Hubble provides rich flow-level observability (service dependencies, latency, TCP flow metrics) through the same eBPF data path. NetworkPolicy enforcement is native Kubernetes-compatible with Cilium-specific extensions for DNS-aware policies, HTTP-aware policies, and cluster mesh. Resource overhead is negligible (~5-10% of Istio's overhead). No sidecar injection means no application restart for mesh enablement. CNCF graduated project. Supports mTLS via wireguard or IPsec integration.

- **Istio 1.24:** Mature, widely-adopted service mesh with comprehensive features — mTLS, traffic management (canary, blue-green, circuit breaking), observability (grafana dashboards, tracing), and authorization policies. However: Istio requires Envoy sidecar proxies injected into every pod. Each sidecar consumes ~50-100 MB RAM and adds ~5-10 ms latency per request. For a deployment with 200+ services, this translates to 10-20 GB of RAM consumed by sidecars alone — unacceptable for Plane 1 (Raspberry Pi) and unnecessarily wasteful for Plane 2+. Istio control plane components (istiod, ingress/egress gateways) add additional resource overhead. Sidecar injection complexity adds operational burden for the single founder.

- **Linkerd 2.x:** Lightweight service mesh with a Rust-based proxy (linkerd-proxy). Significantly smaller resource footprint than Istio (~10 MB per sidecar, ~1-2 ms added latency). Simpler to install and operate than Istio. However: fewer features than Cilium or Istio — no HTTP-aware network policies, limited traffic management, no multi-cluster mesh. Observability integration requires separate Prometheus and Grafana configuration. Not a CNI replacement — still needs Flannel, Calico, or Cilium for networking.

- **Raw Kubernetes NetworkPolicy:** Native Kubernetes network policies without a service mesh. Simplest approach — no additional components, no sidecars, no control plane. However: Kubernetes NetworkPolicy is limited to L3/L4 (IP addresses and ports). No L7 policies (HTTP methods, paths, headers), no mTLS, no observability, no traffic splitting, no circuit breaking. Unsuitable for a zero-trust architecture — network policies alone cannot authenticate service identity.

- **Traditional CNI (Calico) + separate service mesh (Istio):** The conventional enterprise approach — Calico for networking and network policies, Istio for mTLS and traffic management. However: combines the complexity and resource overhead of both systems. Calico's eBPF dataplane (if enabled) overlaps with Cilium's capabilities. The dual-MPLS approach adds operational burden and resource consumption without clear benefit over Cilium's unified approach.

---

## 4. Reason for Decision

1. **Resource efficiency — critical for Plane 1:** Cilium's eBPF approach operates at the kernel level without sidecar proxies. Each pod saves 50-100 MB RAM compared to Istio. On a Raspberry Pi with 8 GB RAM shared across all services, this difference is decisive. The ~60% overhead reduction compared to Istio (as measured by CNCF benchmarks) is essential for resource-constrained deployment contexts.

2. **Unified CNI + service mesh:** Cilium replaces both the CNI (Flannel/Calico role) and the service mesh (Istio/Linkerd role) with a single component. This simplifies the infrastructure stack — one component to install, configure, and maintain instead of two or three. For a single-founder project, fewer components means less operational surface area.

3. **No sidecar — no application restart:** Cilium enables the mesh without modifying pods — no sidecar injection, no init containers, no application restart. The mesh can be enabled gradually across a cluster without redeploying applications. This is critical for existing deployments and for Plane 1 where every megabyte of RAM counts.

4. **Hubble observability:** Hubble provides rich flow-level observability from the same eBPF data path as Cilium's networking. Service dependency graphs, latency distributions, TCP flow metrics, and DNS query visibility are available without additional instrumentation. This satisfies the observability needs for a single founder operating the platform without dedicated SRE tooling.

5. **eBPF for performance:** eBPF programs run in the kernel with near-native performance. Cilium's kube-proxy replacement performs service load balancing more efficiently than iptables-based kube-proxy. This is especially beneficial for FHIR API services where network throughput directly impacts clinical data access latency.

6. **Multi-cluster cluster mesh:** Cilium's ClusterMesh enables cross-cluster service communication without a separate multi-cluster service mesh solution. Services in Plane 3 (national cloud, multiple clusters) can discover and communicate with services in other clusters transparently.

---

## 5. Consequences

**Positive:**

- ~60% less resource overhead than Istio — critical for Plane 1 (Raspberry Pi) deployments
- Single component replaces CNI + service mesh — simpler infrastructure stack
- No sidecar injection — mesh enables without application restart
- Hubble provides rich flow-level observability without separate instrumentation
- eBPF-native networking provides near-kernel performance for FHIR API traffic
- mTLS via wireguard/IPsec for zero-trust service-to-service encryption
- ClusterMesh supports multi-cluster deployments for Plane 3 and Plane 4
- CNCF graduated — vendor-independent, community-governed
- NetworkPolicy compatible with Kubernetes-native policies plus Cilium-specific L7 extensions

**Negative:**

- eBPF requires Linux kernel 5.10+ — older kernels are incompatible (Raspberry Pi OS with kernel 6.1+ is fine)
- Cilium's feature set is broader than strictly necessary — many features (e.g., service mesh L7 policies, cluster mesh) may not be used initially
- Debugging eBPF programs requires specialized knowledge — uncommon skill set
- CiliumNetworkPolicy extends Kubernetes NetworkPolicy with its own CRD — teams must learn Cilium-specific policy syntax
- Hubble UI is less mature than Istio's Kiali dashboard for visualizing service meshes
- The Cilium service mesh implementation (L7 policies, mTLS) is newer than Istio's — some enterprise features may be less mature

---

## 6. Status

Accepted. Cilium 1.20.0 is the Kubernetes networking, security policy, and observability layer for all ZarishSphere Platform clusters (Plane 2+). Cilium replaces both the CNI layer and the service mesh layer — no separate Flannel, Calico, or Istio installation is required. Plane 0 and Plane 1 deployments that do not use Kubernetes may use standard Linux networking with application-level security.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **008-zs-adrs/004-adr-no-hapi-fhir.md** — ADR-004: no JVM dependency (JVM-based mesh alternatives ruled out)
→ **008-zs-adrs/016-adr-opentofu-infrastructure-as-code.md** — ADR-016: OpenTofu infrastructure provisioning (cluster networking defined as code)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
