# 010-sop-incident-response.md
## SOP-010: Incident response and P0 critical playbook
### Standard Operating Procedure — incident management

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

To define a standardised incident response procedure for the ZarishSphere Platform that ensures consistent severity classification, rapid detection and containment, systematic diagnosis, clear communication, and blameless postmortems. The primary goal is to minimise impact on patient care and data integrity.

---

## 2. Scope

**In scope:** All incidents affecting ZarishSphere Platform production services — including service outages, data corruption, performance degradation, authentication failures, and security breaches. P0 (Critical), P1 (High), P2 (Medium), and P3 (Low) severity incidents. The P0 critical playbook covering the first 15 minutes of response. Postmortem documentation within 7 days of resolution.

**Out of scope:** Security-specific incidents (see → **[011-sop-security-incident.md](009-zs-operations/011-sop-security-incident.md)**). Deployment rollbacks (see → **[009-sop-deployment.md](009-zs-operations/009-sop-deployment.md)**). Database recovery procedures (see → **[012-sop-backup-dr.md](009-zs-operations/012-sop-backup-dr.md)**). Monitoring and alerting configuration (see → **[015-sop-monitoring-dashboards.md](009-zs-operations/015-sop-monitoring-dashboards.md)**).

---

## 3. Roles

| Role | Who |
|---|---|
| **Incident Commander** | First responder who declares the incident and coordinates response |
| **Technical Lead** | Owner investigating root cause (default: `@codeandbrain`) |
| **Infrastructure Lead** | Owner managing infrastructure issues (default: `@devopsariful`) |
| **Communications Lead** | Owner managing stakeholder notifications (default: `@arwazarish`) |
| **Scribe** | Person documenting timeline and actions during the incident |
| **Postmortem Author** | Person responsible for writing the postmortem within 7 days |

---

## 4. Preconditions

- All responders have access to:
  - Grafana dashboard: `https://grafana.zarishsphere.com`
  - Argo CD UI: `https://argocd.zarishsphere.com`
  - Kubernetes cluster access via `kubectl`
  - GitHub repository access
  - Platform owner group (WhatsApp/Signal)
- Incident severity classification table is known (see §5.1).
- Escalation contact list is current.
- The postmortem template is available (see §5.7).

---

## 5. Steps

### 5.1 Classify the incident severity

Use this table to determine the severity level of any incident:

| Level | Name | Description | Response Time | Examples |
|---|---|---|---|---|
| **P0** | Critical | System completely down or PHI breach | Immediate — < 1 hour | Database down, security breach, complete outage |
| **P1** | High | Major feature broken, some users affected | < 4 hours | FHIR API returning 500, auth failing for some users |
| **P2** | Medium | Minor feature broken, workaround exists | < 24 hours | Slow search, PDF generation failing |
| **P3** | Low | Cosmetic issue, no functional impact | < 7 days | UI typo, wrong timezone in report |

**Checklist item:** ☐ Severity level determined and documented.

### 5.2 First steps for any incident

1. **Do not panic.** Read this SOP completely before acting.
2. **Determine severity** using the table in §5.1.
3. **Check if already known.** Look at GitHub Issues and active Grafana alerts at `https://grafana.zarishsphere.com`.
4. **Declare the incident** in the platform owner group (WhatsApp/Signal) with:
   ```
   INCIDENT DECLARED
   Severity: P{N}
   Description: {one line description}
   Impact: {who is affected}
   Declared by: @{your-name}
   ```

**Checklist item:** ☐ Incident declared in the platform owner group.

### 5.3 P0 critical playbook — the first 15 minutes

For all P0 incidents, follow these steps in strict order. Time target: resolved within 4 hours.

#### Minute 0–5: Detect and declare

1. **Alert received** — from Grafana alert, GitGuardian notification, or user report.
2. **Verify it is real** — check Grafana at `https://grafana.zarishsphere.com`:
   - Is error rate > 50%?
   - Is the FHIR API returning HTTP 500?
