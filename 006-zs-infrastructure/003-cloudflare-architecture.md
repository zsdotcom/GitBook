# 003-cloudflare-architecture.md
## Cloudflare edge routing, WAF and nameserver architecture
### All Cloudflare services, DNS zones, security policies, and edge configuration

**Document type:** Architecture
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Draft

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Account configuration](#2-account-configuration)
3. [Services inventory](#3-services-inventory)
4. [Domain and subdomain configuration](#4-domain-and-subdomain-configuration)
5. [DNS zone architecture](#5-dns-zone-architecture)
6. [WAF rules and security policies](#6-waf-rules-and-security-policies)
7. [Cloudflare Pages](#7-cloudflare-pages)
8. [Cloudflare Workers](#8-cloudflare-workers)
9. [R2 storage](#9-r2-storage)
10. [Email routing](#10-email-routing)
11. [SSL/TLS configuration](#11-ssltls-configuration)
12. [DDoS protection](#12-ddos-protection)
13. [Zero Trust and Access policies](#13-zero-trust-and-access-policies)
14. [Cross-references](#14-cross-references)

---

## 1. Purpose

This document defines the complete Cloudflare configuration for the ZarishSphere ecosystem. Cloudflare is the sole edge provider per ADR-002, chosen for its free tier, global CDN, integrated WAF, serverless Workers platform, and email routing capability.

Every configuration value, rule, and policy described here must match the live Cloudflare Dashboard. No changes to Cloudflare configuration are applied without first updating this document and the corresponding infrastructure-as-code repository (`zs-infra`).

---

## 2. Account configuration

| Property | Value |
|---|---|
| Account email | `hello@zarishsphere.com` |
| Plan | Free |
| Account ID | Set as `CLOUDFLARE_ACCOUNT_ID` GitHub secret |
| API token | Set as `CLOUDFLARE_API_TOKEN` GitHub secret |
| Two-factor authentication | Enabled |
| Additional domains | 0 (single domain: zarishsphere.com) |
| Zones | 1 |

---

## 3. Services inventory

All Cloudflare services operate on the free plan. The table below lists every service used and its function:

| Service | Free tier limit | ZarishSphere use |
|---|---|---|
| **DNS** | Unlimited records, 2 zones | Authoritative DNS for zarishsphere.com |
| **CDN** | Unlimited bandwidth | Global content delivery for all subdomains |
| **WAF** | Core rules free, rate limiting free | OWASP top 10, rate limiting, bot mitigation |
| **SSL/TLS** | Unlimited Universal SSL certs | Auto-provisioned TLS for all subdomains |
| **Pages** | 500 builds/month, 1 build concurrency | Static site hosting for console, docs, marketplace, home |
| **Workers** | 100,000 requests/day, 10ms CPU time | API routing, request orchestration, edge logic |
| **Email Routing** | Unlimited inbound-only aliases, 2 routing rules | Catch-all and alias-based email routing |
| **R2** | 10 GB storage, 1M writes/month | Standards index artifacts, static asset storage |
| **DDoS** | Always-on L3/4/7 mitigation | Inbound DDoS protection |
| **Analytics** | Web analytics (free) | Privacy-first site analytics |
| **Caching** | Automatic static asset caching | Cache HTML, CSS, JS, images at the edge |

---

## 4. Domain and subdomain configuration

### 4.1 Primary domain

| Property | Value |
|---|---|
| Domain | `zarishsphere.com` |
| Status | Active |
| Nameservers | Cloudflare-assigned (set at registrar) |
| Registrar | Separate from Cloudflare |
| Proxy status | Proxied (orange cloud) for all subdomains |

### 4.2 Subdomain assignment

| Subdomain | Service | Target | Proxy status |
|---|---|---|---|
| `zarishsphere.com` (root) | Cloudflare Pages | Landfall page | Proxied |
| `home.zarishsphere.com` | Cloudflare Pages | Home landing page (CNAME alias to root) | Proxied |
| `console.zarishsphere.com` | Cloudflare Pages | Console interface | Proxied |
| `marketplace.zarishsphere.com` | Cloudflare Pages | Marketplace UI | Proxied |
| `builder.zarishsphere.com` | Cloudflare Pages + Workers | Builder app | Proxied |
| `docs.zarishsphere.com` | Cloudflare Pages | Published docs | Proxied |
| `index.zarishsphere.com` | Cloudflare Pages | ZARISH-INDEX public interface | Proxied |
| `standards.zarishsphere.com` | Cloudflare Pages | ZARISH-STANDARDS public interface | Proxied |
| `api.zarishsphere.com` | Cloudflare Workers | API gateway | Proxied |
| `fhir.zarishsphere.com` | Cloudflare Workers | FHIR R5 endpoint | Proxied |
| `identity.zarishsphere.com` | Cloudflare Workers | Identity service | Proxied |
| `status.zarishsphere.com` | Cloudflare Pages | Status page | Proxied |
| `cdn.zarishsphere.com` | Cloudflare CDN + R2 | Asset delivery | Proxied |
| `mail.zarishsphere.com` | DNS only | Email routing configuration | DNS only |

`home.zarishsphere.com` is defined in → **006-zs-infrastructure/004-domain-architecture.md** §3.1 as a CNAME alias to the root domain.

---

## 5. DNS zone architecture

### 5.1 Nameserver configuration

Cloudflare acts as the authoritative DNS provider for `zarishsphere.com`. The domain registrar uses Cloudflare-assigned nameservers. The current nameservers are obtained from the Cloudflare Dashboard at initial zone setup.

> **Constraint:** Nameservers must never be manually set outside Cloudflare. All DNS management occurs through the Cloudflare Dashboard or API.

### 5.2 DNS record inventory

| Type | Name | Value | TTL | Proxy |
|---|---|---|---|---|
| A | `@` | `192.0.2.1` (placeholder documentation IP — Pages uses CNAME) | Auto | Proxied |
| CNAME | `www` | `zarishsphere.com` | Auto | Proxied |
| CNAME | `home` | `zarishsphere.com` | Auto | Proxied |
| CNAME | `console` | `console.zarishsphere.com.pages.dev` | Auto | Proxied |
| CNAME | `marketplace` | `marketplace.zarishsphere.com.pages.dev` | Auto | Proxied |
| CNAME | `builder` | `builder.zarishsphere.com.pages.dev` | Auto | Proxied |
| CNAME | `docs` | `docs.zarishsphere.com.pages.dev` | Auto | Proxied |
| CNAME | `index` | `index.zarishsphere.com.pages.dev` | Auto | Proxied |
| CNAME | `standards` | `standards.zarishsphere.com.pages.dev` | Auto | Proxied |
| CNAME | `status` | `status.zarishsphere.com.pages.dev` | Auto | Proxied |
| TXT | `@` | `v=spf1 include:_spf.mx.cloudflare.net ~all` | Auto | DNS only |
| TXT | `_dmarc` | `v=DMARC1; p=quarantine; rua=mailto:security@zarishsphere.com` | Auto | DNS only |
| TXT | `zarishsphere.com` | `google-site-verification=...` (if needed) | Auto | DNS only |
| MX | `@` | `mx1.migadu.com` (if not using Cloudflare routing) | Auto | DNS only |

> **Note on email delivery:** Cloudflare Email Routing (inbound-only) is the operational path for all `@zarishsphere.com` mail (see §10 and → **006-zs-infrastructure/005-email-architecture.md**). The conditional `mx1.migadu.com` MX record is a retired/optional record retained for migration scenarios only; it is not active while Cloudflare Email Routing is enabled.

### 5.3 TTL strategy

| Scenario | TTL |
|---|---|
| Standard DNS records | Auto (default Cloudflare) |
| Records expected to change during migration | 120 seconds |
| Static, security-critical records (TXT, MX) | Auto |

---

## 6. WAF rules and security policies

### 6.1 Managed rulesets (free tier)

| Ruleset | Status |
|---|---|
| Cloudflare OWASP Core Ruleset | Enabled (score-based threshold) |
| Cloudflare Managed Ruleset | Enabled |
| DDoS L3/4 | Always-on |
| DDoS L7 | Enabled |

### 6.2 Custom WAF rules

| Rule name | Description | Action | Priority |
|---|---|---|---|
| Block known bots | Block requests from known malicious bot IPs | Block | 1 |
| Rate limit API | Limit API requests to 1000 requests per 10 minutes from a single IP | Block | 2 |
| Rate limit login | Limit login endpoint to 10 requests per minute | Block | 3 |
| Block countries (admin) | Block non-whitelist country access to admin routes (future) | Block | 4 |
| Allow health check | Allow Cloudflare health check traffic | Allow | 5 |

### 6.3 Security level

| Setting | Value |
|---|---|
| Security level | Medium |
| Challenge passage | 30 minutes |
| Browser integrity check | Enabled |
| Bot management | Bot Fight Mode (Cloudflare bot management), enabled on free tier |

### 6.4 IP access rules

| IP / Range | Action | Reason |
|---|---|---|
| `0.0.0.0/0` (catch-all) | Allow | Default — all traffic allowed through WAF |

---

## 7. Cloudflare Pages

### 7.1 Pages projects

| Project | Subdomain | Build command | Build output | Repo connected |
|---|---|---|---|---|
| `zarishsphere-home` | `zarishsphere.com` | `npm run build` | `/` | `zs-home` |
| `zs-console` | `console.zarishsphere.com` | `npm run build` | `/` | `zs-console` |
| `zs-marketplace` | `marketplace.zarishsphere.com` | `npm run build` | `/` | `zs-marketplace` |
| `zs-builder` | `builder.zarishsphere.com` | `npm run build` | `/` | `zs-builder` |
| `zs-docs` | `docs.zarishsphere.com` | `python3 scripts/010-refresh-files.py && mkdocs build` | `/` | `zs-docs` |
| `zs-index` | `index.zarishsphere.com` | `npm run build` | `/` | `zs-zarish-index` |
| `zs-standards` | `standards.zarishsphere.com` | `npm run build` | `/` | `zs-zarish-standards` |
| `zs-status` | `status.zarishsphere.com` | `npm run build` | `/` | `zs-status` |

> **Note:** The build output directory is normalized to `/` (the GitHub Pages default source) for all projects, including `zs-docs`.

### 7.2 Pages build settings (all projects)

| Setting | Value |
|---|---|
| Build system | Default (Cloudflare-managed) |
| Node.js version | 22 (pinned) |
| Environment variables | Per project (deployed via GitHub Actions) |
| Preview deployments | Enabled for PR branches |
| Deployment retention | Unlimited |
| Custom domain | Set per project (see subdomain table) |

### 7.3 Preview deployments

Each Pages project generates a preview URL for every PR branch. Preview URLs follow the pattern:

```
https://{project-name}.pages.dev
```

Preview deployments are used for:
- Visual review of UI changes before merge
- Cross-browser testing
- Integration testing with Workers preview

---

## 8. Cloudflare Workers

### 8.1 Worker routes

| Route | Worker name | Purpose |
|---|---|---|
| `api.zarishsphere.com/*` | `zs-api-gateway` | API request routing and authentication |
| `fhir.zarishsphere.com/*` | `zs-fhir-proxy` | FHIR R5 request proxying to Go server |
| `identity.zarishsphere.com/*` | `zs-identity` | Authentication and token validation |
| `console.zarishsphere.com/api/*` | `zs-console-api` | Console backend API |
| `builder.zarishsphere.com/api/*` | `zs-builder-api` | Builder backend API |

### 8.2 Worker configuration (shared)

| Setting | Value |
|---|---|
| Runtime | Cloudflare Workers (V8 / JavaScript) |
| Memory | 128 MB (free tier) |
| CPU time | 10ms per request (free tier) |
| Request limit | 100,000 requests/day (free tier) |
| Environment variables | Per worker (deployed via GitHub Actions + Wrangler) |
| Secrets | Per worker (set via `wrangler secret put`) |
| Triggers | HTTP routes (see routes table above) |
| Logs | Cloudflare Dashboard → Workers → Logs |

### 8.3 Worker deployment

Workers are deployed via GitHub Actions using `wrangler` CLI:

```yaml
# Simplified worker deployment step
- name: Deploy worker
  run: wrangler deploy
  env:
    CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
```

### 8.4 Worker KV (future)

| Namespace | Purpose | Status |
|---|---|---|
| `ZS_AUTH_SESSIONS` | Session token cache (identity) | Future |
| `ZS_CACHE` | API response cache (reduces origin load) | Future |

---

## 9. R2 storage

### 9.1 R2 buckets

| Bucket name | Purpose | Public access | Estimated size |
|---|---|---|---|
| `zs-standards-index` | ZARISH-INDEX harvested data and metadata files | No | < 1 GB |
| `zs-standards-artifacts` | ZARISH-STANDARDS transformation outputs | No | < 2 GB |
| `zs-static-assets` | Shared static assets (images, fonts, icons) | Yes (via CDN) | < 500 MB |
| `zs-backup` | Configuration and database backups (Plane 3/4) | No | < 5 GB |

### 9.2 R2 access control

| Bucket | Access pattern | Auth method |
|---|---|---|
| `zs-standards-index` | Worker-only access | R2 API token |
| `zs-standards-artifacts` | Worker-only access | R2 API token |
| `zs-static-assets` | Public read via CDN | Public bucket |
| `zs-backup` | Admin-only access | R2 API token |

### 9.3 R2 limits (free tier)

| Limit | Value |
|---|---|
| Storage | 10 GB |
| Class A operations (writes) | 1 million/month |
| Class B operations (reads) | 10 million/month |

---

## 10. Email routing

### 10.1 Email routing configuration

Cloudflare Email Routing handles all inbound email for `@zarishsphere.com` addresses. No mail server is operated — all emails are forwarded to the Foundation's mailbox.

| Setting | Value |
|---|---|
| Email Routing | Enabled |
| Catch-all rule | Enabled — all `*@zarishsphere.com` to `zarishsphere@gmail.com` |
| Custom addresses | Individual rules per alias (see below) |
| DKIM | Enabled (auto-generated) |

### 10.2 Routing addresses

| Address | Forward to | Purpose |
|---|---|---|
| `hello@zarishsphere.com` | `zarishsphere@gmail.com` | General contact |
| `founder@zarishsphere.com` | `zarishsphere@gmail.com` | Founder direct contact |
| `contribute@zarishsphere.com` | `zarishsphere@gmail.com` | Contribution inquiries |
| `index@zarishsphere.com` | `zarishsphere@gmail.com` | ZARISH-INDEX submissions |
| `standards@zarishsphere.com` | `zarishsphere@gmail.com` | ZARISH-STANDARDS inquiries |
| `security@zarishsphere.com` | `zarishsphere@gmail.com` | Security disclosure |
| `legal@zarishsphere.com` | `zarishsphere@gmail.com` | Licensing and legal inquiries |

---

## 11. SSL/TLS configuration

### 11.1 Universal SSL

| Setting | Value |
|---|---|
| SSL/TLS encryption mode | Full (strict) |
| Universal SSL | Enabled — auto-renewed |
| Certificate type | Cloudflare-issued (edge certificate) |
| Minimum TLS version | 1.2 |
| Opportunistic encryption | Enabled |
| TLS 1.3 | Enabled |
| Automatic HTTPS rewrites | Enabled |
| Certificate transparency monitoring | Enabled |

### 11.2 Origin server certificates

| Setting | Value |
|---|---|
| Origin certificate | Not used (cloudflare-proxied only) |
| Authenticated origin pulls | Not enabled (Planes 3/4 only) |

### 11.3 SSL for subdomains

All subdomains are covered by the Cloudflare Universal SSL certificate. No additional certificate configuration is needed.

---

## 12. DDoS protection

| Setting | Value |
|---|---|
| DDoS L3/4 | Always-on — enterprise-grade |
| DDoS L7 | Enabled — HTTP DDoS attack protection |
| Attack alerting | Enabled — email notification to `security@zarishsphere.com` |
| Under attack mode | Available for manual activation via Cloudflare Dashboard |

---

## 13. Zero Trust and Access policies

Zero Trust / Cloudflare Access is **not enabled** during the pre-launch phase. All pages sites are publicly accessible. Access policies will be configured post-launch for admin routes.

| Policy | Status | Target |
|---|---|---|
| Console admin routes | Future | `console.zarishsphere.com/admin/*` |
| Builder API | Future | `builder.zarishsphere.com/api/*` |
| Internal Workers | Future | Worker routes with sensitive data |

---

## 14. Cross-references

→ **006-zs-infrastructure/004-domain-architecture.md** — DNS zones, subdomains, and security records
→ **006-zs-infrastructure/005-email-architecture.md** — Email routing and DMARC policy
→ **006-zs-infrastructure/006-ci-cd-architecture.md** — Pages/Workers/R2 deployment pipelines
→ **006-zs-infrastructure/002-github-org-architecture.md** — GitHub org secrets and Pages sources
→ **008-zs-adrs/002-adr-cloudflare-as-edge-platform.md** — ADR-002: Cloudflare as the sole edge provider
→ **007-zs-tech-stack/006-oss-tool-catalog.md** — OSS tools used with Cloudflare services

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
