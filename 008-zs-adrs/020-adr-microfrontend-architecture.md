# 020-adr-microfrontend-architecture.md
## ADR-020: Microfrontend Architecture with Vite 8.2.0 Module Federation
### Independently Deployable, Independently Versioned, Runtime Composition

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

Use microfrontend architecture for all ZarishSphere Platform frontend applications, composed at runtime via Vite 8.2.0 Module Federation (Module Federation 2.0). Each domain module (health, education, logistics), ecosystem application (Console, Marketplace, Builder), and functional area is an independently deployable, independently versioned React 19.2.8 application. A shell application (the Console shell) composes microfrontends at runtime, loading only the modules relevant to the current user context. The shared component library (Carbon Design System 1.113.0) and shared dependencies (React, Next.js 16.2.12) are federated as shared singletons.

---

## 2. Context

The ZarishSphere Platform frontend spans multiple distinct application surfaces:

- **ZarishSphere Console** — The primary management interface for platform administration, module configuration, user management, deployment monitoring, and system settings.
- **ZarishSphere Marketplace** — The discovery and deployment hub for modules, apps, templates, and distributions.
- **ZarishSphere Builder** — GUI-based no-code tool for creating forms, workflows, and modules.
- **Domain Apps** — Pre-built applications for specific domains including health (clinical workflows, patient management, FHIR resource browser), education (student records, curriculum management), logistics (supply chain tracking, inventory management), and all 40+ domains.
- **Forms Engine** — Dynamic form rendering engine embedded in domain apps.

Key architectural requirements:

- **Independent deployability:** Each domain module must be deployable independently — a fix to the health module must not require redeploying the education module. Constitution Law 7 (Module sovereignty) requires that modules be independently deployable without co-dependency.
- **Independent versioning:** Different domains may be on different versions at the same time. A tenant using health v1.2 and logistics v2.0 must be supported without forcing one to upgrade to match the other.
- **Team autonomy:** While currently a single-founder project, the architecture must support future parallel development by multiple contributors or teams without coordination bottlenecks.
- **Runtime composition:** The specific set of modules available to a user depends on their deployment context — a clinic in Cox's Bazar may only have the health module, while a national Ministry of Health may have health, logistics, and education. Modules must compose at runtime, not build time.
- **Shared design system:** All microfrontends use Carbon Design System (ADR-019) for visual consistency. Shared dependencies (React, Next.js, Carbon) must be singleton instances to avoid bundle bloat.
- **Offline capable:** Plane 0 and Plane 1 deployments may not have access to a remote module registry. The architecture must support pre-loading microfrontends for offline operation.

---

## 3. Alternatives Considered

- **Microfrontend via Vite 8.2.0 Module Federation (chosen):** Each microfrontend is a standalone Vite 8.2.0 + React 19.2.8 application with its own build pipeline, package.json, and deployment lifecycle. Module Federation 2.0 enables runtime composition in the browser — the shell loads the manifest of available modules, fetches the relevant entry points, and composes the UI at runtime. Shared dependencies (React, Carbon, utility libraries) are declared as shared singletons — loaded once and shared across all microfrontends. CSS is scoped via CSS modules or Carbon's design tokens. Independent Git repositories per microfrontend with independent CI/CD pipelines. Supports offline pre-loading via service worker manifest caching.

- **Monolith SPA (single Next.js application):** All domain functionality in a single Next.js application. Simplest architecture, fastest initial development, no microfrontend infrastructure complexity. However: as the platform grows to 40+ domains, a single application becomes unmanageable — build times increase, deployment requires coordinated releases, a single buggy domain module can crash the entire application, and team autonomy is impossible. Conflicts with Constitution Law 7 (Module sovereignty) — domains that should be independent become tightly coupled in a single codebase. Code splitting via lazy loading mitigates bundle size but does not solve independent deployability.

- **iFrames:** Each module rendered in a separate iframe with postMessage communication. Maximum isolation — each iframe is a completely independent HTML document with its own JavaScript context, CSS scope, and memory. However: iframes provide poor user experience (no shared navigation, inconsistent scroll behavior, cross-origin communication complexity), poor performance (each iframe loads a complete page), and state management across iframes is extremely difficult. CSS cannot be shared across iframes for visual consistency. Accessibility issues with screen readers navigating across iframe boundaries.

