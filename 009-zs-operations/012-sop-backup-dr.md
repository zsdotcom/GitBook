# 012-sop-backup-dr.md
## SOP-012: Database backup verification, disaster recovery, and restore procedures
### Standard Operating Procedure — backup and disaster recovery

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

To define the complete backup and disaster recovery procedure for the ZarishSphere Platform — ensuring that all production data (PostgreSQL 18.4 with TimescaleDB) is backed up, verifiable, and recoverable within the defined Recovery Time Objective (RTO) of 4 hours and Recovery Point Objective (RPO) of 1 hour.

---

## 2. Scope

**In scope:** PostgreSQL 18.4 database backups (all schemas: `fhir`, `clinical`, `audit`, `terminology`, `analytics`). WAL streaming to standby. Daily full backups to Cloudflare R2 (AES-256 encrypted). Backup verification procedures (weekly and monthly). Three disaster recovery scenarios: accidental data deletion, complete Kubernetes cluster failure, and backup corruption. Full step-by-step database restore procedure. Post-recovery verification checklist.

**Out of scope:** Application-level backups (covered by GitOps — Argo CD re-deploys from Git). Keycloak session data (not needed for recovery). Monitoring and alerting configuration (see → **[015-sop-monitoring-dashboards.md](009-zs-operations/015-sop-monitoring-dashboards.md)**). Infrastructure provisioning (see → **[001-infrastructure-overview.md](006-zs-infrastructure/001-infrastructure-overview.md)**).

---

## 3. Roles

| Role | Who |
|---|---|
| **Backup Operator** | Person verifying backup status (default: any maintainer) |
| **DR Lead** | Person executing disaster recovery (default: `@devopsariful`) |
| **Technical Lead** | Person providing technical oversight (default: `@codeandbrain`) |
| **Verifier** | Person running post-recovery smoke tests |
| **Monthly Test Lead** | Person performing monthly restore test (default: `@devopsariful` on first Monday) |

---

## 4. Preconditions

- PostgreSQL 18.4 is running with WAL archiving enabled (see → **[013-adr-postgresql-primary-database.md](008-zs-adrs/013-adr-postgresql-primary-database.md)**).
- Cloudflare R2 bucket `zarishsphere-backups-{cc}` exists with AES-256 encryption.
- Vault contains the backup encryption key at `secret/zarishsphere/prod/backup-encryption-key`.
- `rclone` is configured with Cloudflare R2 access.
- `kubectl` context points to the production cluster.
- A standby PostgreSQL instance is configured for WAL replay.

---

## 5. Steps

### 5.1 Backup architecture

The ZarishSphere backup system uses a layered architecture:

```
PostgreSQL 18.4 (primary)
    │
    ├── WAL streaming → PostgreSQL Standby (hot standby, same region)
    │
    ├── WAL archiving (continuous) → Cloudflare R2: zarishsphere-backups/wal/
    │
    └── pg_dump (daily 02:00 UTC) → Cloudflare R2: zarishsphere-backups/daily/
                                         ↓
                                  AES-256 encryption (key in Vault)
                                         ↓
                                  7-year retention (HIPAA requirement)
```

**Backup contents include:**
- All schemas: `fhir`, `clinical`, `audit`, `terminology`, `analytics`
- All TimescaleDB hypertables
- Sequence states
- Extensions and configuration

**Excluded from backup:**
- Session data (Keycloak sessions — not needed for recovery)
- Temporary tables
- `pg_toast` data

### 5.2 Verify automated backups are running

**Daily check via Kubernetes:**

1. Open a terminal.
2. Check the backup CronJob status:
   ```bash
   kubectl get cronjob -n zs-data | grep backup
   ```
   Expected output: `zs-backup-postgres   0/6h   ACTIVE`

3. Check the last 5 backup jobs:
   ```bash
   kubectl get jobs -n zs-data | grep backup | tail -5
   ```
   All should show `Complete`.

**Daily check via Cloudflare dashboard:**

1. Open a browser and navigate to **Cloudflare Dashboard** → **R2**.
2. Click the `zarishsphere-backups-{cc}` bucket.
3. Sort files by date.
4. Verify the most recent file is less than 6 hours old.

