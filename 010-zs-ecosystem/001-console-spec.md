# 001-console-spec.md
## ZarishSphere Console specification
### Browser-based management center

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Capabilities](#2-capabilities)
3. [User interface principles](#3-user-interface-principles)
4. [Architecture](#4-architecture)
5. [Plane 0 operation](#5-plane-0-operation)
6. [Cross-references](#6-cross-references)

---

## 1. Purpose

The Console is the primary user interface for the entire ZarishSphere ecosystem. It provides browser-based access to every ecosystem function — browsing standards, managing modules, deploying apps, configuring users, monitoring health — without requiring terminal access, code editing, or programming knowledge.

## 2. Capabilities

| Capability | Description |
|---|---|
| Dashboard | System health, usage stats, recent activity |
| Index browser | Browse and search ZARISH-INDEX across all domains |
| Marketplace | Discover, install, and manage ecosystem components |
| Module manager | Deploy, configure, update, and remove domain modules |
| App launcher | Launch and manage pre-built applications |
| Form manager | Browse, customize, and deploy form templates |
| Builder | Launch the no-code Builder tool |
| User management | Manage users, roles, permissions |
| Deployment manager | Configure and monitor deployments across planes |
| Data export | Export data in open formats |
| Audit log viewer | View and search audit logs |
| Settings | System configuration, integrations, notifications |

## 3. User interface principles

- **No-code first** — every function via GUI. CLI is always secondary (Law 6).
- **Progressive disclosure** — simple views by default, advanced options on request.
- **Offline-capable** — Console works as PWA on Plane 0 and Plane 1.
- **Mobile-responsive** — full functionality on tablets and phones.
- **Multi-language** — English, Bengali, Rohingya languages from day one.
- **Accessible** — WCAG 2.2 AA compliance.

## 4. Architecture

| Component | Technology |
|---|---|
| Frontend framework | React 19 + TypeScript |
| Build tool | Vite 8 |
| PWA | Workbox 7 |
| State management | Zustand |
| API communication | REST + GraphQL |
| Authentication | Keycloak OIDC (GitHub OAuth optional) |

## 5. Plane 0 operation

The Console runs as a fully offline PWA on Plane 0. All management functions work without connectivity. Changes are queued locally and synced when connectivity is available.

## 6. Cross-references

→ **010-zs-ecosystem/002-marketplace-spec.md** — Marketplace specification: the component discovery and deployment hub accessed from the Console
→ **010-zs-ecosystem/003-builder-spec.md** — Builder specification: the no-code creation tool launched from the Console
→ **010-zs-ecosystem/005-forms-spec.md** — Forms specification: the form engine rendered inside the Console
→ **010-zs-ecosystem/007-cli-spec.md** — CLI specification: the secondary interface, always behind the Console
→ **010-zs-ecosystem/008-api-spec.md** — API specification: the REST and GraphQL endpoints the Console consumes
→ **003-zs-platform/003-deployment-planes.md** — Deployment plane model: Console offline operation on Plane 0 and Plane 1
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 6: every capability operable through the Console without a terminal

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