3. **Declare P0** — message all owners immediately:
   ```
   🚨 P0 INCIDENT DECLARED
   Time: {HH:MM UTC}
   Issue: {one line}
   Impact: {clinics/users affected}
   Commander: @{your name}
   Next update: {HH:MM UTC — 30 min from now}
   ```

#### Minute 5–10: Preserve evidence

**Before making ANY changes**, capture system state:

```bash
# Take a database snapshot immediately
kubectl exec -n zs-data postgres-0 -- \
  pg_dump -U zarish zarishsphere > /tmp/incident-snapshot-$(date +%Y%m%d-%H%M).sql

# Save current pod state
kubectl get pods --all-namespaces > /tmp/pod-state-$(date +%Y%m%d-%H%M).txt

# Save recent logs (last 1000 lines)
kubectl logs -n zs-core deployment/zs-core-fhir-engine --tail=1000 > /tmp/fhir-engine-logs.txt
```

#### Minute 10–15: Contain

**If database breach suspected:**
```bash
# Immediately block external access
kubectl patch ingressroute zs-api-gateway -n zs-gateway \
  --type=json -p '[{"op":"add","path":"/spec/routes/0/middlewares","value":[{"name":"block-all"}]}]'
```

**If deployment caused the issue:**
Follow → **[009-sop-deployment.md](009-zs-operations/009-sop-deployment.md)** §5.5 (Rollback) immediately.

**If infrastructure failure:**
Contact `@devopsariful` to switch to backup region.

### 5.4 Diagnosis checklist

Work through these checks in order:

```
□ Is the FHIR API responding?
  curl -s https://api.zarishsphere.com/health | jq .status

□ Is PostgreSQL 18.4 healthy?
  kubectl exec -n zs-data postgres-0 -- pg_isready

□ Is Keycloak 26.7.0 responding?
  curl -s https://auth.zarishsphere.com/health/ready

□ Are NATS 2.14.4 connections established?
  curl -s http://nats:8222/healthz

□ Is Valkey 9.1.1 responding?
  kubectl exec -n zs-data valkey-0 -- valkey-cli ping

□ What changed in the last 2 hours?
  Check GitHub: recent merges to main in affected repos

□ Are pods crashing?
  kubectl get pods --all-namespaces | grep -v Running

□ What do logs show?
  kubectl logs -n zs-core deployment/zs-core-fhir-engine --tail=100
```

**Checklist item:** ☐ Diagnosis checklist completed and findings documented.

### 5.5 Resolution by root cause

Once root cause is identified, take the appropriate resolution action:

| Root Cause | Resolution |
|---|---|
| Bad deployment | Follow → **[009-sop-deployment.md](009-zs-operations/009-sop-deployment.md)** §5.5 |
| Database out of disk | `kubectl exec postgres-0 -- vacuum`; expand PVC |
| Keycloak crashed | `kubectl rollout restart deployment/keycloak -n zs-auth` |
| NATS lost JetStream | `kubectl rollout restart deployment/nats -n zs-data` |
| Valkey OOM | `kubectl rollout restart deployment/valkey -n zs-data` |
| Secret expired | Rotate via Vault; restart affected deployments |
| PHI breach | Follow → **[011-sop-security-incident.md](009-zs-operations/011-sop-security-incident.md)** |

### 5.6 Communication during incident

| Action | Who | Frequency |
|---|---|---|
| Status update | Incident Commander | Every 30 min (P0), every 2h (P1) |
| Country program notification | `@arwazarish` or `@healthbgd` | On P0/P1 affecting their data |
| Resolution notification | Incident Commander | Once resolved |
| Postmortem scheduling | All owners | Within 7 days of resolution |

**Checklist item:** ☐ Communication log maintained with timestamps.

### 5.7 Postmortem template

Complete a postmortem within 7 days of incident resolution. Copy this template into a new GitHub Issue or document:

