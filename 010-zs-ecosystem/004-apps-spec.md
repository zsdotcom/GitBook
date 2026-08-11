# 004-apps-spec.md
## ZarishSphere Apps specification
### Pre-built domain applications

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [App catalog](#2-app-catalog)
3. [App structure](#3-app-structure)
4. [App lifecycle](#4-app-lifecycle)
5. [App manifest format](#5-app-manifest-format)
6. [Example apps](#6-example-apps)
7. [Configuration guide](#7-configuration-guide)
8. [Security boundary](#8-security-boundary)
9. [Offline behaviour](#9-offline-behaviour)
10. [Customization path](#10-customization-path)
11. [Cross-references](#11-cross-references)

---

## 1. Purpose

ZarishSphere Apps are pre-built, ready-to-use domain applications that solve real-world problems. They are deployable immediately without configuration and customizable through the Builder.

## 2. App catalog

| Domain | Example apps |
|---|---|
| Health | Patient registry, NCD tracker, immunization scheduler, disease surveillance |
| Education | Student registry, attendance tracker, learning outcome assessment |
| Logistics | Supply chain tracker, inventory manager, distribution planner |
| Protection | Case management, referral tracker, incident reporter |
| WASH | Water point mapper, sanitation tracker, hygiene promotion |
| Nutrition | Nutritional screening, supplementation tracker, MUAC monitor |
| Human rights | Violation documentation, legal aid tracker, remedy follow-up |
| Environment | Emissions inventory, compliance tracker, impact assessment |

## 3. App structure

Every app is composed of:

- **Forms** (1+) — data collection forms (FHIR Questionnaire)
- **Workflows** (0+) — approval and data flow definitions
- **Dashboards** (1+) — reporting views
- **Configuration** — app manifest, defaults, permissions

Apps are stored in `zs-content-*` repositories and listed in the Marketplace.

Service dependencies per app:

```yaml
services:
  identity: required     # User authentication and role checks
  audit: required        # Operation logging
  sync: optional         # Offline data sync between planes
  notification: optional # Email and in-app alerts
  export: optional       # Data export in open formats
```

## 4. App lifecycle

Every app follows a defined lifecycle from creation to retirement:

```
Create → Configure → Deploy → Monitor → Update → Deprecate → Remove
```

| Stage | Description | Performed by |
|---|---|---|
| **Create** | App definition authored via Builder or directly as YAML manifest | Developer / Builder |
| **Configure** | Plane-specific parameters set (DB connection, storage path, auth mode) | Deployer |
| **Deploy** | App installed to target plane through Console or CLI | Deployer |
| **Monitor** | Health checks, usage metrics, audit log reviewed | Operator |
| **Update** | New version published to Marketplace, deployed to planes | Developer |
| **Deprecate** | Old version marked deprecated in Marketplace, migration path provided | Foundation |
| **Remove** | App uninstalled from plane, data archived per retention policy | Operator |

### Deployment states

| State | Meaning |
|---|---|
| `draft` | App definition created but not yet published |
| `published` | Listed in Marketplace, available for install |
| `deployed` | Installed and running on a target plane |
| `failed` | Deployment error — check logs |
| `updating` | New version being applied |
| `deprecated` | Replaced by newer version, migration recommended |
| `removed` | Uninstalled from plane |

## 5. App manifest format

Every app includes a `manifest.yaml` file at its root:

```yaml
# manifest.yaml
apiVersion: zarishsphere.io/v1
kind: App
metadata:
  name: immunization-tracker
  displayName: "Immunization Tracker"
  domain: health
  version: "1.2.0"
  description: >
    Track immunization schedules, record administered doses,
    and generate coverage reports per WHO/EPI guidelines.
  authors:
    - name: "ZarishSphere Foundation"
      email: "foundation@zarishsphere.com"
  tags:
    - immunization
    - epi
    - who
    - child-health
  license: "Apache-2.0"

spec:
  minPlane: 0              # Minimum plane for deployment
  maxPlane: 4              # Maximum plane supported
  dependencies:
    modules:
      - health-common      # Shared health data models
      - immunization       # Domain-specific module
    apps: []
    services:
      - identity
      - audit
  forms:
    - "forms/child-registration.yaml"
    - "forms/vaccine-administration.yaml"
    - "forms/adverse-event.yaml"
  workflows:
    - "workflows/reminder-schedule.yaml"
    - "workflows/stock-alert.yaml"
  dashboards:
    - "dashboards/coverage-report.yaml"
    - "dashboards/stock-status.yaml"
  permissions:
    required:
      - "immunization:record"
      - "immunization:read"
      - "patient:read"
    admin:
      - "immunization:admin"
      - "immunization:config"
  config:
    defaults:
      scheduleReminderDays: 7
      autoSyncInterval: "5m"
    secrets:
      - "api.who-service-key"  # Optional external API key
```

The manifest schema is validated by the Engine at deploy time. All references to modules and forms must resolve to entries in the Marketplace or local content repositories.

## 6. Example apps

### Immunization Tracker

Tracks childhood immunization schedules per WHO/EPI guidelines. Records doses, generates reminder letters, produces coverage reports.

- **Forms:** Child registration, vaccine administration, adverse event reporting
- **Workflows:** Overdue reminder, stock level alert
- **Dashboards:** Coverage by antigen, dropout rate, stock status
- **Target planes:** 0-4 (full offline support at Plane 0)
- **Cross-ref:** → **[010-zs-ecosystem/010-modules-spec.md](010-zs-ecosystem/010-modules-spec.md)** — depends on `health-common` and `immunization` modules

### Nutrition Screening

Screens children under 5 for acute malnutrition using MUAC, weight-for-height, and edema assessment.

- **Forms:** MUAC measurement, dietary diversity questionnaire, referral form
- **Workflows:** Positive referral, default tracer
- **Dashboards:** MUAC trend, admission vs discharge, cure rate
- **Target planes:** 0-4
- **Cross-ref:** → **[010-zs-ecosystem/005-forms-spec.md](010-zs-ecosystem/005-forms-spec.md)** — form engine rendering screening forms

### Supply Chain Monitor

Tracks essential medicine and supply inventory from central warehouse to distribution point.

- **Forms:** Stock receipt, distribution log, wastage report
- **Workflows:** Stock-out alert, expiry warning, resupply request
- **Dashboards:** Stock status by facility, consumption trends, order fulfilment rate
- **Target planes:** 1-4 (requires connectivity for stock-level synchronization)

## 7. Configuration guide

Apps accept configuration at three levels:

### Level 1: Default configuration (shipped with app)

Embedded in the app manifest under `spec.config.defaults`. These are sensible defaults for typical deployments.

### Level 2: Plane-specific overrides

Applied during deployment via the Console or CLI:

```bash
zs app deploy immunization-tracker \
    --plane 1 \
    --config ./configs/edge-clinic.yaml
```

Example plane override:

```yaml
# configs/edge-clinic.yaml
plane: 1
config:
  scheduleReminderDays: 3          # Shorter reminder window for mobile clinic
  autoSyncInterval: "2m"           # More frequent sync for unstable connectivity
  forms:
    vaccine-administration:
      requireWeight: false          # Skip weight field when scale unavailable
```

### Level 3: Environment variable injection

Sensitive configuration (database passwords, API keys) is injected via environment variables managed by the System layer's encrypted config store. Secrets are referenced in the manifest under `spec.config.secrets`:

```yaml
config:
  defaults:
    reminderFromAddress: "noreply@zarishsphere.com"
  secrets:
    - "smtp.password"
    - "who.service-key"
```

At Plane 0, secrets are stored in an encrypted SQLite database with a master key provisioned during setup. At Planes 1-4, secrets are stored in the System layer's vault-backed secrets engine.

## 8. Security boundary

Each app operates within a defined security boundary that enforces Module Sovereignty (Law 7):

| Boundary | Restriction |
|---|---|
| **Module access** | An app can only access modules declared in `spec.dependencies.modules`. Unlisted modules are inaccessible. |
| **Data scope** | An app can only read and write data for its own module contexts. Cross-module data access requires explicit permission. |
| **Service access** | An app uses only the services listed in `spec.dependencies.services`. Unlisted services are blocked. |
| **Network access** | Outbound network connections are restricted to URLs listed in `spec.config.allowedDomains`. Plane 0 has no outbound access by default. |
| **File system** | App data is isolated to a plane-local directory. Cross-app file access is prohibited. |
| **Admin scope** | Admin-level permissions (`spec.permissions.admin`) are gated by the Identity service. Only users with the `app_admin` role can modify app configuration. |

The security boundary is enforced by the Engine at deployment time and audited by the Audit service at runtime. Violations are logged and alerts are sent to the designated operator.

## 9. Offline behaviour

All apps are designed for offline operation from Plane 0 upward:

### Plane 0 (air-gapped)

- Fully functional with no network connectivity
- Data stored in local SQLite database
- Forms rendered by local form engine (PWA cache)
- Sync bundles prepared for USB transfer
- Notifications displayed in-app only
- Authentication via local accounts

### Plane 1 (intermittent connectivity)

- Web app served from local PWA cache
- Data synchronized periodically (configurable interval)
- Forms cached locally, updated on sync
- Queued notifications delivered on next sync

### Plane 2+ (continuous connectivity)

- Full real-time operation
- All services available
- Live dashboards and notifications
- Continuous data replication

### Degraded mode

If connectivity is lost at Plane 2+, the app falls back to Plane 1 behaviour automatically:

- Write operations continue locally
- Reads serve from local cache
- Sync queue builds up
- Full resumption when connectivity returns
- No data loss during transition

## 10. Customization path

Users can customize any app through the Builder:

1. Install the app from the Marketplace
2. Open the app in the Builder
3. Modify forms, workflows, or dashboards via GUI
4. Save as a custom version
5. Deploy the customized app

No code editing required at any step. Customized apps retain compatibility with future updates through a three-way merge process:

- Base version (original app)
- Local modifications (customizations)
- New upstream version
- Merged result (presented in Builder for review)

## 11. Cross-references

→ **010-zs-ecosystem/010-modules-spec.md** — Modules specification: the domain modules apps declare in their manifests
→ **010-zs-ecosystem/005-forms-spec.md** — Forms specification: the form engine rendering app forms
→ **010-zs-ecosystem/003-builder-spec.md** — Builder specification: the customization path for pre-built apps
→ **010-zs-ecosystem/002-marketplace-spec.md** — Marketplace specification: app listing and installation
→ **010-zs-ecosystem/012-engine-spec.md** — Engine specification: validates app manifests at deployment time
→ **010-zs-ecosystem/009-services-spec.md** — Services specification: the identity, audit, sync, notification, and export services apps depend on
→ **003-zs-platform/003-deployment-planes.md** — Deployment plane model: offline behaviour and connectivity per plane
→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 7: module sovereignty enforced by the app security boundary

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
