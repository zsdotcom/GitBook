# 002-adr-cloudflare-as-edge-platform.md
## ADR-002: Selection of Cloudflare as the Edge Platform
### Free-tier Cloudflare for CDN, WAF, DNS, Workers, Pages, and email routing

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

Use Cloudflare's free tier as the edge platform for all ZarishSphere web properties. This includes Cloudflare DNS (authoritative nameserver), CDN (caching and static asset delivery), WAF (web application firewall), Cloudflare Pages (hosting for static sites including zs-docs and zarishsphere.com), Cloudflare Workers (serverless function runtime for API edge logic), and Cloudflare Email Routing (inbound email forwarding).

---

## 2. Context

The ZarishSphere ecosystem requires global web hosting for multiple properties:

- **zarishsphere.com** — public landing site and documentation portal
- **app.zarishsphere.com** — Console (SPA application)
- **api.zarishsphere.com** — API gateway endpoints
- **docs.zarishsphere.com** — published documentation

Each property needs HTTPS termination, DDoS protection, WAF rules, global CDN distribution, and reliable DNS resolution. As a zero-budget project (Constitution Law 5), every infrastructure component must be available on a genuine free tier — not a trial, not a limited-time offer, not a credit-card-required signup. Additionally, per Constitution Law 9 (vendor freedom), the chosen platform must not create irreversible vendor lock-in.

---

## 3. Alternatives Considered

| Alternative | Pros | Cons |
|---|---|---|
| **Cloudflare Free Tier** | Generous free tier: unlimited CDN bandwidth, 100,000 Workers requests/day, 3 Pages builds/month, 1 GB R2 storage, advanced DDoS protection, free SSL, integrated DNS (unlimited records), email routing (up to 100 destinations), WAF with OWASP rules, bot management | Free tier has limits (100K Workers req/day, 10 Workers/account, 3 Pages builds/min); primarily US/EU edge presence limited in some regions; ADR-009 requires abstraction to avoid lock-in |
| **AWS CloudFront + Route 53 + WAF** | Most mature CDN (200+ edge locations), deep integration with other AWS services, strong WAF rules | Free tier expires after 12 months (1 TB transfer, 10M requests); Route 53 charges $0.50/hosted zone/month; WAF charges $5/month + rules; cost for solo founder is prohibitive; complex IAM configuration overhead |
| **Vercel** | Excellent Next.js integration, generous hobby tier (100 GB bandwidth, 6,000 build minutes/month), automatic SSL and CDN | No native WAF; limited DDoS protection; no Workers equivalent (edge functions are Vercel-specific); vendor lock-in to Vercel runtime; no email routing; plans to monetize free tier (credit-card-gated features) |
| **Netlify** | Generous free tier (100 GB bandwidth, 300 build minutes/month), form handling, functions | No WAF beyond basic DDoS; limited edge logic (Netlify Edge Functions are new and limited); no DNS management (Netlify DNS is basic); fewer edge locations than Cloudflare; no email routing |
| **Self-hosted (nginx + Let's Encrypt)** | Zero vendor dependency, full control, no limits | Requires a VPS (cost: $5-10/month for minimal instance); single point of failure without load balancing; no global CDN; no DDoS scrubbing; SSL renewal automation needed; Plane 0/1 are local-only and don't need this |

---

## 4. Reason for Decision

1. **Genuine free tier:** Cloudflare's free tier is permanent, not a trial. No credit card is required to start. The free tier includes features that competitors charge for — advanced DDoS mitigation (unlimited), WAF with OWASP Top 10 rules, and authoritative DNS with unlimited records. This aligns with Constitution Law 5 (zero-cost structural guarantee).

2. **Integrated stack:** Cloudflare provides DNS, CDN, WAF, serverless Workers, static Pages, and email routing under a single dashboard. This eliminates the complexity of managing separate providers for each function — critical for a solo founder with limited DevOps experience (founder profile §2.4: Docker/containerisation at beginner level).

3. **Workers for edge API:** Cloudflare Workers (based on Service Workers API) allows running JavaScript/WebAssembly at the edge. This enables API authentication, rate limiting, request transformation, and routing without running a dedicated backend server. Workers free tier (100,000 requests/day) is sufficient for V1 traffic.

4. **Security baseline:** Free DDoS mitigation (up to 100 Tbps scrubbing capacity) and WAF are essential for a health-interoperability platform that may process sensitive data. Cloudflare's security posture exceeds what a solo founder could configure on a self-hosted solution.

5. **Pages for documentation:** Cloudflare Pages serves static sites from git integration. `zs-docs` can be published via `wrangler pages publish` or git push. No build server required. The 3 builds/minute limit is sufficient for a documentation-only repo.

---

## 5. Consequences

**Positive:**

- All web properties behind DDoS protection and WAF from day one
- Global CDN reduces latency for documentation and Console to <100 ms avg
- Zero ongoing cost for edge infrastructure during V1 development
- Unified dashboard for DNS, CDN, security, and email routing
- Quick SSL certificate management (automatic, wildcard support)

**Negative:**

- Vendor dependency on Cloudflare (mitigated by ADR-009 — abstract infra behind interfaces)
- Free tier limits (100K Workers req/day, 3 Pages builds/min) may constrain future scale
- Workers runtime limited to V8 isolates — cannot run Go binaries directly (Workers must be JS/WASM)
- R2 object storage on free tier limited to 1 GB (may need upgrade for asset storage)
- Cloudflare cache purging is API-only on free tier (no instant purge button)

---

## 6. Status

Accepted. This decision is subject to ADR-009 (no vendor lock-in), which requires all Cloudflare interactions to be abstracted behind interfaces so the platform can run on any infrastructure.

---

## 7. Cross-references

→ **008-zs-adrs/009-adr-no-vendor-lock-in.md** — ADR-009: infrastructure abstraction against vendor lock-in
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: zero-cost toolchain requirement
→ **006-zs-infrastructure/003-cloudflare-architecture.md** — Cloudflare services, DNS, WAF, and edge configuration detail
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 5 (zero cost), Law 9 (vendor freedom)
→ **001-zs-meta/003-zarishsphere-founder-profile.md** — Founder DevOps skill profile (§2.4)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
