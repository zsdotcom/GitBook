# 019-adr-carbon-design-system.md
## ADR-019: IBM Carbon Design System 1.113.0 as UI Component Library
### Healthcare-Optimized, WCAG 2.2 AA Accessible, Apache 2.0 Licensed

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

Use IBM Carbon Design System 1.113.0 (`@carbon/react` 1.113.0) as the foundational UI component library for all ZarishSphere Platform frontend applications — Console, Marketplace, Builder, Forms engine, and all domain apps. Carbon provides healthcare-appropriate design language, WCAG 2.2 AA accessibility compliance, a comprehensive design token system for theming, and is released under the Apache 2.0 license. All ZarishSphere user interfaces use Carbon components, following Carbon's design guidance for layout, typography, color, and interaction patterns.

---

## 2. Context

The ZarishSphere Platform requires a design system that serves:

- **Clinical interfaces:** Patient registration, encounter management, clinical decision support, lab result viewing, medication administration — interfaces where clarity, accuracy, and reliability are paramount. Health data is used by clinicians making time-sensitive decisions; the UI must not introduce ambiguity or cognitive friction.
- **Administrative interfaces:** User management, module configuration, deployment management, reporting dashboards — used by platform operators who may not have design expertise.
- **Public-facing interfaces:** Patient portal, community health worker mobile apps, public health dashboards — used by individuals with varying levels of digital literacy, including users in resource-constrained settings with limited bandwidth and older devices.
- **Cross-domain interfaces:** The design system must serve not only health but also education, logistics, finance, and all 40+ domains indexed by the ZarishIndex. It must be general enough to handle diverse data types while maintaining visual consistency.

Key constraints:

- **WCAG 2.2 AA compliance:** Clinical software must be accessible to users with disabilities. This is not optional — it is a regulatory requirement in many jurisdictions and an ethical requirement in all. Constitution Law 6 (No-code first) extends to accessibility: the platform must be usable by anyone, regardless of ability.
- **Healthcare-appropriate design language:** The visual design must convey seriousness, trustworthiness, and clinical authority. Bright gaming aesthetics, playful animations, and consumer-oriented patterns are inappropriate for clinical contexts.
- **Free and open source:** The design system must be Apache 2.0 or equivalent — no paid licenses for components, no pro-tier feature gating. Constitution Law 5 (zero cost) and ADR-006 require this.
- **React 19.2.8 compatibility:** The frontend ecosystem uses React 19.2.8 and Next.js 16.2.12 (per ADR-001 and the tech stack master). The design system must provide native React components.
- **Theming for multi-tenant and multi-domain:** Different organizations, different domains, and different regions may need distinct visual identities (brand colors, logos, fonts). The design system must support comprehensive theming via design tokens.

---

## 3. Alternatives Considered

- **IBM Carbon Design System 1.113.0:** Enterprise-grade design system built for data-heavy, complex applications — the profile of clinical and administrative interfaces. WCAG 2.2 AA compliant out of the box. Comprehensive React component library with 80+ components covering data tables, forms, navigation, modals, notifications, and complex data visualization. Design token system (CSS custom properties) enables full theme customization. Apache 2.0 licensed with no usage restrictions. IBM uses it for Watson Health, IBM Cloud, and enterprise products — validated at scale. Healthcare-specific patterns available (patient summary, clinical timeline, vital signs display). Large community and extensive documentation.

- **Material UI (MUI) 6.x:** Most popular React component library, huge ecosystem, comprehensive component set, excellent documentation, free community edition. However: Material Design is Google's design language, optimized for consumer applications — not healthcare. The visual language (card-based layouts, floating action buttons, bottom navigation) is inappropriate for clinical interfaces. Accessibility varies by component and requires careful implementation. Themes can be customized but Material Design's visual DNA is hard to override completely. MUI's pro features (data grid, date pickers, charts) require a commercial license — violates ADR-006's zero-cost requirement for the feature set needed.

- **Ant Design 5.x:** Comprehensive enterprise component library with excellent feature coverage, internationalization (i18n) support, and a mature design language. However: Ant Design's visual language is a distinctly enterprise tech-company aesthetic — not appropriate for humanitarian health contexts. Some documentation is Chinese-only. Accessibility (WCAG compliance) is not a primary design goal — many components fail accessibility audits. The component API patterns differ significantly from standard React patterns. The Baidu/Alibaba ecosystem dependency raises concerns about Constitution Law 9 (vendor freedom) and data sovereignty (Law 4).

- **Radix UI / Shadcn UI:** Low-level, unstyled, accessible component primitives that give complete control over visual design. Excellent accessibility (Radix is WCAG compliant). Copy-paste component model (Shadcn UI). However: low-level primitives require significant custom design and styling work to build a complete UI. For a single-founder project, building 80+ components from primitives is not feasible. No built-in design language, theming system, or design tokens — everything must be created from scratch. No healthcare-specific patterns. Suitable as a supplement for custom components but not as the primary design system.

