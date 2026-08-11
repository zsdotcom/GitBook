# 009-adr-no-vendor-lock-in.md
## ADR-009: Decoupling the Platform Layer from Cloud Infrastructure Provider Specific Dependencies
### Abstract all infrastructure behind interfaces — no platform depends on a single vendor

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

All infrastructure provider dependencies (Cloudflare, GitHub, and any future cloud services) must be abstracted behind well-defined interfaces. The platform core must never import or depend on a vendor-specific SDK directly. Vendor-specific implementations must be swappable via configuration, with at least two alternative implementations (one production, one local/mock) for every abstracted interface.

This ADR directly implements Constitution Law 9 (vendor freedom): "No ZarishSphere component, module, service, or workflow may depend on a proprietary API, a paid service, or a single-vendor infrastructure component as a required dependency."

---

## 2. Context

ZarishSphere uses several infrastructure providers' free tiers during V1 development:

- **Cloudflare:** DNS, CDN, WAF, Workers (serverless), Pages (hosting), R2 (object storage), Email Routing
- **GitHub:** Repositories, Actions (CI/CD), Pages (documentation hosting), API (bot integration)
- **Cloudflare R2:** Object storage (S3-compatible, on free tier)
- **Cloudflare Workers:** Edge API functions and routing

While these providers are used as architectural convenience (Constitution Law 9 commentary explicitly states "Cloudflare is used as an edge layer because it can be replaced"), the platform must operate on any infrastructure:

- Plane 0 (air-gapped): No internet, no cloud dependencies
- Plane 1 (Raspberry Pi): Local network only, no cloud services
- Plane 2 (district server): Intermittent internet, may use local services
- Plane 3 (national cloud): Self-hosted cloud, may prefer AWS or on-premise
- Plane 4 (global SaaS): Could be Cloudflare-based or not

If the platform code has hard imports of `cloudflare-go` SDK, or relies on Cloudflare-specific features that have no open alternative, the platform violates Law 9.

---

## 3. Alternatives Considered

| Alternative | Pros | Cons |
|---|---|---|
| **Abstract all infra behind interfaces** | Full vendor freedom; any plane can run independently; provider can be swapped via config (switching from Cloudflare Workers to Deno Deploy or Node.js runtime); local dev with mock implementations; platform truly forkable (Law 11) | Extra abstraction layer adds code complexity (~10-15% more code); cannot use some advanced vendor features that have no open equivalent; must maintain multiple implementations |
| **Use vendor SDKs directly, document swap-out plan** | Simpler codebase; full access to vendor-specific features; faster initial development | Violates Law 9 structurally (not just policy); if the plan is not implemented, lock-in is real; single founder may never get around to the abstraction; risk if Cloudflare changes free tier or goes out of business |
| **Avoid cloud services entirely (local-only)** | No vendor dependency at all; simplest architecture; Plane 0/1 perfect | Plane 2-4 cannot benefit from cloud efficiency; no global CDN; no DDoS scrubbing; no serverless edge functions; reinvents every cloud primitive; unrealistic for global SaaS (Plane 4) |
| **Use multi-cloud from day one (CF + AWS + GCP)** | No single vendor can hold the project hostage | 3x operational complexity; impossible to maintain by single founder; each free tier has 3x different APIs to learn; no budget for multi-cloud (free tiers are per-provider); violates ADR-006 (zero-cost toolchain — cannot run in parallel for free) |

---

## 4. Reason for Decision

1. **Constitutional requirement:** Law 9 states: "Vendor lock-in is classified as a structural failure equivalent to a security vulnerability. It is not a trade-off. It is not a temporary pragmatic choice. It is a violation of this constitution." Architectural abstraction is not optional — it is a compliance requirement.

2. **Deployment plane diversity:** The five deployment planes have fundamentally different infrastructure requirements. Plane 0 and 1 cannot use any cloud service. Plane 3 may run on AWS, Azure, or on-premise. Plane 4 (SaaS) might use any or multiple providers. A single-vendor-dependent platform core would require rewrites for each plane.

