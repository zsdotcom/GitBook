# 015-sop-monitoring-dashboards.md
## SOP-015: Monitoring dashboards, alerting, and observability
### Standard Operating Procedure — Grafana dashboard management

**Document type:** Standard Operating Procedure
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Scope](#2-scope)
3. [Roles](#3-roles)
4. [Preconditions](#4-preconditions)
5. [Steps](#5-steps)
6. [Expected outcome](#6-expected-outcome)
7. [Escalation](#7-escalation)

---

## 1. Purpose

To define how operators and maintainers use Grafana 13.1.1 dashboards to monitor the ZarishSphere Platform — including daily health checks, key metric thresholds (service health, FHIR engine performance, database health, message queues, and caching), alert interpretation, dashboard provisioning via GitOps, and the procedure for creating new dashboards.

---

## 2. Scope

**In scope:** All 9 standard ZarishSphere Grafana dashboards: Platform Overview, FHIR Engine, Clinical Services, Database (PostgreSQL 18.4), NATS JetStream, Valkey, Keycloak, Kubernetes, and Country Health. Daily health check procedures. Alert conditions and response actions. Dashboard creation and provisioning via `zs-iac-observability` repository. Metric thresholds for healthy, warning, and critical states.

**Out of scope:** Infrastructure monitoring for Cloudflare or GitHub (handled by their respective dashboards). Application-level logging (ELK/Loki). Incident response triggered by alerts (see → **[010-sop-incident-response.md](009-zs-operations/010-sop-incident-response.md)**). Grafana server administration.

---

## 3. Roles

| Role | Who |
|---|---|
| **Daily Monitor** | Any maintainer performing the daily health check |
| **Alert Responder** | On-call engineer responding to Grafana alerts |
| **Dashboard Developer** | Person designing and provisioning new dashboards |
| **Reviewer** | Maintainer reviewing dashboard PRs before merge |

---

## 4. Preconditions

- Grafana 13.1.1 is deployed at `https://grafana.zarishsphere.com` (or `http://localhost:3000` locally).
- The operator has login credentials for Grafana.
- Prometheus (or the relevant data source) is configured and sending metrics.
- Dashboards are provisioned from the `zs-iac-observability` GitOps repository.
- The operator can access the Kubernetes cluster for `kubectl` commands if needed.

---

## 5. Steps

### 5.1 Dashboard inventory

Access all dashboards at `https://grafana.zarishsphere.com`. Nine standard dashboards are provisioned:

| Dashboard | URL Path | Purpose | Primary Audience |
|---|---|---|---|
| Platform Overview | `/d/platform-overview` | All services health at a glance | All owners |
| FHIR Engine | `/d/fhir-engine` | Request rates, latencies, error rates | Technical |
| Clinical Services | `/d/clinical-services` | Patient, encounter, observation services | Clinical leads |
| Database | `/d/postgresql` | PostgreSQL 18.4 query performance | DevOps |
| NATS JetStream | `/d/nats` | Message throughput, consumer lag | DevOps |
| Valkey | `/d/valkey` | Cache hit rate, memory usage | DevOps |
| Keycloak | `/d/keycloak` | Authentication rates, failed logins | Security |
| Kubernetes | `/d/kubernetes` | Pod health, resource usage | DevOps |
| Country Health | `/d/country-{cc}` | per-country program indicators | Program managers |

### 5.2 Daily health check procedure

Perform these checks at least once per day (every morning). Total time: approximately 5 minutes.

#### 5.2.1 Platform Overview dashboard

1. Open a browser and navigate to `https://grafana.zarishsphere.com`.
2. Log in with your credentials.
3. Navigate to **Platform Overview** (from the dashboard list or search).
4. Check all service panels:
   - **Green** panels = healthy — no action needed.
   - **Yellow** panels = degraded — plan intervention.
   - **Red** panels = investigate immediately → proceed to §5.2.2.

**Checklist item:** ☐ Platform Overview shows all green panels.

#### 5.2.2 Investigate red/yellow panels

If any panel shows red or yellow:

1. Click the affected panel to see details.
2. Note the service name and metric value.
3. Check the **FHIR Engine** dashboard for more detail.
4. Check the **Database** dashboard for PostgreSQL health.
5. If error rate > 1% or service is down → declare an incident per → **[010-sop-incident-response.md](009-zs-operations/010-sop-incident-response.md)**.

### 5.3 Key metrics and thresholds

#### 5.3.1 Service health (Platform Overview)

| Indicator | Interpretation |
|---|---|
| **Green** | Service healthy, no issues |
| **Yellow** | Degraded — increased latency or error rate, plan intervention |
| **Red** | Critical — service down or error rate > 1%, investigate immediately |

#### 5.3.2 FHIR Engine performance

Open the **FHIR Engine** dashboard at `/d/fhir-engine`.

| Metric | Healthy | Warning | Critical |
|---|---|---|---|
| P95 response time | < 200 ms | 200–500 ms | > 500 ms |
| Error rate | < 0.1% | 0.1–1% | > 1% |
| Request rate | Any non-zero | — | 0 (no traffic) |

**Checklist item:** ☐ FHIR Engine P95 response time < 200 ms.

**Checklist item:** ☐ FHIR Engine error rate < 0.1%.

#### 5.3.3 Database (PostgreSQL 18.4)

Open the **Database** dashboard at `/d/postgresql`.

| Metric | Healthy | Warning | Critical |
|---|---|---|---|
| Active connections | < 80 | 80–100 | > 100 |
| Replication lag | < 100 ms | 100 ms–1 s | > 1 s |
| Disk usage | < 70% | 70–85% | > 85% |

**Checklist item:** ☐ Database active connections < 80.

**Checklist item:** ☐ Database replication lag < 100 ms.

**Checklist item:** ☐ Database disk usage < 70%.

#### 5.3.4 NATS JetStream

Open the **NATS JetStream** dashboard at `/d/nats`.

| Metric | Healthy | Warning | Critical |
|---|---|---|---|
| Consumer lag | < 1,000 | 1,000–10,000 | > 10,000 |
| Message throughput | Stable | Fluctuating | Zero |

#### 5.3.5 Valkey cache

Open the **Valkey** dashboard at `/d/valkey`.

| Metric | Healthy | Warning | Critical |
|---|---|---|---|
| Cache hit rate | > 80% | 60–80% | < 60% |
| Memory usage | < 70% | 70–85% | > 85% |

#### 5.3.6 Keycloak authentication

Open the **Keycloak** dashboard at `/d/keycloak`.

| Metric | Healthy | Warning | Critical |
|---|---|---|---|
| Failed logins (5 min) | < 10 | 10–50 | > 50 |
| Active sessions | Stable | — | Sudden drop |

### 5.4 Alert configuration and response

Alerts are configured in the `zs-iac-observability` repository. The following alerts are active in production:

| Alert Name | Condition | Duration | Action |
|---|---|---|---|
| `FHIREngineDown` | FHIR engine returns non-200 | 2 minutes | Page `@devopsariful` |
| `DatabaseConnectionPoolExhausted` | Active connections > 95 | 30 seconds | Page `@devopsariful` |
| `DiskUsageCritical` | Disk usage > 85% | 5 minutes | Email all owners |
| `NATSConsumerLag` | Consumer lag > 10,000 | 5 minutes | Email `@codeandbrain` |
| `HighErrorRate` | Error rate > 1% | 5 minutes | Email all owners |

#### Responding to an alert:

1. Acknowledge the alert in Grafana:
   - Open the alert notification.
   - Click **"Silence"** or **"Acknowledge"** (if appropriate).
2. Investigate the affected dashboard.
3. If the alert indicates a P0 or P1 incident → follow → **[010-sop-incident-response.md](009-zs-operations/010-sop-incident-response.md)**.
4. If the issue resolves automatically → document in the daily log.
5. If the alert is a false positive → check alert thresholds and consider adjusting in `zs-iac-observability`.

**Checklist item:** ☐ All active alerts acknowledged and investigated.

### 5.5 Adding a new dashboard

**All dashboards must be provisioned via GitOps — do not save dashboards only in the Grafana UI, as they will be lost on pod restart.**

1. Design the dashboard in the **Grafana UI**:
   - Open Grafana at `https://grafana.zarishsphere.com`.
   - Click **"+"** → **"New Dashboard"**.
   - Add panels, configure queries, and arrange the layout.
2. Export the dashboard as JSON:
   - Click the **"Share"** icon (top-right of the dashboard).
   - Select the **"Export"** tab.
   - Click **"Save to file"** — this downloads a JSON file.
3. Save the JSON to the provisioning repository:
   - Open a browser and navigate to `https://github.com/zsdotcom/zs-iac-observability`.
   - Navigate to `dashboards/` directory.
   - Click **"Add file"** → **"Upload files"**.
   - Upload the JSON file with the name `{dashboard-name}.json`.
4. Create a PR:
   - Enter a commit message: `monitoring(dashboards): add {Dashboard Name} dashboard`.
   - Create a new branch and open a pull request.
   - Assign the PR to a reviewer.
5. After the PR merges, Grafana provisions the dashboard within 5 minutes.
6. Verify the new dashboard appears in Grafana:
   - Refresh the Grafana UI.
   - Search for the dashboard name in the search bar.

**Checklist item:** ☐ New dashboard created, JSON exported, and committed to `zs-iac-observability`.

**Checklist item:** ☐ Dashboard appears in Grafana after provisioning.

### 5.6 Grafana provisioning

Grafana dashboards are provisioned from the `zs-iac-observability` repository via GitOps.

**Directory structure in `zs-iac-observability`:**
```
zs-iac-observability/
├── dashboards/
│   ├── platform-overview.json
│   ├── fhir-engine.json
│   ├── postgresql.json
│   ├── nats.json
│   ├── valkey.json
│   ├── keycloak.json
│   ├── kubernetes.json
│   └── country-bgd.json
├── alerts/
│   ├── fhir-engine-alerts.yaml
│   ├── database-alerts.yaml
│   └── ...
└── provisioning.yaml
```

To update an existing dashboard:
1. Make changes in the Grafana UI (for visualization tuning).
2. Export the updated JSON (see §5.5 step 2).
3. Replace the existing JSON file in `zs-iac-observability/dashboards/`.
4. Create a PR and merge.

---

## 6. Expected outcome

- Daily health checks are completed in approximately 5 minutes each morning.
- All service panels show green on the Platform Overview dashboard.
- FHIR Engine P95 response time is below 200 ms and error rate is below 0.1%.
- Database active connections are below 80, replication lag is below 100 ms, and disk usage is below 70%.
- All active alerts are acknowledged, investigated, and resolved or documented.
- New dashboards are provisioned exclusively through GitOps (JSON export → commit to `zs-iac-observability` → PR → merge).
- No dashboards are saved only in the Grafana UI (they would be lost on pod restart).

---

## 7. Escalation

| Issue | Action |
|---|---|
| Grafana UI is unreachable (HTTP 502/503) | Check Grafana pod status: `kubectl get pods -n zs-monitoring | grep grafana`. Restart if needed: `kubectl rollout restart deployment/grafana -n zs-monitoring`. |
| Dashboard shows "No data" | Verify Prometheus or data source is running: `kubectl get pods -n zs-monitoring | grep prometheus`. Check data source configuration in Grafana → Configuration → Data Sources. |
| Alert firing but no one receiving notifications | Check alert notification channels in Grafana → Alerting → Contact points. Verify email/webhook configuration. Check `zs-iac-observability/alerts/` for correct routing. |
| New dashboard does not appear after provisioning | Verify the JSON file is in the correct directory (`dashboards/`). Check Grafana provisioning logs: `kubectl logs -n zs-monitoring deployment/grafana --tail=50`. |
| Metrics show unexpected values | Check if the service is running correctly. Verify the Prometheus exporter for the service is active. Check for recent deployments that may have changed metric names. |
| Country Health dashboard not showing data | Verify the country deployment is active and sending metrics. Check Prometheus targets for the country-specific exporter. |
| Grafana shows "Dashboard not found" after pod restart | The dashboard was likely saved only in the UI and not committed to `zs-iac-observability`. Re-create and export as JSON following §5.5. |
| Alert thresholds need adjustment | Modify the alert rule in `zs-iac-observability/alerts/`. Open a PR with the new thresholds. Document the reason for the threshold change. |

---

## Cross-references

- → **[010-sop-incident-response.md](009-zs-operations/010-sop-incident-response.md)** — SOP-010: Incident response when alerts trigger P0/P1
- → **[009-sop-deployment.md](009-zs-operations/009-sop-deployment.md)** — SOP-009: Deployment (monitor after deployment)
- → **[012-sop-backup-dr.md](009-zs-operations/012-sop-backup-dr.md)** — SOP-012: Backup monitoring (disk usage, backup jobs)
- → **[013-adr-postgresql-primary-database.md](008-zs-adrs/013-adr-postgresql-primary-database.md)** — ADR-013: PostgreSQL database
- → **[014-adr-nats-jetstream-messaging.md](008-zs-adrs/014-adr-nats-jetstream-messaging.md)** — ADR-014: NATS JetStream
- → **[015-adr-valkey-for-caching.md](008-zs-adrs/015-adr-valkey-for-caching.md)** — ADR-015: Valkey caching
- → **[006-ci-cd-architecture.md](006-zs-infrastructure/006-ci-cd-architecture.md)** — CI/CD architecture (observability pipeline)
- → **[007-security-architecture.md](006-zs-infrastructure/007-security-architecture.md)** — Security architecture (Keycloak monitoring)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