**Checklist item:** ☐ Backup CronJob is active and last 5 jobs completed successfully.

**Checklist item:** ☐ R2 bucket contains a backup file less than 6 hours old.

### 5.3 Trigger a manual backup

Use this when an immediate backup is needed before a risky operation:

1. Open a terminal.
2. Trigger an immediate backup:
   ```bash
   kubectl create job --from=cronjob/zs-backup-postgres \
     manual-backup-$(date +%Y%m%d%H%M) -n zs-data
   ```
3. Watch the backup complete:
   ```bash
   kubectl logs -n zs-data job/manual-backup-{timestamp} -f
   ```
4. Verify the backup appears in R2 (follow §5.2 cloudflare check).

**Checklist item:** ☐ Manual backup completed and verified in R2.

### 5.4 Monthly backup verification (restore test)

**Schedule:** First Monday of each month, performed by `@devopsariful`.

1. Download the most recent backup from R2:
   ```bash
   rclone copy r2:zarishsphere-backups-{cc}/zarishsphere-YYYY-MM-DD.pgc.enc /tmp/restore-test/
   ```

2. Decrypt the backup:
   ```bash
   vault kv get -field=key secret/zarishsphere/prod/backup-encryption-key > /tmp/backup.key
   openssl enc -d -aes-256-gcm -in /tmp/restore-test/zarishsphere-YYYY-MM-DD.pgc.enc \
     -out /tmp/restore-test/zarishsphere.pgc \
     -pass file:/tmp/backup.key
   ```

3. Restore to a fresh PostgreSQL instance (dev environment):
   ```bash
   docker run -d --name pg-restore-test \
     -e POSTGRES_PASSWORD=restore \
     timescale/timescaledb:2.25-pg18
   pg_restore -h localhost -U postgres -d zarishsphere /tmp/restore-test/zarishsphere.pgc
   ```

4. Run validation queries:
   ```sql
   SELECT COUNT(*) FROM fhir.resources;
   SELECT MAX(updated_at) FROM fhir.resources;
   SELECT COUNT(DISTINCT tenant_id) FROM fhir.resources;
   ```

5. Record results in the execution log.

**Checklist item:** ☐ Monthly restore test completed and results recorded.

### 5.5 Recovery objectives

| Metric | Target | Notes |
|---|---|---|
| **RTO** (Recovery Time Objective) | 4 hours | Time from disaster to service restoration |
| **RPO** (Recovery Point Objective) | 1 hour | Maximum acceptable data loss window |
| Backup frequency | Hourly (WAL) + Daily (full dump) | |
| Backup retention | 30 days daily, 12 months monthly | |
| Backup location | Cloudflare R2 (encrypted AES-256) | |

### 5.6 Disaster recovery scenarios

#### Scenario 1: Accidental data deletion

1. **Stop writes** to the affected database immediately:
   ```bash
   kubectl scale deployment zs-fhir-engine --replicas=0 -n zs-core
   ```
2. **Identify the deletion time** from audit logs:
   ```sql
   SELECT * FROM audit.events
   WHERE event_type = 'delete'
   ORDER BY recorded_at DESC LIMIT 100;
   ```
3. **Download the last backup before deletion** from R2:
   ```bash
   rclone copy r2:zarishsphere-backups/daily/{DATE}/ /tmp/restore/
   ```
4. **Restore to a test PostgreSQL instance first** (see §5.7 steps 1–4).
5. **Extract and re-import** deleted records.
6. **Verify data integrity** — run validation queries.
7. **Restart services:**
   ```bash
   kubectl scale deployment zs-fhir-engine --replicas=2 -n zs-core
   ```

#### Scenario 2: Complete Kubernetes cluster failure

1. **Provision new cluster** from Infrastructure as Code:
   ```bash
   cd ~/zarishsphere/zs-infra-bgd
   tofu init && tofu apply
   ```
   See → **[016-adr-opentofu-infrastructure-as-code.md](008-zs-adrs/016-adr-opentofu-infrastructure-as-code.md)**.

