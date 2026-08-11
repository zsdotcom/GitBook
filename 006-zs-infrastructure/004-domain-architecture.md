# 004-domain-architecture.md
## Domain hierarchy and internal routing subsystem
### All domains, subdomains, DNS records, and routing rules

**Document type:** Architecture
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Draft

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Domain tree](#2-domain-tree)
3. [Subdomain mapping](#3-subdomain-mapping)
4. [Internal routing between subdomains](#4-internal-routing-between-subdomains)
5. [DNS record types and TTL strategy](#5-dns-record-types-and-ttl-strategy)
6. [Security records (CAA, SPF, DKIM, DMARC)](#6-security-records-caa-spf-dkim-dmarc)
7. [Plane 0 domain behaviour](#7-plane-0-domain-behaviour)
8. [Cross-references](#8-cross-references)

---

## 1. Purpose

This document defines the complete domain hierarchy for the ZarishSphere ecosystem: every subdomain, its purpose, its DNS record configuration, and how requests route between subdomains. Domain architecture is the bridge between the Cloudflare edge layer and the service layer — it determines how users and services find each other.

All domain configuration is managed through Cloudflare DNS. No external DNS provider is used.

---

## 2. Domain tree

```
zarishsphere.com (root)
├── Public subdomains
│   ├── www.zarishsphere.com           ← Redirect to root
│   ├── console.zarishsphere.com       ← Console management centre
│   ├── marketplace.zarishsphere.com   ← Component discovery hub
│   ├── builder.zarishsphere.com       ← No-code builder tool
│   ├── docs.zarishsphere.com          ← Published documentation
│   ├── index.zarishsphere.com         ← ZARISH-INDEX public interface
│   ├── standards.zarishsphere.com     ← ZARISH-STANDARDS public interface
│   ├── status.zarishsphere.com        ← Ecosystem status page
│   └── home.zarishsphere.com          ← Ecosystem landing page (alias)
│
├── API and service subdomains
│   ├── api.zarishsphere.com           ← API gateway
│   ├── fhir.zarishsphere.com          ← FHIR R5 endpoint
│   └── identity.zarishsphere.com      ← Identity and authentication
│
├── Infrastructure subdomains
│   ├── cdn.zarishsphere.com           ← CDN asset delivery
│   └── mail.zarishsphere.com          ← Email configuration
│
└── Reserved subdomains
    ├── app.zarishsphere.com           ← Reserved for future hosted apps
    └── dev.zarishsphere.com           ← Reserved for developer portal
```

---

## 3. Subdomain mapping

### 3.1 Public front-end subdomains

| Subdomain | Service | Target platform | Purpose | Authentication |
|---|---|---|---|---|
| `zarishsphere.com` | Cloudflare Pages | Static site | Ecosystem landfall page | None |
| `www.zarishsphere.com` | Redirect | → `zarishsphere.com` | WWW redirect | None |
| `console.zarishsphere.com` | Cloudflare Pages + Workers | Static + API | Management centre | Future: Cloudflare Access |
| `marketplace.zarishsphere.com` | Cloudflare Pages | Static site | Component discovery | None |
| `builder.zarishsphere.com` | Cloudflare Pages + Workers | Static + API | No-code creation tool | Future: Cloudflare Access |
| `docs.zarishsphere.com` | Cloudflare Pages | Static site | Published `zs-docs` | None |
| `index.zarishsphere.com` | Cloudflare Pages | Static site | ZARISH-INDEX public browse | None |
| `standards.zarishsphere.com` | Cloudflare Pages | Static site | ZARISH-STANDARDS public browse | None |
| `status.zarishsphere.com` | Cloudflare Pages | Static site | Service status dashboard | None |
| `home.zarishsphere.com` | Cloudflare Pages | Static site | CNAME alias to root | None |

### 3.2 API and service subdomains

| Subdomain | Service | Target platform | Purpose | Authentication |
|---|---|---|---|---|
| `api.zarishsphere.com` | Cloudflare Workers | Serverless | API gateway — routes to backend services | API key / JWT |
| `fhir.zarishsphere.com` | Cloudflare Workers | Serverless + Go | FHIR R5 endpoint proxy | API key / JWT |
| `identity.zarishsphere.com` | Cloudflare Workers | Serverless | Token issuance and validation | None (public keys) |

### 3.3 Infrastructure subdomains

| Subdomain | Service | Target platform | Purpose |
|---|---|---|---|
| `cdn.zarishsphere.com` | Cloudflare CDN + R2 | Edge cache | Static asset delivery (images, fonts, CSS, JS) |
| `mail.zarishsphere.com` | DNS only | — | Email routing configuration (MX, TXT records) |

---

## 4. Internal routing between subdomains

### 4.1 Cross-subdomain communication

Subdomains are independent and do not communicate directly. All cross-subdomain routing follows these rules:

- **Front-end to API:** Console, Builder, and other front-end apps call `api.zarishsphere.com/*` for REST API requests. API calls originate from the browser, not from server-side code.
- **API to services:** `api.zarishsphere.com` workers route to backend services based on path prefixes:
  - `api.zarishsphere.com/v1/identity/*` → identity service
  - `api.zarishsphere.com/v1/data/*` → data service
  - `api.zarishsphere.com/v1/sync/*` → sync service
- **FHIR endpoint:** `fhir.zarishsphere.com` is a dedicated endpoint that proxies FHIR R5 requests to the Go FHIR server. It is separate from the main API gateway for performance isolation.
- **CDN:** All subdomains reference static assets via `cdn.zarishsphere.com/*` URLs for unified caching.

### 4.2 Request flow example

```
User browser
  └─ https://console.zarishsphere.com/
       ├─ HTML/CSS/JS → Cloudflare Pages (console.zarishsphere.com)
       ├─ Static assets → cdn.zarishsphere.com (Cloudflare CDN + R2)
       └─ API calls → api.zarishsphere.com/v1/* (Cloudflare Worker)
              └─ Routing → Go backend service
```

---

## 5. DNS record types and TTL strategy

### 5.1 Record types used

| Record type | Purpose | Used for |
|---|---|---|
| **A** | IPv4 address mapping | Root domain (placeholder) |
| **AAAA** | IPv6 address mapping | Root domain (placeholder) |
| **CNAME** | Canonical name alias | All subdomain Pages targets |
| **TXT** | Text records | SPF, DKIM, DMARC, domain verification |
| **MX** | Mail exchange | Email routing configuration |
| **CAA** | Certificate authority authorization | SSL certificate policy (future) |
| **SRV** | Service location | Reserved for future use |

### 5.2 TTL strategy

| Record category | TTL | Rationale |
|---|---|---|
| CNAME (Pages targets) | Auto | Rarely changes; Cloudflare manages |
| TXT (SPF/DKIM/DMARC) | Auto | Security records must be stable |
| MX | Auto | Email routing must be reliable |
| A/AAAA | Auto | Standard Cloudflare proxy |
| Records under active change | 120 seconds | During domain migration only |

---

## 6. Security records (CAA, SPF, DKIM, DMARC)

### 6.1 CAA records (Certificate Authority Authorization)

CAA records restrict which certificate authorities can issue SSL certificates for the domain. These are configured but not mandatory while using Cloudflare Universal SSL.

| Name | Tag | Value |
|---|---|---|
| `zarishsphere.com` | `issue` | `digicert.com` |
| `zarishsphere.com` | `issue` | `letsencrypt.org` |
| `zarishsphere.com` | `issue` | `cloudflare.com` |
| `zarishsphere.com` | `iodef` | `mailto:security@zarishsphere.com` |

### 6.2 SPF (Sender Policy Framework)

| Record | Value |
|---|---|
| Type | TXT |
| Name | `@` |
| Value | `v=spf1 include:_spf.mx.cloudflare.net ~all` |

The `~all` (soft fail) policy permits the domain to send email while marking unauthorised senders as suspicious. This is the recommended setting for organisations without a dedicated mail server.

### 6.3 DKIM (DomainKeys Identified Mail)

DKIM is configured and managed through Cloudflare Email Routing. The DKIM public key is published as a TXT record:

| Record | Value |
|---|---|
| Type | TXT |
| Name | `*._domainkey.zarishsphere.com` |
| Value | Auto-generated by Cloudflare Email Routing |

### 6.4 DMARC (Domain-based Message Authentication, Reporting, and Conformance)

| Record | Value |
|---|---|
| Type | TXT |
| Name | `_dmarc.zarishsphere.com` |
| Value | `v=DMARC1; p=quarantine; rua=mailto:security@zarishsphere.com; pct=100` |

| DMARC tag | Value | Meaning |
|---|---|---|
| `p` | `quarantine` | Mail that fails SPF/DKIM is sent to spam |
| `rua` | `mailto:security@zarishsphere.com` | Aggregate reports to security |
| `pct` | `100` | Policy applies to 100% of messages |

---

## 7. Plane 0 domain behaviour

On Plane 0 (air-gapped), there is no public DNS. All subdomain resolution is handled locally:

### 7.1 Local domain resolution

| Plane 0 mechanism | Equivalent public DNS |
|---|---|
| `/etc/hosts` entries | A/AAAA records |
| Local self-signed TLS | Cloudflare Universal SSL |
| No CDN | Local file serving (Caddy) |
| No public subdomains | Single localhost port or `zarishsphere.local` |

### 7.2 Subdomain mapping at Plane 0

| Public subdomain | Plane 0 equivalent |
|---|---|
| `zarishsphere.com` | `http://zarishsphere.local` or `http://localhost:8080` |
| `console.zarishsphere.com` | `http://zarishsphere.local/console` |
| `api.zarishsphere.com` | `http://zarishsphere.local:9090/api` |
| `fhir.zarishsphere.com` | `http://zarishsphere.local:9090/fhir` |
| `identity.zarishsphere.com` | `http://zarishsphere.local:9090/auth` |
| `cdn.zarishsphere.com` | Local filesystem static directory |
| All subdomains | Path-routed under a single local server |

### 7.3 Configuration for local DNS

For Plane 1+ deployments with local network access, a lightweight DNS server (CoreDNS or dnsmasq) can be configured to resolve `*.zarishsphere.com` to the local server IP:

```bash
# /etc/hosts entries for Plane 1 local resolution
192.168.1.100  zarishsphere.com
192.168.1.100  console.zarishsphere.com
192.168.1.100  api.zarishsphere.com
192.168.1.100  fhir.zarishsphere.com
```

---

## 8. Cross-references

→ **006-zs-infrastructure/003-cloudflare-architecture.md** — Cloudflare DNS records, nameservers, and edge configuration
→ **006-zs-infrastructure/005-email-architecture.md** — SPF/DKIM/DMARC record implementation and email routing
→ **003-zs-platform/003-deployment-planes.md** — Plane definitions and Plane 0 deployment behaviour
→ **003-zs-platform/006-api-design.md** — API routing rules used across service subdomains

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
