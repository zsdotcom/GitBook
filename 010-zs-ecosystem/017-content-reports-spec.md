# 017-content-reports-spec.md
## zs-content-reports specification
### Report templates and dashboard definitions

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Repository structure](#2-repository-structure)
3. [Report definition format](#3-report-definition-format)
4. [Dashboard definition format](#4-dashboard-definition-format)
5. [Data sources and filters](#5-data-sources-and-filters)
6. [Report lifecycle](#6-report-lifecycle)
7. [Plane 0 operation](#7-plane-0-operation)
8. [Cross-references](#8-cross-references)

---

## 1. Purpose

The `zs-content-reports` repository stores, versions, and distributes report templates and dashboard configurations for the ZarishSphere ecosystem. Reports enable users to visualize data collected through forms and modules across all 40 domains.

Key design principles:

- **YAML-defined** — reports and dashboards are declarative YAML files
- **Domain-agnostic** — any report type can serve any domain
- **Plane-portable** — the same report definition works offline and online
- **Versioned** — every report template is immutable once published

## 2. Repository structure

```
zs-content-reports/
├── reports/
│   ├── health/
│   │   ├── ncd-prevalence-monthly-v1.yaml
│   │   ├── immunization-coverage-v2.yaml
│   │   └── maternal-outcomes-v1.yaml
│   ├── education/
│   │   ├── enrollment-by-grade-v1.yaml
│   │   └── attendance-rate-v1.yaml
│   ├── logistics/
│   │   ├── stock-levels-v1.yaml
│   │   └── supply-chain-performance-v2.yaml
│   ├── protection/
│   │   ├── case-load-by-type-v1.yaml
│   │   └── referral-completion-rate-v1.yaml
│   └── cross-domain/
│       ├── donor-dashboard-v1.yaml
│       └── executive-summary-v2.yaml
├── dashboards/
│   ├── health-program-overview.yaml
│   ├── field-operations.yaml
│   └── executive-leadership.yaml
├── visualizations/
│   ├── chart-templates/
│   │   ├── bar-chart.yaml
│   │   ├── line-chart.yaml
│   │   ├── pie-chart.yaml
│   │   ├── heatmap.yaml
│   │   ├── geo-map.yaml
│   │   └── gauge.yaml
│   └── custom/
│       └── organ-transplant-donut.yaml
├── schemas/
│   ├── report-schema.json
│   ├── dashboard-schema.json
│   └── visualization-schema.json
├── examples/
│   └── health-dashboard-complete.yaml
└── tests/
    ├── validate-reports.go
    └── test-fixtures/
```

### 2.1 Directory conventions

| Directory | Purpose |
|---|---|
| `reports/{domain}/` | Individual report templates by domain |
| `dashboards/` | Multi-report dashboard configurations |
| `visualizations/` | Reusable chart type and visualization definitions |
| `schemas/` | JSON Schema for report and dashboard validation |
| `examples/` | Complete working examples |
| `tests/` | Automated validation and rendering tests |

## 3. Report definition format

### 3.1 Structure

```yaml
# ncd-prevalence-monthly-v1.yaml
report:
  id: zs-ncd-prevalence-monthly-v1
  name: NCD Prevalence — Monthly
  version: 1.0.0
  status: active
  domain: health

  schedule:
    frequency: monthly
    day: 1

  data_source:
    type: fhir
    resource: Observation
    filter:
      code: "http://loinc.org|LP7803-7"  # NCD-related observations
      date_range: last_month

  dimensions:
    - field: condition.code
      label: Condition Type
    - field: encounter.location
      label: Clinic
    - field: patient.gender
      label: Gender

  measures:
    - field: patient.id
      aggregation: count_distinct
      label: Total Patients
    - field: observation.value
      aggregation: avg
      label: Average Value

  charts:
    - id: chart-1
      type: bar
      title: NCD Cases by Condition
      x: condition.code
      y: count

    - id: chart-2
      type: pie
      title: Gender Distribution
      x: patient.gender
      y: count

  export_formats:
    - pdf
    - csv
    - json
    - fhir
```

### 3.2 Report types

| Report type | Description | Use case |
|---|---|---|
| Tabular | Grid with columns and rows | Data tables, line lists |
| Summary | Aggregated metrics with totals | Monthly summaries, KPIs |
| Trend | Time-series visualizations | Disease trends over time |
| Comparative | Side-by-side comparisons | Facility comparison, before/after |
| Geographic | Map-based visualizations | Disease mapping, coverage heatmaps |
| Compliance | Standards adherence tracking | WHO guideline compliance |

## 4. Dashboard definition format

```yaml
# health-program-overview.yaml
dashboard:
  id: zs-health-program-overview-v1
  name: Health Program Overview
  version: 1.0.0
  status: active

  layout:
    type: grid
    columns: 3

  panels:
    - id: panel-1
      title: Total Patients
      type: metric
      data_source:
        report: zs-patient-count-v1
      width: 1
      height: 1

    - id: panel-2
      title: NCD Prevalence by Clinic
      type: chart
      data_source:
        report: zs-ncd-prevalence-monthly-v1
        chart: chart-1
      width: 2
      height: 2

    - id: panel-3
      title: Immunization Coverage
      type: chart
      data_source:
        report: zs-immunization-coverage-v2
      width: 1
      height: 2

    - id: panel-4
      title: Recent Alerts
      type: table
      data_source:
        report: zs-alert-log-v1
      width: 3
      height: 1

  refresh_interval: 300
  auto_refresh: true
```

## 5. Data sources and filters

### 5.1 Supported data source types

| Source type | Description | Example |
|---|---|---|
| `fhir` | FHIR resource query | Patient, Observation, Encounter |
| `form` | Form submission data | QuestionnaireResponse |
| `module` | Module-specific data | Stock levels from logistics module |
| `system` | System metrics | Audit log, error counts |
| `external` | Third-party API | DHIS2 aggregate indicators |

### 5.2 Filter operations

| Operator | Description |
|---|---|
| `eq` | Equal to |
| `neq` | Not equal to |
| `gt` | Greater than |
| `gte` | Greater than or equal |
| `lt` | Less than |
| `lte` | Less than or equal |
| `in` | In list |
| `between` | Between two values |
| `contains` | String contains |
| `date_range` | Date range filter |

## 6. Report lifecycle

```
[Draft] → [Review] → [Published] → [Deprecated]
```

| Stage | Description |
|---|---|
| Draft | Initial development, may change |
| Review | Under review by domain experts |
| Published | Ready for production use |
| Deprecated | Superseded by newer version |

> **Constraint:** The `latest` tag must never be used. Consumers must pin to a specific semantic version.

## 7. Plane 0 operation

At Plane 0:

- Reports render using locally cached data — no network queries
- Dashboards use pre-computed aggregates stored in the local database
- Export formats (PDF, CSV) generate entirely on-device
- Charts render via embedded charting library — no external CDN
- Geo-map visualizations use bundled tile sets or are disabled
- Report scheduling uses a local timer — no cron dependency

## 8. Cross-references

→ **010-zs-ecosystem/001-console-spec.md** — Console specification: the primary interface that renders reports and dashboards
→ **010-zs-ecosystem/012-engine-spec.md** — Engine specification: executes the queries behind report data sources
→ **010-zs-ecosystem/013-system-spec.md** — System specification: the local database that stores the pre-computed aggregates dashboards use at Plane 0
→ **010-zs-ecosystem/010-modules-spec.md** — Modules specification: the domain modules whose data reports visualize
→ **010-zs-ecosystem/014-content-forms-spec.md** — zs-content-forms specification: form submission data consumed as a `form`-type data source
→ **010-zs-ecosystem/015-content-protocols-spec.md** — zs-content-protocols specification: protocol definitions used for compliance reporting
→ **010-zs-ecosystem/016-content-templates-spec.md** — zs-content-templates specification: distribution packages that bundle report templates
→ **010-zs-ecosystem/019-fhir-hub-spec.md** — FHIR Integration Hub specification: the DHIS2 adapter that feeds external aggregate indicators
→ **003-zs-platform/008-domain-registry.md** — Domain registry: the 40-domain taxonomy behind the `reports/{domain}/` directory structure
→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS §4.3: the `latest` tag prohibition applied to report version pinning

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
