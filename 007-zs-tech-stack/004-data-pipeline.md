# 004-data-pipeline.md
## Data pipeline specification
### Go-native ETL, PostgreSQL storage, analytics export, and local reporting

**Document type:** Specification
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Architecture overview](#1-architecture-overview)
2. [Data sources](#2-data-sources)
3. [Pipeline architecture](#3-pipeline-architecture)
4. [Batch processing](#4-batch-processing)
5. [Scheduled sync](#5-scheduled-sync)
6. [Export formats](#6-export-formats)
7. [Reporting](#7-reporting)
8. [Data retention and archiving](#8-data-retention-and-archiving)
9. [Cross-references](#9-cross-references)

---

## 1. Architecture overview

The ZarishSphere data pipeline is a lightweight, Go-native ETL system designed for offline and resource-constrained environments. It transforms operational FHIR data into analytics-ready formats without requiring any JVM-based infrastructure.

### 1.1 Design principles

| Principle | Rationale |
|---|---|
| Single binary ETL | No Spark, Flink, Kafka, or JVM tools. Pure Go. |
| Async backbone | NATS 2.14.4 JetStream carries inter-service events and pipeline triggers on Planes 1+. |
| Offline transport | Data moves as files (CSV, JSON, Parquet) on Plane 0 and as a fallback on all planes. |
| Offline-first | Full pipeline runs on air-gapped systems. Sync is optional. |
| Source of truth in PostgreSQL | PostgreSQL 18.4 is the canonical store (SQLite on Planes 0/1). All export formats are derived. |
| Append-only audits | All transformations log to an immutable audit table. |

> **Constraint:** No Kafka, JMS, RabbitMQ, or any JVM-based message broker may be introduced. NATS 2.14.4 JetStream is the async backbone on Planes 1+; on Plane 0 data moves between pipeline stages via file writes or direct database reads.

### 1.2 High-level data flow

```
┌──────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Data Sources │────▶│  Go ETL Core │────▶│ PostgreSQL 18.4 │
│              │     │              │     │ / SQLite (0-1)  │
│  FHIR Server │     │  Validate    │     │                 │
│  ZARISH-INDEX│     │  Transform   │     │  FHIR resources │
│  ZARISH-STDS │     │  Aggregate   │     │  Metrics tables │
│  App usage   │     │  Resolve     │     │  Audit log      │
└──────────────┘     └──────────────┘     │  Metadata       │
                              │            └────────┬────────┘
                              │ NATS JetStream       │
                              │ (Planes 1+)          ▼
                              │            ┌─────────────────┐
                              └───────────▶│  Export Engine   │
                                           │                 │
                                           │  CSV / JSON     │
                                           │  FHIR Bundle    │
                                           │  Parquet        │
                                           │  Go Templates   │
                                           └─────────────────┘
```

---

## 2. Data sources

### 2.1 Source registry

| Source | Format | Ingestion method | Frequency |
|---|---|---|---|
| FHIR server | FHIR JSON (Bundle) | Direct database read | Real-time (triggered) |
| ZARISH-INDEX | JSON / YAML | File import | On schedule |
| ZARISH-STANDARDS | YAML | File import | On schedule |
| App usage logs | JSON lines | File watch | Every 5 minutes |
| Form submissions | FHIR QuestionnaireResponse | Direct database read | Real-time |
| Module configuration | YAML | File import | On deploy |
| External CSV import | CSV | Upload + parse | Manual or scheduled |

### 2.2 FHIR server integration

The data pipeline reads directly from the FHIR server's database (PostgreSQL 18.4 on Planes 2+, SQLite on Planes 0/1) for operational analytics. This avoids dual-write patterns.

```go
// ETL reads from the FHIR database directly
type FHIRSource struct {
    db *sql.DB
    
    // Resource types to extract
    resourceTypes []string
    
    // Date range filter for incremental loads
    lastExtracted time.Time
}

func (s *FHIRSource) Extract(ctx context.Context) ([]ResourceRow, error) {
    rows, err := s.db.QueryContext(ctx, `
        SELECT resource_type, resource_json, last_updated
        FROM fhir_resources
        WHERE last_updated > ?
        AND is_deleted = 0
        ORDER BY last_updated
    `, s.lastExtracted)
    // ...
}
```

### 2.3 ZARISH-INDEX and ZARISH-STANDARDS ingestion

These sources are ingested as file imports:

```
1. CI/CD publishes updated export to GitHub
2. systemd timer triggers zs-data-import
3. Importer downloads/reads the current ZARISH-INDEX export
4. Parse and validate against metadata schema
5. Upsert into PostgreSQL / SQLite index tables
6. Log import results to audit table
```

---

## 3. Pipeline architecture

### 3.1 ETL component

The pipeline core is a single Go binary (`zs-etl`) with subcommands for each stage:

```bash
# Extract data from sources
zs-etl extract --source=fhir --since=2026-06-01

# Transform and aggregate
zs-etl transform --config=pipeline/ncd-indicators.yml

# Load into analytics tables
zs-etl load --target=analytics.db

# Full pipeline run
zs-etl run --config=pipeline/daily.yml

# Export to formats
zs-etl export --format=parquet --output=/exports/
```

### 3.2 Pipeline configuration

ETL pipelines are configured with YAML:

```yaml
# pipeline/daily-ncd-report.yml
name: "Daily NCD Indicators"
schedule: "0 2 * * *"  # Daily at 2 AM

sources:
  - name: fhir-server
    type: fhir
    resources:
      - Observation
      - Condition
      - Encounter
      - Patient
    filter:
      last_updated: "-24h"

transforms:
  - name: ncd-indicators
    type: aggregate
    query: |
      SELECT 
        enc.period_start AS date,
        pat.address_city AS district,
        COUNT(DISTINCT enc.subject_id) AS unique_patients,
        COUNT(DISTINCT CASE WHEN cond.code LIKE 'I10%' THEN enc.subject_id END) AS hypertension,
        COUNT(DISTINCT CASE WHEN cond.code LIKE 'E11%' THEN enc.subject_id END) AS diabetes
      FROM encounters enc
      JOIN patients pat ON enc.subject_id = pat.id
      LEFT JOIN conditions cond ON cond.subject_id = pat.id
      GROUP BY enc.period_start, pat.address_city

exports:
  - format: csv
    path: /exports/ncd-indicators.csv
  - format: parquet
    path: /exports/ncd-indicators.parquet
  - format: dashboard
    template: ncd-dashboard.tmpl
    path: /exports/dashboards/ncd-dashboard.html
```

### 3.3 Analytics database

Separate from the FHIR operation database, an analytics database stores aggregated data (PostgreSQL 18.4 on Planes 2+, SQLite on Planes 0/1):

```
analytics.db
├── daily_indicators
├── monthly_aggregates
├── facility_summary
├── patient_demographics
├── form_completion_rates
├── module_usage_stats
├── sync_audit_log
└── etl_run_history
```

### 3.4 Audit and lineage

Every ETL run is recorded:

```sql
CREATE TABLE etl_run_history (
    run_id        TEXT PRIMARY KEY,
    pipeline_name TEXT NOT NULL,
    started_at    TEXT NOT NULL,
    completed_at  TEXT,
    status        TEXT NOT NULL,  -- 'running', 'completed', 'failed'
    records_read  INTEGER,
    records_written INTEGER,
    error_message TEXT,
    config_snapshot TEXT         -- Full pipeline config at time of run
);

CREATE TABLE transform_lineage (
    id            INTEGER PRIMARY KEY AUTOINCREMENT,
    run_id        TEXT NOT NULL,
    source_table  TEXT NOT NULL,
    target_table  TEXT NOT NULL,
    records_affected INTEGER,
    transform_sql TEXT
);
```

---

## 4. Batch processing

### 4.1 Batch model

All data processing is batch-oriented. Stream processing is not needed for the scale and deployment context.

| Characteristic | Value |
|---|---|
| Processing model | Batch (incremental or full) |
| Window size | Configurable per pipeline (15m, 1h, 1d) |
| State management | `last_extracted` timestamp + offset sampling |
| Error handling | Fail-stop per batch, configurable retry (3 attempts) |
| Idempotency | All transforms are idempotent (upsert pattern) |

### 4.2 Incremental vs full loads

```yaml
pipeline:
  mode: incremental  # or 'full'
  
  # For incremental: track last successful run
  watermark:
    column: last_updated
    table: fhir_resources
  
  # For full: truncate and reload
  full_load:
    schedule: "0 0 1 * *"  # First of month
    truncate_tables:
      - monthly_aggregates
```

### 4.3 Bulk processing limits

| Resource | Max batch size | Memory per batch |
|---|---|---|
| Patient | 10,000 | ~50 MB |
| Observation | 50,000 | ~200 MB |
| Encounter | 10,000 | ~80 MB |
| Condition | 10,000 | ~40 MB |
| QuestionnaireResponse | 5,000 | ~100 MB |

> **Constraint:** Maximum memory per ETL run must not exceed 256 MB. This ensures reliable operation on Raspberry Pi 5 (8 GB) alongside the FHIR server and other services.

---

## 5. Scheduled sync

### 5.1 Sync mechanisms

| Plane | Sync method | Trigger | Direction |
|---|---|---|---|
| Plane 0 | None (air-gapped) | N/A | N/A |
| Plane 1 | USB / manual file copy | Manual | Export → USB drive |
| Plane 2 | Periodic HTTPS sync | systemd timer | Bidirectional |
| Plane 3 | Continuous sync | systemd timer + NATS event | Bidirectional |
| Plane 4 | Real-time sync | WebSocket + NATS + timer | Global |

### 5.2 systemd timer configuration

```ini
# /etc/systemd/system/zs-daily-etl.timer
[Unit]
Description=ZarishSphere Daily ETL Pipeline
Requires=zs-fhir-server.service

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=1800

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/zs-daily-etl.service
[Unit]
Description=ZarishSphere Daily ETL Pipeline
After=zs-fhir-server.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/zs-etl run --config=/etc/zarishsphere/pipeline/daily.yml
User=zarishsphere
Group=zarishsphere
MemoryMax=256M
```

### 5.3 Sync conflict resolution

| Scenario | Resolution strategy |
|---|---|
| Same record, different data | Last-write-wins with audit trail |
| Record deleted on one side | Tombstone propagation |
| Concurrent patient registration | UUID-based, no collision |
| Offline period > 30 days | Full sync instead of incremental |

---

## 6. Export formats

### 6.1 Supported formats

| Format | Use case | Implementation |
|---|---|---|
| CSV | Spreadsheet import, reporting | Go encoding/csv |
| JSON | API consumption, backup | Go encoding/json |
| FHIR Bundle | Healthcare interoperability | FHIR Bundle builder |
| Parquet | Analytics, large datasets | Apache Arrow Go |
| HTML | Embedded dashboard | Go html/template |

### 6.2 Export engine

The export engine is a separate Go binary (`zs-export`) or subcommand of `zs-etl`:

```bash
# Export all patients as CSV
zs-export --resource=Patient --format=csv --output=patients.csv

# Export observations from last 7 days as Parquet
zs-export --resource=Observation --format=parquet \
  --filter="last_updated >= datetime('now', '-7 days')" \
  --output=observations.parquet

# Export complete database as FHIR Bundle
zs-export --format=fhir-bundle --output=backup.json

# Generate HTML dashboard
zs-export --format=dashboard --template=ncd-report.tmpl --output=report.html
```

### 6.3 Parquet schema for analytics

```go
// Auto-generated Parquet schema from aggregated query results
type NCDIndicatorRow struct {
    Date            string  `parquet:"name=date, type=BYTE_ARRAY, converted=UTF8"`
    District        string  `parquet:"name=district, type=BYTE_ARRAY, converted=UTF8"`
    UniquePatients  int64   `parquet:"name=unique_patients, type=INT64"`
    Hypertension    int64   `parquet:"name=hypertension, type=INT64"`
    Diabetes        int64   `parquet:"name=diabetes, type=INT64"`
}
```

### 6.4 Export size estimates

| Export scope | CSV size | Parquet size | JSON size |
|---|---|---|---|
| 10,000 patients | ~2 MB | ~800 KB | ~15 MB |
| 100,000 observations | ~8 MB | ~3 MB | ~60 MB |
| Full clinic (1 year) | ~50 MB | ~15 MB | ~350 MB |
| District (50 clinics) | ~2.5 GB | ~750 MB | ~17.5 GB |

---

## 7. Reporting

### 7.1 Embedded dashboards

Dashboards are server-side rendered HTML files generated by Go's `html/template` package. No JavaScript framework is needed for the report layer.

```go
// Dashboard template rendering
func RenderDashboard(w io.Writer, data *DashboardData) error {
    tmpl := template.Must(template.ParseFiles("templates/ncd-dashboard.tmpl"))
    return tmpl.Execute(w, data)
}
```

### 7.2 Dashboard types

| Dashboard | Frequency | Data source | Audience |
|---|---|---|---|
| NCD clinic summary | Daily | FHIR analytics tables | Clinic staff |
| Module usage | Weekly | Module logs | Admin |
| Form completion | Weekly | QuestionnaireResponse | QA team |
| Population health | Monthly | Aggregated indicators | District health |
| Export status | On demand | ETL run history | Admin |

### 7.3 Dashboard template example

```html
<!-- templates/ncd-dashboard.tmpl -->
<!DOCTYPE html>
<html>
<head>
    <title>{{.Title}} - ZarishSphere</title>
    <style>
        body { font-family: system-ui, sans-serif; margin: 2rem; }
        .card { border: 1px solid #ddd; border-radius: 8px; padding: 1rem; margin: 1rem 0; }
        .stat { display: inline-block; margin: 1rem; text-align: center; }
        .stat-value { font-size: 2rem; font-weight: bold; }
        .stat-label { font-size: 0.875rem; color: #666; }
        table { width: 100%; border-collapse: collapse; }
        th, td { text-align: left; padding: 0.5rem; border-bottom: 1px solid #ddd; }
    </style>
</head>
<body>
    <h1>{{.Title}}</h1>
    <p>Generated: {{.GeneratedAt}}</p>
    <p>Period: {{.PeriodStart}} — {{.PeriodEnd}}</p>
    
    <div class="card">
        <h2>Key indicators</h2>
        {{range .Stats}}
        <div class="stat">
            <div class="stat-value">{{.Value}}</div>
            <div class="stat-label">{{.Label}}</div>
        </div>
        {{end}}
    </div>
    
    <div class="card">
        <h2>Monthly trend</h2>
        <table>
            <tr><th>Month</th><th>Patients</th><th>HTN</th><th>DM</th></tr>
            {{range .MonthlyTrend}}
            <tr>
                <td>{{.Month}}</td>
                <td>{{.Patients}}</td>
                <td>{{.Hypertension}}</td>
                <td>{{.Diabetes}}</td>
            </tr>
            {{end}}
        </table>
    </div>
</body>
</html>
```

### 7.4 Viewing dashboards

- Dashboards are static HTML files served by the FHIR server's file server
- Accessible at `http://[host]:[port]/dashboards/`
- No authentication for Plane 0 (localhost)
- JWT auth for Plane 1+
- Dashboards can be printed or exported to PDF via browser

---

## 8. Data retention and archiving

### 8.1 Retention policy

| Data category | Active retention | Archive after | Deletion |
|---|---|---|---|
| FHIR Patient records | Indefinite | N/A | Never (constitutional right) |
| Clinical observations | 10 years | 10 years | Never |
| Encounter records | 10 years | 10 years | Never |
| Form submissions | 10 years | 10 years | Never (audit requirement) |
| Audit logs | 7 years | 7 years | 10 years |
| App usage logs | 1 year | 1 year | 3 years |
| ETL run history | 1 year | 1 year | 3 years |
| Analytics aggregates | 15 years | N/A | Never |

> **Constraint:** Patient data must never be deleted. Law 7 (data sovereignty) guarantees that deployers own their data and no automatic deletion applies to clinical records.

### 8.2 Archive process

```
1. Mark records for archival based on retention policy
2. Export to Parquet + FHIR Bundle format
3. Compress with gzip
4. Store in Cloudflare R2 (Plane 3+) or local filesystem (Plane 0-2)
5. Remove from active database if needed (metadata marker retained)
6. Record archive manifest in archive_log table
```

### 8.3 Archive restore

```bash
# List available archives
zs-etl archive list

# Restore archive to working database
zs-etl archive restore --archive=2025-observations.parquet

# Verify archive integrity
zs-etl archive verify --archive=2025-observations.parquet
```

---

## 9. Cross-references

→ **007-zs-tech-stack/001-tech-stack-master.md** — Master production tech stack mapping
→ **007-zs-tech-stack/006-oss-tool-catalog.md** — Authoritative, version-pinned OSS tool catalog
→ **007-zs-tech-stack/002-go-fhir-server.md** — FHIR server specification (primary data source)
→ **003-zs-platform/007-data-model.md** — Platform data model and storage conventions
→ **008-zs-adrs/014-adr-nats-jetstream-messaging.md** — ADR-014: NATS JetStream messaging backbone
→ **008-zs-adrs/013-adr-postgresql-primary-database.md** — ADR-013: PostgreSQL 18.4 primary database
→ **008-zs-adrs/006-adr-zero-cost-toolchain.md** — ADR-006: zero-cost toolchain
→ **006-zs-infrastructure/003-cloudflare-architecture.md** — Cloudflare R2 object storage for archives
→ **009-zs-operations/010-sop-incident-response.md** — Incident response SOP (pipeline failure handling)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