- **Custom in-house design system:** Complete control over visual design, perfect alignment with ZarishSphere brand identity, no third-party dependency. However: building and maintaining a full design system with 80+ accessible components is a multi-person-year effort. A single founder cannot build both a complete design system and the application logic. Accessibility compliance would require dedicated testing. No healthcare-specific patterns built-in. Violates the principle that documentation precedes existence (Constitution Law 2) — the design system must be built before applications, extending the timeline significantly.

---

## 4. Reason for Decision

1. **Healthcare-appropriate design language:** Carbon was designed for IBM's enterprise products in regulated industries including healthcare. Its visual language — neutral color palette, clear typographic hierarchy, generous whitespace, restrained use of emphasis — is appropriate for clinical interfaces where users are making life-critical decisions. The design does not feel consumer-oriented or gaming-oriented — it conveys seriousness and trust.

2. **Accessibility compliance:** Carbon is WCAG 2.2 AA compliant as a design system — not just individual components but the entire system including color contrast, focus indicators, screen reader support, keyboard navigation, and motion preferences. This satisfies regulatory requirements for clinical software and implements Constitution Law 6 (usable by anyone) at the design system level.

3. **Component breadth for complex data:** ZarishSphere interfaces require complex data tables (patient lists, resource inventories), multi-step forms (patient registration, FHIR resource creation), interactive visualizations, and structured navigation. Carbon provides production-ready components for all of these patterns, saving months of development time compared to building from primitives.

4. **Design token theming:** Carbon's design token system provides comprehensive theming via CSS custom properties. Colors, typography, spacing, and component-specific tokens can be overridden per tenant, per domain, or per deployment. This enables multi-tenant white-labeling for national cloud deployments, domain-specific visual differentiation (health looks different from logistics), and regional brand adaptation.

5. **Apache 2.0 license:** Carbon is Apache 2.0 licensed with no pro tier, no paid components, no usage restrictions. Full source is available. This satisfies ADR-006's zero-cost requirement unconditionally and ADR-009's vendor independence.

6. **Active maintenance by IBM:** IBM maintains Carbon as its primary design system for IBM Cloud, Watson, and enterprise products. It receives regular updates, security patches, and new components. IBM's investment in Carbon ensures long-term viability consistent with Constitution Law 11 (The platform outlives its creators).

---

## 5. Consequences

**Positive:**

- Healthcare-appropriate design language for clinical interfaces — trusted by clinicians
- WCAG 2.2 AA compliant out of the box — regulatory compliance for health deployments
- 80+ production-ready React components save months of development time
- Design token system enables multi-tenant theming and domain-specific visual differentiation
- Apache 2.0 licensed — zero cost, full source access, no licensing restrictions
- Actively maintained by IBM — long-term viability
- Comprehensive documentation and community resources
- Consistent visual language across all ZarishSphere applications (Console, Marketplace, Builder, Apps)

**Negative:**

- Carbon's component API is verbose — React components require more props and configuration than lighter alternatives
- Carbon is designed for desktop-first enterprise applications — mobile responsiveness requires careful implementation and testing
- Carbon's CSS bundle is large (~200 KB) — requires tree-shaking and code-splitting to manage bundle size for low-bandwidth deployments
- IBM design language may feel corporate — some NGO and humanitarian users may prefer a less enterprise-oriented aesthetic
- Adding custom components that match Carbon's design language requires following Carbon's pattern guidelines — custom development takes longer than with unstyled primitives
- Carbon's opinionated layout system may constrain creative freedom for landing pages and public marketing content

---

## 6. Status

Accepted. IBM Carbon Design System 1.113.0 (`@carbon/react` 1.113.0) is the primary UI component library for all ZarishSphere Platform frontend applications. All Console, Marketplace, Builder, Forms engine, and domain app interfaces must use Carbon components and follow Carbon design guidance. Custom components may be added only when Carbon does not provide a suitable component, and must follow Carbon's pattern specification for visual consistency.

**Verification:** last_verified August 01, 2026 · next_review February 01, 2027 · verified_by ZarishSphere Foundation

---

## 7. Cross-references

→ **008-zs-adrs/001-adr-go-as-primary-language.md** — ADR-001: frontend stack context (React 19.2.8, Next.js 16.2.12)
→ **008-zs-adrs/006-adr-flutter-mobile-apps.md** — ADR-006: zero-cost open source requirement
→ **008-zs-adrs/009-adr-no-php-web-backends.md** — ADR-009: vendor independence
→ **007-zs-tech-stack/003-frontend-stack.md** — Frontend technology stack definition
→ **007-zs-tech-stack/006-oss-tool-catalog.md** — Open source tool catalog with pinned versions

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
