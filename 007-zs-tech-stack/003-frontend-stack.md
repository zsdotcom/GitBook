# 003-frontend-stack.md
## Frontend framework and UI architecture
### React 19.2.8, Next.js 16.2.12, TypeScript, Carbon Design System

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Framework choice](#1-framework-choice)
2. [Styling and component system](#2-styling-and-component-system)
3. [State management](#3-state-management)
4. [Offline support](#4-offline-support)
5. [Forms engine integration](#5-forms-engine-integration)
6. [Build tooling and development](#6-build-tooling-and-development)
7. [Accessibility](#7-accessibility)
8. [Dashboard layout for Console](#8-dashboard-layout-for-console)
9. [Cross-references](#9-cross-references)

---

## 1. Framework choice

### 1.1 Next.js 16.2.12 with App Router

Next.js 16.2.12 is the meta-framework for all ZarishSphere frontend applications. Selected for its hybrid rendering model and zero-cost hosting on Cloudflare Pages.

| Feature | Next.js 16.2.12 | Alternative | Why Next.js wins |
|---|---|---|---|
| Rendering | SSR, SSG, ISR, Streaming | Vite + React Router | All from one framework |
| Hosting | Cloudflare Pages | Vercel (paid) | Free tier on Cloudflare |
| Server components | Built-in | Manual in React | Reduced client JS size |
| Image optimization | Built-in | Manual | No extra tooling |
| Middleware | Edge and server | Express proxy | Unified in Next.js |
| API routes | Built-in | Express server | Server and API in one deploy |

> **Constraint:** All frontend applications must deploy to Cloudflare Pages free tier (with GitHub Pages for static documentation sites). No Vercel-specific features may be used. Any feature that requires a paid hosting plan is rejected.

### 1.2 Frontend applications

| Application | Route prefix | Rendering strategy | Purpose |
|---|---|---|---|
| Console | `/console/*` | Client + Server components | Management dashboard, module admin, user management |
| Forms engine | `/forms/*` | Client-side rendering | Dynamic form rendering from YAML/JSON |
| Public site | `/` (root) | Static site generation | Documentation, landing pages, blog |
| Marketplace | `/marketplace/*` | SSR + ISR | Module discovery, one-click deploy |
| Builder | `/builder/*` | Client-side rendering | No-code form and workflow creation |

### 1.3 Package structure

```
zs-frontend/
├── apps/
│   ├── console/          — Console application
│   ├── forms/            — Forms engine
│   ├── marketplace/      — Marketplace
│   ├── builder/          — No-code builder
│   └── site/             — Public website
├── packages/
│   ├── ui/               — Shared Carbon + shadcn/ui components
│   ├── forms-engine/     — Dynamic form renderer
│   ├── api-client/       — Go backend API client
│   ├── auth/             — Auth context and hooks
│   └── config/           — Shared TypeScript config
├── pnpm-workspace.yaml
├── tsconfig.json
└── package.json
```

---

## 2. Styling and component system

### 2.1 Carbon Design System (primary)

Carbon Design System is the primary UI component library, provided by `@carbon/react` 1.113.0 (Apache 2.0, WCAG 2.2 AA). It supplies the healthcare-focused, accessible component set used by the Console, Marketplace, Builder, and Forms engine.

| Component category | Components | Status |
|---|---|---|
| Forms | TextInput, NumberInput, Select, Checkbox, RadioButton, DatePicker, Toggle | V1 |
| Data display | DataTable, StructuredList, Card, Tag, ProgressIndicator, Skeleton | V1 |
| Navigation | Header, SideNav, Tabs, Breadcrumb, OverflowMenu | V1 |
| Feedback | ToastNotification, InlineNotification, Modal, Tooltip, Loading | V1 |
| Layout | Grid, Flex, Column, Content, Layer | V1 |

Design tokens and theming are consumed through Carbon's SCSS variables and CSS custom properties; the platform keeps its own token layer for the brand palette (primary `#0A5C4A`) on top.

### 2.2 Tailwind CSS 4.3.3 (content surfaces)

Tailwind CSS 4.3.3 provides utility-first styling with zero runtime overhead for content surfaces (public site, marketing pages, documentation). Configuration uses CSS-based theming instead of `tailwind.config.js`.

```css
/* app/globals.css */
@import "tailwindcss";

@theme {
  --color-primary: #0A5C4A;
  --color-primary-light: #1A7C6A;
  --color-surface: #F8FAFC;
  --color-surface-alt: #F1F5F9;
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
}
```

### 2.3 Shadcn UI (content surfaces)

Shadcn UI provides accessible, copy-paste React components built on Radix UI primitives for content surfaces alongside Tailwind.

| Component category | Components | Status |
|---|---|---|
| Forms | Button, Input, Select, Checkbox, Radio, Textarea, Switch | V1 |
| Data display | Table, Card, Badge, Avatar, Progress, Skeleton | V1 |
| Navigation | Tabs, Breadcrumb, Dropdown Menu, Sidebar | V1 |
| Feedback | Toast, Alert, Dialog, Sheet, Tooltip | V1 |
| Layout | Container, Grid, Separator, Scroll Area | V1 |
| Advanced | Command (kbar), Combobox, Date Picker, Data Table | V1.1 |

### 2.4 Design tokens

| Token | Value | Usage |
|---|---|---|
| Font family | Inter (system-ui fallback) | All text |
| Monospace | JetBrains Mono | Code, identifiers |
| Heading scale | 16 / 20 / 24 / 30 / 36 px | Typography |
| Spacing | 4px base (Tailwind defaults) | Layout |
| Breakpoints | 640 / 768 / 1024 / 1280 / 1536 px | Responsive design |

### 2.5 Theme support

- Light theme: default for all deployments
- Dark theme: available as toggle, respects `prefers-color-scheme`
- High contrast: WCAG AAA variant for accessibility
- Themes stored in CSS custom properties, toggled via `class="dark"` on `<html>`

---

## 3. State management

### 3.1 No Redux decision

> **Constraint:** Redux, Zustand, Jotai, and similar external client-state management libraries are not used. Client-side application state is managed through React 19.2.8 hooks, Context, and Next.js server components. The Carbon Design System consumes React Context for theming and layout state.

### 3.2 State architecture

| State type | Mechanism | Examples |
|---|---|---|
| Server state | Next.js fetch + React cache, TanStack Query v5.64.0 | FHIR resources, module list, user data |
| Global UI state | React Context | Theme, sidebar open/closed, auth session |
| Local UI state | useState + useReducer | Form input, modal open, dropdown |
| Form state | React Hook Form + Zod 4.4.3 | Form validation, submission state |
| URL state | Next.js search params | Filters, pagination, search queries |

### 3.3 Context structure

```typescript
// Auth context
interface AuthContextType {
  user: User | null;
  session: Session | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

// Theme context
interface ThemeContextType {
  theme: 'light' | 'dark' | 'high-contrast';
  setTheme: (theme: string) => void;
}

// Console context
interface ConsoleContextType {
  activeModule: string | null;
  breadcrumbs: Breadcrumb[];
  notifications: Notification[];
}
```

---

## 4. Offline support

### 4.1 Service worker

All frontend applications include a service worker for offline access:

- **Cache-first** strategy for static assets (JS, CSS, fonts, images)
- **Network-first** strategy for API calls (fallback to cached response)
- **Background sync** for form submissions when offline
- **Precache** critical pages on first load

### 4.2 IndexedDB for offline data

```typescript
interface OfflineStore {
  // Cached FHIR resources for read access
  fhirCache: {
    resourceType: string;
    id: string;
    resource: object;
    cachedAt: number;
  }[];
  
  // Pending form submissions
  pendingSubmissions: {
    formId: string;
    patientId: string;
    data: object;
    queuedAt: number;
    retryCount: number;
  }[];
  
  // Module definitions for offline module browsing
  moduleCache: {
    moduleId: string;
    manifest: object;
    cachedAt: number;
  }[];
}
```

### 4.3 Connectivity detection

```typescript
// Hook for connectivity-aware components
function useConnectivity(): {
  isOnline: boolean;
  lastOnlineAt: Date | null;
  pendingSyncCount: number;
  syncNow: () => Promise<void>;
}
```

---

## 5. Forms engine integration

### 5.1 Dynamic form rendering

The forms engine receives YAML or JSON form definitions and renders them as interactive forms using React 19.2.8 dynamic component loading.

```typescript
interface FormDefinition {
  id: string;
  title: string;
  pages: FormPage[];
  resources: FormResourceBinding[];
  validation: FormValidation[];
}

interface FormPage {
  title: string;
  fields: FormField[];
  condition?: string;  // Conditional visibility expression
}

interface FormField {
  type: 'text' | 'number' | 'select' | 'date' | 'reference' | 'group' | 'table';
  id: string;
  label: string;
  required: boolean;
  fhirPath: string;  // Maps to FHIR Questionnaire path
  options?: FieldOption[];
  validation?: ValidationRule[];
}
```

### 5.2 FHIR Questionnaire binding

Forms are bound to FHIR Questionnaire resources:

```
Form Definition (YAML/JSON)  →  Questionnaire Resource  →  QuestionnaireResponse
        ↕                              ↕                           ↕
   Builder UI                   Server validation          FHIR storage
```

### 5.3 Form submission flow

```
1. User opens form (loaded from YAML or Questionnaire)
2. Form engine renders dynamic fields
3. User fills fields with validation feedback
4. On submit: data → QuestionnaireResponse JSON
5. POST to zs-fhir-server /QuestionnaireResponse
6. Server validates against form definition
7. Return success / OperationOutcome
```

### 5.4 Cross-reference

→ **007-zs-tech-stack/005-no-code-tools.md** — No-code tooling specification for the forms engine
→ **010-zs-ecosystem/005-forms-spec.md** — Forms engine component specification
→ **005-zs-standards/013-form-schema-specification.md** — ZARISH-STANDARDS form schema

---

## 6. Build tooling and development

### 6.1 Toolchain

| Tool | Version | Purpose |
|---|---|---|
| Node.js | 22 LTS | JavaScript runtime |
| pnpm | 10 | Package manager (monorepo) |
| Vite | 8.2.0 | Dev server and build tool |
| TypeScript | 7.0.2 | Type checking (strict mode) |
| Zod | 4.4.3 | TypeScript-first schema validation |
| ESLint | 9 | Code quality and linting |
| Prettier | 3 | Code formatting |
| Playwright | 1.52.0 | End-to-end browser testing |
| Vitest | 3.1.0 | Unit and integration testing |

### 6.2 TypeScript strict mode

All frontend packages use `strict: true` in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "preserve",
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

### 6.3 Development workflow

```bash
# Install dependencies
pnpm install

# Start all applications in dev mode
pnpm dev

# Type check all packages
pnpm typecheck

# Lint all packages
pnpm lint

# Run tests
pnpm test

# Build for production
pnpm build
```

---

## 7. Accessibility

### 7.1 WCAG 2.1 AA target

All ZarishSphere frontend applications target WCAG 2.1 AA compliance (Carbon Design System ships WCAG 2.2 AA components):

| Requirement | Implementation |
|---|---|
| Color contrast | 4.5:1 for normal text, 3:1 for large text |
| Keyboard navigation | All interactive elements reachable via Tab |
| Focus indicators | Visible focus ring on all interactive elements |
| Screen reader | ARIA labels, roles, live regions |
| Form labels | All inputs have associated labels |
| Error announcements | Form errors announced via aria-live |
| Heading hierarchy | Single h1 per page, sequential h2-h6 |
| Alternative text | All images have alt text |

### 7.2 Components with accessibility built in

Carbon Design System and shadcn/ui components include ARIA attributes by default. Custom components must pass an accessibility review before inclusion in the shared `ui` package.

### 7.3 Automated testing

```bash
# Run accessibility audit
npx @axe-core/cli https://console.zarishsphere.com

# Run Playwright a11y checks
pnpm test:e2e --a11y
```

---

## 8. Dashboard layout for Console

### 8.1 Layout structure

```
┌─────────────────────────────────────────────────┐
│  Top navigation bar    [Search] [Theme] [User]  │
├──────────┬──────────────────────────────────────┤
│          │                                      │
│  Sidebar │      Main content area               │
│  (240px) │                                      │
│          │  ┌─────┬─────┬─────┬─────┐           │
│  Home    │  │ Mod │ Pat │ Rep │ Alrt│           │
│  Modules │  ├─────┼─────┼─────┼─────┤           │
│  Patients│  │ 12  │ 342 │ 89  │ 3   │           │
│  Reports │  └─────┴─────┴─────┴─────┘           │
│  Settings│                                      │
│          │  Recent activity / data table         │
│          │                                      │
│          │  ┌─────────────────────────────┐     │
│          │  │                             │     │
│          │  │     Analytics chart          │     │
│          │  │                             │     │
│          │  └─────────────────────────────┘     │
├──────────┴──────────────────────────────────────┤
│  Footer: version, sync status, connection        │
└─────────────────────────────────────────────────┘
```

### 8.2 Responsive behavior

| Breakpoint | Sidebar | Layout |
|---|---|---|
| ≥1280px (xl) | Visible, 280px | Sidebar + main |
| 768-1279px (md) | Collapsible (hamburger) | Sidebar slides over |
| <768px (sm) | Hidden (bottom nav) | Single column |

### 8.3 Key dashboard components

| Component | Description |
|---|---|
| StatCard | Metric card with icon, value, trend, and click handler |
| DataTable | Sortable, filterable, paginated table (Carbon DataTable) |
| ActivityFeed | Scrollable chronological event list |
| QuickActions | Context-sensitive action buttons |
| StatusBadge | Color-coded status indicator (online, offline, error, warning) |
| SyncIndicator | Shows last sync time, pending changes, connection status |
| ModuleCard | Card displaying module name, version, status, and actions |
| NotificationToast | Non-blocking notification with action button |

### 8.4 Console application specifics

The Console is the primary browser-based management interface for the ZarishSphere ecosystem. It provides:

- **Module management**: Install, configure, monitor domain modules
- **Patient management**: Browse, search, create patient records
- **Form management**: Deploy and manage form definitions
- **Report dashboard**: View analytics and export data
- **User management**: Create and manage local user accounts
- **System settings**: Configure deployment, sync, backup

See → **010-zs-ecosystem/001-console-spec.md** — the complete Console specification.

---

## 9. Cross-references

→ **007-zs-tech-stack/001-tech-stack-master.md** — Master production tech stack mapping
→ **007-zs-tech-stack/006-oss-tool-catalog.md** — Authoritative, version-pinned OSS tool catalog
→ **010-zs-ecosystem/001-console-spec.md** — Console application specification
→ **010-zs-ecosystem/005-forms-spec.md** — Forms engine specification
→ **010-zs-ecosystem/003-builder-spec.md** — Builder application specification
→ **005-zs-standards/013-form-schema-specification.md** — ZARISH-STANDARDS form schema
→ **008-zs-adrs/019-adr-carbon-design-system.md** — ADR-019: Carbon Design System as primary UI library
→ **008-zs-adrs/022-adr-typescript-strict-mode.md** — ADR-022: TypeScript strict mode
→ **008-zs-adrs/020-adr-microfrontend-architecture.md** — ADR-020: microfrontend architecture
→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS documentation and formatting rules
→ **006-zs-infrastructure/003-cloudflare-architecture.md** — Cloudflare Pages hosting and edge delivery

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