3. **Provider volatility:** Free tiers change. Cloudflare's free tier could reduce limits, or a new leadership could change pricing model. GitHub could be acquired. The platform must not be at the mercy of a vendor's business decisions. The zero-cost guarantee (Law 5) must be structurally enforced, not dependent on a vendor's goodwill.

4. **Succession and forkability (Law 11):** If the ZarishSphere Foundation ceases to exist, the platform must remain fully deployable. If the platform is hard-coded to use cloudflare.com Workers routes and R2 buckets, a fork would need to replace large portions of the codebase to use alternative infrastructure. Interface-based abstraction makes the fork viable with minimal changes (just implement the interface for a new provider).

### 4.1 Abstraction strategy

| Infrastructure Primitive | Interface | Cloudflare Implementation | Local / Mock Implementation | Open Alternative |
|---|---|---|---|---|
| Object storage | `ObjectStore` (Get, Put, Delete, List) | `R2Store` (via `aws-sdk-go` S3-compatible API) | `LocalFileStore` (filesystem-based, for dev/Plane 0) | MinIO RELEASE.2026-05-29T18-24-33Z (S3-compatible self-hosted) |
| Edge functions | `FunctionRuntime` (HTTP handler) | `WorkersHandler` (WASM/JS) | `LocalHTTPHandler` (Go net/http server) | Deno Deploy, Self-hosted Node.js |
| CDN / caching | `CacheProvider` (purge, warm) | `CloudflareCache` (API-based purge) | `NoopCache` (pass-through for dev) | Fastly, Self-hosted nginx + varnish |
| DNS | `DNSProvider` (record management) | `CloudflareDNS` (API-based) | `LocalHostsFile` (for dev/Plane 0) | Self-hosted BIND, AWS Route 53 |
| Email routing | `EmailProvider` (send, receive) | `CloudflareEmailRouting` + Mailgun | `SMTPDirect` (Go net/smtp, for dev) | Self-hosted Postfix |
| CI/CD | `CIRunner` (build, test, deploy) | GitHub Actions (YAML-based) | Local `taskfile` or `Makefile` | GitLab CI, Drone, Woodpecker |

---

## 5. Consequences

**Positive:**

- Full compliance with Constitution Law 9 (vendor freedom)
- Every deployment plane can run without any cloud vendor dependency
- Local development uses mock services — no internet required for `go run`
- Provider switching is configuration change, not code change
- Platform is structurally forkable — a fork can use entirely different infrastructure
- New deployment planes (e.g., "Plane 5: Allied Health Network") can plug in without platform changes

**Negative:**

- ~10-15% additional code volume for abstraction interfaces and multiple implementations
- Cannot use vendor-specific features that have no abstractable counterpart (e.g., Cloudflare Workers' Durable Objects, Queue, KV — must build these into the abstraction or use open alternatives)
- Testing must verify multiple implementations for each interface
- Documentation must cover configuration for each supported provider
- Development iteration slower than direct SDK usage (must implement interface + test before using a feature)

---

## 6. Status

Accepted. This ADR is structurally enforced in the architecture: the `platform/internal/infrastructure/` package contains interfaces only. Cloudflare-specific implementations are in a separate `platform/internal/infrastructure/cloudflare/` package that is compiled only when the `cloudflare` build tag is active. The default build tag (`local`) uses filesystem-based implementations so the platform compiles and runs with zero cloud dependencies.

---

## 7. Cross-references

→ **008-zs-adrs/002-adr-cloudflare-as-edge-platform.md** — ADR-002: Cloudflare as edge platform (abstracted per this ADR)
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: zero-cost toolchain
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 9 (vendor freedom), Law 11 (forkability)
→ **003-zs-platform/003-deployment-planes.md** — Five deployment planes and their infrastructure requirements

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
