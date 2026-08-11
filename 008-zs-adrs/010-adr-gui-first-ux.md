# 010-adr-gui-first-ux.md
## ADR-010: GUI-First UX for All Ecosystem Tools
### The browser-based GUI is the primary interface — CLI and API are always secondary

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

All ZarishSphere ecosystem tools and interfaces are GUI-first. The primary user interface is a browser-based graphical application (the Console). Command-line interfaces (CLI) and programmatic APIs are secondary — always optional, never required. Every feature that is available via CLI or API must also be available via the GUI, but not vice versa: GUI-only features are permitted.

The frontend stack is React 19.2.8 with Next.js 16.2.12, compiled to a Progressive Web App (PWA) with offline support via service workers. The GUI is designed for touch, mouse, and keyboard input, and must function on low-bandwidth (<100 Kbps) and offline networks.

---

## 2. Context

Constitution Law 6 (no-code first) mandates: "Every function, configuration, workflow, and deployment process in the ZarishSphere ecosystem must be achievable through a graphical, browser-based interface before a command-line or programmatic equivalent is built."

The target users of ZarishSphere include:

- **Community health workers** in Cox's Bazar refugee camps — may have basic literacy, limited technical training, no terminal access
- **District health officers** — need to configure modules, view dashboards, generate reports without command-line knowledge
- **NGO program managers** — need to deploy and manage the platform across multiple sites without DevOps skills
- **Government administrators** — need to manage users, audit logs, and system configuration through a web interface

The founder profile (§2.4, §6.2) confirms: the founder himself prefers GUI-first interaction and needs "exact, copy-paste ready, step-by-step, numbered" CLI commands when CLI is unavoidable. If the creator of the platform needs GUI-first, the platform's users need it more.

Three interface paradigms were evaluated: GUI-first (Console primary), CLI-first (terminal primary), and API-first (programmatic primary).

---

## 3. Alternatives Considered

| Alternative | Pros | Cons |
|---|---|---|
| **GUI-first (Console primary, CLI secondary)** | Accessible to non-technical users (health workers, administrators); no terminal/command knowledge required; works on tablets and phones via browser; PWA enables offline use; React/Next.js provides modern component ecosystem; Constitution Law 6 compliance | More development effort for GUI features than equivalent CLI; browser-based apps have lower performance than native; accessibility for users with disabilities requires deliberate design; screen reader support must be maintained |
| **CLI-first (terminal primary, optional web UI)** | Faster to develop (text output is simpler than GUI); effective for automation and scripting; works over SSH (low bandwidth); natural for DevOps/sysadmin users | Inaccessible to health workers and non-technical users; violates Constitution Law 6 structurally; founder himself has "Basic — always needs copy-paste exact commands" terminal skill (founder profile §2.4); terminal on mobile devices is impractical |
| **API-first (REST/gRPC primary, GUI and CLI as clients)** | Maximum flexibility for integration; documentation-first approach (OpenAPI); supports any client type (web, mobile, IoT) | Requires every user to build or use a client; no built-in user interface; most health workers cannot consume APIs; violates Law 6 by making GUI secondary by default |
| **Native mobile app (Flutter/React Native + backend)** | Best mobile experience; push notifications; offline-first by design; camera and sensor access for field data collection | Maintenance burden: iOS + Android + web = 3 separate codebases; solo founder cannot maintain; app store distribution complexity; cannot deploy on Plane 0 without app store; updates require app store review; less critical for health records than a good web app |
| **Terminal UI (TUI — e.g., Bubble Tea in Go)** | Works over SSH, extremely low bandwidth, fast, great for system administration | Terrible for non-technical users; no images/diagrams/charts easily; limited accessibility; not suitable for forms and complex data entry; contradicts every user persona except the DevOps team (which doesn't exist) |

---

## 4. Reason for Decision

1. **Constitutional mandate:** Law 6 is a Tier II right. It states: "A feature that exists only as a CLI command, API call, or code configuration is an incomplete feature. A feature that exists only as a documented, tested GUI workflow is a complete feature." This ADR implements Law 6 as an engineering standard.

2. **Target user reality:** The primary deployment context (Cox's Bazar refugee camps, humanitarian health facilities) serves users who are not software developers. They are doctors, nurses, midwives, nutritionists, logisticians, and community health workers. A CLI-first tool is inaccessible to them. A GUI-first tool with touch-friendly forms, clear navigation, and visual data presentation is essential.

3. **Founder alignment:** The founder profile (§6.2) explicitly states "Always GUI-first, then CLI as fallback." The founder's own workflow preference aligns with the constitutional requirement. A platform designed for users who are less technical than the founder must be at least as GUI-centric as the founder needs.

4. **Offline capability:** A PWA with service workers caches application code and data for offline use. This is critical for Plane 0 (air-gapped, no internet) and Plane 1 (intermittent connectivity). Users can continue working with forms, viewing records, and entering data while offline, with sync when connectivity returns. No other interface paradigm provides this combination of rich interaction and offline resilience.

5. **Pragmatic completeness:** The GUI includes the Console (management), Builder (no-code creation), Marketplace (discovery), Apps (pre-built applications), and Forms (dynamic forms). These five interfaces cover every user workflow. The CLI (→ **010-zs-ecosystem/007-cli-spec.md** — CLI specification, secondary interface) provides automation for sysadmin tasks (bulk import/export, log analysis, system monitoring) but no user-facing feature depends on it.

---

## 5. Consequences

**Positive:**

- All ecosystem functions accessible through a web browser — no software installation required
- Health workers can use the platform on low-cost devices (Chromebooks, tablets, older phones) with a browser
- PWA + service workers enable full offline functionality for Plane 0 and 1
- React 19.2.8 + Next.js 16.2.12 provide server components for fast initial loads, client components for interactivity
- Touch-friendly UI works for field data collection on mobile devices
- CLI and API remain available for automation and integration without being mandatory

**Negative:**

- Web applications have higher resource usage than CLI tools (browser memory, ~200-500 MB for a complex SPA)
- Some users in very low-bandwidth areas (<50 Kbps) may experience slow initial page loads (mitigated by service worker caching and Next.js streaming)
- Accessibility (screen readers, keyboard navigation, high contrast) must be deliberately engineered — not automatic
- GUI development is more time-consuming than CLI: a form that takes 1 hour to build as CLI may take 8-16 hours as a polished GUI
- Browser compatibility testing across Chrome, Firefox, Safari takes ongoing effort
- Some advanced configuration options may be easier to express in YAML than in a GUI — must balance ease-of-use with flexibility

---

## 6. Status

Accepted. This ADR implements Constitution Law 6 (no-code first) and is the UX design principle for every ecosystem component. The Console (→ **010-zs-ecosystem/001-console-spec.md** — Console specification, primary user interface) is the primary user touchpoint. The CLI (→ **010-zs-ecosystem/007-cli-spec.md** — CLI specification, secondary interface) is explicitly marked as secondary in its own specification.

---

## 7. Cross-references

→ **010-zs-ecosystem/001-console-spec.md** — Console specification (primary user interface)
→ **010-zs-ecosystem/007-cli-spec.md** — CLI specification (secondary interface)
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 6 (no-code first)
→ **001-zs-meta/003-zarishsphere-founder-profile.md** — Founder GUI-first preference (§2.4, §6.2)
→ **007-zs-tech-stack/003-frontend-stack.md** — Frontend framework and UI architecture (React/Next.js)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
