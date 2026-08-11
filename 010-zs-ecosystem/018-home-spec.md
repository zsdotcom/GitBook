# 018-home-spec.md
## ZarishSphere Home specification
### Public-facing landing site

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Domain and hosting](#2-domain-and-hosting)
3. [Site structure](#3-site-structure)
4. [Features](#4-features)
5. [Technology stack](#5-technology-stack)
6. [Relation to Console and Marketplace](#6-relation-to-console-and-marketplace)
7. [Plane accessibility](#7-plane-accessibility)
8. [Cross-references](#8-cross-references)

---

## 1. Purpose

ZarishSphere Home (`zarishsphere.com`) is the public-facing landing page for the entire ZarishSphere ecosystem. It serves as the primary entry point for visitors — potential deployers, contributors, partners, and the general public — providing an overview of the platform, download links, documentation, and a blog for announcements.

Key design principles:

- **Static first** — zero server-side processing, zero database
- **Globally available** — served from Cloudflare edge nodes worldwide
- **No authentication required** — fully public
- **Accessible** — WCAG 2.1 AA compliant
- **Minimal** — fast loading, low bandwidth, works on slow connections

## 2. Domain and hosting

### 2.1 Domain

| Property | Value |
|---|---|
| Primary domain | `zarishsphere.com` |
| www redirect | `www.zarishsphere.com` → `zarishsphere.com` |
| DNS | Cloudflare DNS (free tier) |
| DNSSEC | Enabled |
| TLS | Cloudflare Universal SSL (TLS 1.3) |

### 2.2 Hosting

| Property | Value |
|---|---|
| Hosting platform | Cloudflare Pages (free tier) |
| Build output | Static HTML, CSS, JS |
| CDN | Cloudflare global edge network |
| Deployment | GitHub Actions → Cloudflare Pages |
| Preview deployments | Branch-based preview URLs |

### 2.3 Domain architecture

```
zarishsphere.com
├── /                    → Landing page
├── /docs/               → Documentation portal (redirect to zs-docs GitHub Pages)
├── /blog/               → Blog and announcements
├── /marketplace          → Marketplace discovery (redirect to Console)
├── /console/             → Console login (redirect to tenant's Console)
└── /fhir/               → FHIR endpoint documentation
```

## 3. Site structure

```
zs-home/
├── src/
│   ├── pages/
│   │   ├── index.html           ← Main landing page
│   │   ├── about.html           ← About the Foundation
│   │   ├── download.html        ← Download distributions
│   │   ├── documentation.html   ← Documentation links
│   │   ├── community.html       ← Community and contributing
│   │   └── blog/
│   │       ├── index.html       ← Blog listing
│   │       └── posts/           ← Individual blog posts
│   ├── assets/
│   │   ├── css/
│   │   ├── js/
│   │   ├── images/
│   │   └── fonts/
│   ├── components/
│   │   ├── header.html
│   │   ├── footer.html
│   │   ├── hero.html
│   │   └── features.html
│   └── layouts/
│       └── base.html
├── static/
│   ├── robots.txt
│   ├── sitemap.xml
│   ├── favicon.ico
│   └── .well-known/
│       ├── security.txt
│       └── openpgpkey/
├── config/
│   └── cloudflare-pages.toml
└── scripts/
    ├── build.sh
    └── deploy.sh
```

## 4. Features

### 4.1 Landing page sections

| Section | Content |
|---|---|
| Hero | Tagline, CTA buttons (Download, Learn More) |
| Features | Key ecosystem capabilities with icons |
| Domains | Visual representation of the 40-domain taxonomy |
| Use cases | Real-world deployment scenarios |
| Distributions | Quick links to pre-built distributions |
| Call to action | Get started links |
| Footer | Foundation info, licenses, social links |

### 4.2 Blog

- Markdown-based blog posts committed to the repository
- Posts are built into static HTML at deploy time
- Categories: Announcements, Guides, Case Studies, Technical
- RSS feed at `/blog/feed.xml`
- No comments section (GitHub Discussions for community interaction)

### 4.3 Download center

- Direct download links for distribution packages
- Links to `zs-docs` for full documentation
- Quick-start guides
- SHA-256 checksums for all downloads

### 4.4 SEO

| Feature | Implementation |
|---|---|
| Sitemap | Auto-generated `sitemap.xml` |
| Meta tags | Open Graph, Twitter Cards, JSON-LD |
| Structured data | Organization, SoftwareApplication schema |
| Canonical URLs | All pages have canonical tags |
| Robots.txt | Properly configured |
| Performance | Lighthouse score target: 95+ |

## 5. Technology stack

| Layer | Technology | Version |
|---|---|---|
| Static site generator | 11ty (Eleventy) | 3.x |
| CSS framework | Tailwind CSS | 4.x |
| JavaScript | Vanilla JS | ES2024 |
| Fonts | System font stack (no custom fonts) | — |
| Icons | SVG inline | — |
| Analytics | Cloudflare Web Analytics (privacy-first) | — |
| Forms | Static form action to GitHub Issues | — |
| Build | GitHub Actions | — |
| Hosting | Cloudflare Pages | — |

> **Constraint:** The Home site must never introduce server-side dependencies. No Node.js at runtime, no database, no API server. Everything runs at build time.

## 6. Relation to Console and Marketplace

| Component | Relationship |
|---|---|
| **Console** | Home links to Console for authenticated users. Users click "Go to Console" from the Home site. |
| **Marketplace** | Home provides a "Browse Marketplace" link that redirects to the Console's marketplace view. |
| **Documentation** | Home links to `zs-docs` GitHub Pages for full documentation. |
| **Download** | Home provides direct download links for distributions from GitHub Releases. |

The Home site is the front door. The Console is the operations center. The Marketplace is the component store. They form a funnel: Home → Console → Marketplace.

## 7. Plane accessibility

The Home site is a Plane 4 (global SaaS) component — it requires internet access. There is no offline version of the Home site.

However, the **content** published on the Home site (documentation, download links) is designed to reach users on any plane:

- Documentation is mirrored in the `zs-docs` repo and accessible via offline documentation bundles
- Distribution packages are downloadable and can be transferred via USB to Plane 0 devices
- Blog posts are available as markdown files in the repository for offline reading

## 8. Cross-references

→ **010-zs-ecosystem/001-console-spec.md** — Console specification: the operations center the Home site links to
→ **010-zs-ecosystem/002-marketplace-spec.md** — Marketplace specification: the component store reached through the Home site's "Browse Marketplace" link
→ **010-zs-ecosystem/011-distributions-spec.md** — Distributions specification: the packages offered in the Download center with SHA-256 checksums
→ **003-zs-platform/008-domain-registry.md** — Domain registry: the 40-domain taxonomy displayed in the Domains section
→ **003-zs-platform/003-deployment-planes.md** — Deployment plane model: the plane definitions behind the Plane 4 accessibility classification
→ **006-zs-infrastructure/002-github-org-architecture.md** — GitHub organization architecture: the GitHub Actions build and deployment pipeline to Cloudflare Pages
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 1: GitHub as the government — site content and blog posts live in the repository

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