- **NPM packages with version coordination:** Each module is published as an NPM package. The shell application installs specific versions of each package at build time. Provides code reuse and version control. However: version coordination becomes a maintenance burden — if module A depends on module B v1.x and module C depends on module B v2.x, the application cannot satisfy both. Every upgrade requires a coordinated release across all modules. Does not support independent deployability — the entire application must be rebuilt and redeployed to change any module version.

- **Web Components:** Each module built as a native Web Component (custom elements, shadow DOM). Framework-agnostic, standards-based, maximum portability. However: React Web Components have poor ergonomics — React's event system does not work with native custom events, React 19.2.8's support for custom elements is improved but still requires careful integration. No code sharing mechanism (no shared singleton pattern). Testing Web Components with React testing tools is difficult. Shadow DOM CSS encapsulation conflicts with Carbon Design System's global design token approach.

---

## 4. Reason for Decision

1. **Module sovereignty (Constitution Law 7):** The constitution requires that every module be independently deployable. Microfrontend architecture is the frontend implementation of this principle — each domain's UI is a standalone application that can be deployed, versioned, and scaled independently.

2. **Independent deployability enables parallel development:** A single founder currently builds the entire platform, but the architecture must support future parallel work. With microfrontends, a contributor can build and deploy the education module independently without touching or affecting the health module.

3. **Vite 8.2.0 Module Federation over Next.js Module Federation:** Vite 8.2.0's Module Federation 2.0 implementation is more mature and flexible than Next.js's integration. It supports dynamic loading of remote modules, declarative shared dependency configuration, and works with any framework (not just Next.js). The Console and Marketplace use Next.js 16.2.12 for SSR/SSG, while domain microfrontends may use plain Vite 8.2.0 + React 19.2.8 for smaller bundle sizes.

4. **Runtime composition for multi-domain deployments:** Different organizations deploy different sets of modules. The Console shell must discover available modules at runtime and compose the appropriate UI. Module Federation enables this — the shell loads a manifest, fetches module entry points, and renders them without a build step.

5. **Shared dependency optimization:** Module Federation's shared singleton pattern ensures that React, Carbon Design System, and utility libraries are loaded once and shared across all microfrontends. Without this, loading eight microfrontends would download eight copies of React — ~500 KB each.

---

## 5. Consequences

**Positive:**

- Each domain module is independently deployable, versionable, and scalable
- Teams can work on different modules without coordination bottlenecks
- Runtime composition enables per-deployment custom module sets
- Module Federation shared singletons prevent bundle bloat from duplicated dependencies
- Independent CI/CD pipelines per microfrontend — a fix to one module does not require redeploying others
- Rollback of a single module does not affect other modules
- New domain modules can be added without modifying existing module code
- Offline pre-loading via service worker caching for Plane 0/1 deployments

**Negative:**

- Microfrontend infrastructure adds complexity: shared dependency configuration, shell development, module discovery, cross-module communication
- Runtime performance overhead from Module Federation's remote module loading
- Cross-module state sharing requires careful design — each module is its own React application with its own state tree
- CSS scoping requires discipline — Carbon design tokens help but cross-module style conflicts can occur
- Testing requires testing each microfrontend independently plus integration testing in the shell
- Build times increase if each microfrontend has its own build pipeline (mitigated by independent builds running in parallel)
- Single founder must learn and maintain the microfrontend infrastructure in addition to the application code
- Not all deployment planes benefit from microfrontend complexity — Plane 0/1 may use pre-bundled single-application builds

---

## 6. Status

Accepted. The ZarishSphere Platform frontend uses a microfrontend architecture with Vite 8.2.0 Module Federation 2.0. The Console shell serves as the primary composition host. Each domain module and ecosystem application is an independently deployable microfrontend. Plane 0 and Plane 1 may use pre-bundled single-application builds for simplicity, but the microfrontend architecture must be the canonical implementation.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **008-zs-adrs/001-adr-go-as-primary-language.md** — ADR-001: frontend stack context (React 19.2.8, Next.js 16.2.12)
→ **008-zs-adrs/019-adr-carbon-design-system.md** — ADR-019: Carbon 1.113.0 as the shared design system across microfrontends
→ **010-zs-ecosystem/001-console-spec.md** — Console specification (primary composition host)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