2. **Install Argo CD** on the new cluster:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f \
     https://raw.githubusercontent.com/argoproj/argo-cd/v3.4.6/manifests/install.yaml
   ```

3. **Argo CD auto-syncs** all applications from the GitOps repository.

4. **Restore PostgreSQL** from the R2 backup (see §5.7).

5. **Verify all services healthy** through smoke tests.

#### Scenario 3: Cloudflare R2 backup corruption

1. **Check backup file integrity** using SHA-256 checksums:
   ```bash
   sha256sum -c zarishsphere-YYYY-MM-DD.pgc.sha256
   ```
2. If the current backup is corrupted, **use the previous day's backup**.
3. Accept RPO = up to 24 hours of data loss.
4. Initiate a P0 incident and notify all owners.
5. Investigate root cause of corruption.

### 5.7 Full database restore procedure

**Warning:** This procedure causes downtime. Notify all active users first.

#### Step 0: Try WAL replay first (less downtime)

If the database is still running and only the last few hours are affected:

1. Check WAL standby lag:
   ```bash
   kubectl exec -n zs-data postgres-standby-0 -- \
     psql -U zarish -c "SELECT now() - pg_last_xact_replay_timestamp() AS replication_delay;"
   ```
2. If delay < 6 hours, **promote standby** instead of restoring from backup:
   ```bash
   kubectl exec -n zs-data postgres-standby-0 -- \
     pg_ctl promote -D /var/lib/postgresql/data
   ```
3. If WAL replay works: skip steps 1–5, go directly to step 6.

**Checklist item:** ☐ WAL replay attempted and either succeeded or ruled out.

#### Step 1: Take services offline

```bash
# Scale down all services that write to PostgreSQL
kubectl scale deployment -n zs-core --replicas=0 --all
kubectl scale deployment -n zs-clinical --replicas=0 --all

# Verify all pods are down
kubectl get pods -n zs-core
kubectl get pods -n zs-clinical
```

**Checklist item:** ☐ All write services are scaled to 0.

#### Step 2: Download backup from R2

```bash
# List available backups
rclone ls r2:zarishsphere-backups-{cc}/ | sort -k2 | tail -20

# Download the most recent full backup
rclone copy r2:zarishsphere-backups-{cc}/{backup-filename}.sql.enc /tmp/recovery/

# Decrypt the backup
vault kv get -field=key secret/zarishsphere/prod/backup-encryption-key > /tmp/backup.key
openssl enc -d -aes-256-gcm -in /tmp/recovery/{backup}.sql.enc \
  -out /tmp/recovery/{backup}.sql \
  -pass file:/tmp/backup.key
```

**Checklist item:** ☐ Backup downloaded and decrypted successfully.

#### Step 3: Drop and recreate database

```bash
kubectl exec -n zs-data postgres-0 -- \
  psql -U zarish postgres -c "
    SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'zarishsphere';
    DROP DATABASE IF EXISTS zarishsphere;
    CREATE DATABASE zarishsphere OWNER zarish;
  "
```

#### Step 4: Restore

```bash
kubectl cp /tmp/recovery/{backup}.sql zs-data/postgres-0:/tmp/restore.sql
kubectl exec -n zs-data postgres-0 -- \
  psql -U zarish zarishsphere < /tmp/restore.sql

# Verify row counts
kubectl exec -n zs-data postgres-0 -- \
  psql -U zarish zarishsphere -c "SELECT COUNT(*) FROM fhir.resources;"
```

#### Step 5: Run pending migrations

```bash
kubectl run migrate-job --image=ghcr.io/zarishsphere/zs-core-fhir-engine:v1.0.0 \
  --env="DATABASE_URL=postgres://zarish:{password}@postgres:5432/zarishsphere" \
  --command -- migrate -path /migrations -database ${DATABASE_URL} up
```

#### Step 6: Bring services back online

```bash
kubectl scale deployment -n zs-core --replicas=2 --all
kubectl scale deployment -n zs-clinical --replicas=2 --all