```markdown
# Incident Postmortem

## Incident Summary
| Field | Value |
|---|---|
| Date | YYYY-MM-DD |
| Duration | {X} hours {Y} minutes |
| Severity | P{N} |
| Incident Commander | @github-handle |
| Services Affected | zs-{service-name} |
| Users Affected | {number} facilities / {number} users |

## Impact
Describe what users experienced during the incident.

## Timeline
| Time (UTC) | Event |
|---|---|
| HH:MM | Alert received |
| HH:MM | Incident declared P{N} |
| HH:MM | Root cause identified |
| HH:MM | Fix deployed |
| HH:MM | Incident resolved |

## Root Cause
One clear sentence describing the root cause.
Then: explain WHY this happened (system reason, not human mistake).

## Contributing Factors
1. ...
2. ...

## What Went Well
1. ...
2. ...

## What We Could Do Better
1. ...
2. ...

## Action Items
| Action | Owner | Due Date | Status |
|---|---|---|---|
| {Specific preventive action} | @{owner} | YYYY-MM-DD | ⬜ Open |

## Lessons Learned
Two or three sentences on what the broader team should learn from this.
```

**Checklist item:** ☐ Postmortem completed and shared with all owners within 7 days.

---

## 6. Expected outcome

- All incidents are classified by severity within 2 minutes of detection.
- P0 incidents follow the structured 15-minute playbook (detect → preserve evidence → contain).
- The diagnosis checklist is systematically worked through before escalating to resolution.
- Communication updates are sent at the required frequency (every 30 min for P0).
- Resolution restores service within target time (4 hours for P0).
- A blameless postmortem is completed within 7 days, documenting root cause, contributing factors, and action items.

---

## 7. Escalation

| Issue | Action |
|---|---|
| Incident not resolved within 1 hour (P0) | Escalate to all 4 owners via phone call. Consider switching to backup region. |
| Root cause not identifiable within 2 hours | Engage wider team. Post full system state in owner group. Consider restoring from backup. |
| Multiple concurrent incidents | Declare separate incidents with different commanders. Prioritise P0 over all other work. |
| PHI breach detected | Immediately isolate affected systems. Follow → **[011-sop-security-incident.md](009-zs-operations/011-sop-security-incident.md)**. Do not notify users until scope is determined. |
| Incident commander unavailable | The first responder becomes commander. If no responder is available, the person who detects the incident assumes command. |
| Postmortem not completed within 7 days | The maintainer (`@arwazarish`) assigns a postmortem author and sets a 48-hour deadline. |
| Stakeholder not receiving updates | Use escalation contact list. Call or message the stakeholder directly. Document all contact attempts. |

---

## Cross-references

- → **[009-sop-deployment.md](009-zs-operations/009-sop-deployment.md)** — SOP-009: Deployment and rollback procedures
- → **[011-sop-security-incident.md](009-zs-operations/011-sop-security-incident.md)** — SOP-011: Security incident playbook
- → **[012-sop-backup-dr.md](009-zs-operations/012-sop-backup-dr.md)** — SOP-012: Backup and disaster recovery
- → **[015-sop-monitoring-dashboards.md](009-zs-operations/015-sop-monitoring-dashboards.md)** — SOP-015: Monitoring dashboards and alerting
- → **[013-adr-postgresql-primary-database.md](008-zs-adrs/013-adr-postgresql-primary-database.md)** — ADR-013: PostgreSQL database
- → **[014-adr-nats-jetstream-messaging.md](008-zs-adrs/014-adr-nats-jetstream-messaging.md)** — ADR-014: NATS JetStream messaging
- → **[015-adr-valkey-for-caching.md](008-zs-adrs/015-adr-valkey-for-caching.md)** — ADR-015: Valkey caching
- → **[006-ci-cd-architecture.md](006-zs-infrastructure/006-ci-cd-architecture.md)** — CI/CD architecture
- → **[007-security-architecture.md](006-zs-infrastructure/007-security-architecture.md)** — Security architecture

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