# Run smoke tests
curl -s https://api.zarishsphere.com/fhir/R5/metadata | jq .resourceType
# Expected: "CapabilityStatement"
```

**Checklist item:** ☐ Services restored and smoke tests pass.

#### Step 7: Communicate resolution

1. Notify all facility coordinators: "ZarishSphere is restored and operational."
2. Specify any data loss window (time between last backup and incident).
3. Open a GitHub Issue for the postmortem.

### 5.8 Backup verification schedule

| Frequency | Test Type | Performed By |
|---|---|---|
| Weekly | Restore to staging and verify row counts | `@devopsariful` |
| Monthly | Full DR simulation (restore to test cluster) | `@codeandbrain` + `@devopsariful` |
| Quarterly | Table-top DR exercise with all owners | All owners |

### 5.9 Post-recovery checklist

After any recovery operation, verify:

- [ ] All services returning HTTP 200 on health checks.
- [ ] FHIR CapabilityStatement endpoint responding at `/fhir/R5/metadata`.
- [ ] AuditEvent logging active (spot-check one event).
- [ ] Keycloak authentication working (test login with a test user).
- [ ] Grafana dashboards showing current data.
- [ ] Incident report opened and timeline documented.
- [ ] Postmortem scheduled within 48 hours.

---

## 6. Expected outcome

- Automated backups run every 6 hours and all jobs complete successfully.
- Backup files are present in Cloudflare R2 with files less than 6 hours old.
- Monthly restore tests confirm that backups are valid and restorable.
- Recovery from any of the three scenarios completes within the 4-hour RTO.
- Data loss does not exceed the 1-hour RPO.
- Post-recovery verification checklist confirms all services are operational.
- A postmortem is completed within 7 days of any disaster recovery event.

---

## 7. Escalation

| Issue | Action |
|---|---|
| Backup CronJob fails for 2 consecutive runs | Investigate: `kubectl logs -n zs-data job/{failed-job-name}`. Common causes: R2 bucket full, encryption key rotated, PostgreSQL connection refused. |
| R2 bucket inaccessible | Check Cloudflare Dashboard for service status. Contact Cloudflare Support. Use secondary backup location if configured. |
| Restore fails — backup file corrupted | Try the previous day's backup. If that also fails, use the standby PostgreSQL instance (promote to primary). |
| Restore succeeds but data is inconsistent | Run full data validation queries. Compare row counts against last known-good snapshot. If inconsistencies found, restore an older backup. |
| WAL replay fails | The standby instance may be too far behind. Fall back to full restore from the daily backup (see §5.7). |
| Encryption key missing from Vault | Check Vault status: `vault status`. If Vault is down, use the break-glass physical key stored in the Foundation safe (per SOP-006). |
| DR test reveals RPO > 1 hour | Increase backup frequency and verify WAL archiving is continuous. Consider reducing daily full backup interval. |
| DR test reveals RTO > 4 hours | Practice the restore procedure. Pre-stage restore scripts. Document timing of each step to identify bottlenecks. |

---

## Cross-references

- → **[009-sop-deployment.md](009-zs-operations/009-sop-deployment.md)** — SOP-009: Deployment procedures (scale down/up commands)
- → **[010-sop-incident-response.md](009-zs-operations/010-sop-incident-response.md)** — SOP-010: Incident response for DR events
- → **[006-sop-credential-succession.md](009-zs-operations/006-sop-credential-succession.md)** — SOP-006: Credential rotation (encryption key)
- → **[013-adr-postgresql-primary-database.md](008-zs-adrs/013-adr-postgresql-primary-database.md)** — ADR-013: PostgreSQL database
- → **[014-adr-nats-jetstream-messaging.md](008-zs-adrs/014-adr-nats-jetstream-messaging.md)** — ADR-014: NATS JetStream messaging
- → **[015-adr-valkey-for-caching.md](008-zs-adrs/015-adr-valkey-for-caching.md)** — ADR-015: Valkey caching
- → **[016-adr-opentofu-infrastructure-as-code.md](008-zs-adrs/016-adr-opentofu-infrastructure-as-code.md)** — ADR-016: OpenTofu for cluster provisioning
- → **[017-adr-argocd-gitops.md](008-zs-adrs/017-adr-argocd-gitops.md)** — ADR-017: Argo CD for GitOps
- → **[001-infrastructure-overview.md](006-zs-infrastructure/001-infrastructure-overview.md)** — Infrastructure overview
- → **[003-cloudflare-architecture.md](006-zs-infrastructure/003-cloudflare-architecture.md)** — Cloudflare R2 configuration

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
