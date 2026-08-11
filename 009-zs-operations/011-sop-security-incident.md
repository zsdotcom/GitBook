# 011-sop-security-incident.md
## SOP-011: Security incident response playbook
### Standard Operating Procedure — security incident management

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

To define the exact procedure for responding to security incidents in the ZarishSphere Platform — including suspected PHI breaches, credential compromise, active attacks, and data exfiltration. The primary objectives are: contain the threat, preserve forensic evidence, assess the scope, notify affected parties, and implement preventive measures.

---

## 2. Scope

**In scope:** All security incidents affecting ZarishSphere production services, including committed secrets, mass authentication failures, unexpected data exports, container CVEs, and active intrusions. First 30-minute response window. Scenario-specific playbooks for the three most common incident types. Regulatory notification templates (GDPR Art. 33 and Art. 34).

**Out of scope:** Non-security incidents (see → **[010-sop-incident-response.md](009-zs-operations/010-sop-incident-response.md)**). Credential rotation procedures (see → **[006-sop-credential-succession.md](009-zs-operations/006-sop-credential-succession.md)**). General infrastructure security policies (see → **[008-security-policies.md](006-zs-infrastructure/008-security-policies.md)**).

---

## 3. Roles

| Role | Who |
|---|---|
| **Security Incident Commander** | First responder — coordinates the security response |
| **Technical Investigator** | Engineer performing forensic analysis |
| **Communications Lead** | Owner managing breach notifications (default: `@arwazarish`) |
| **Legal Counsel** | External legal advisor for regulatory notifications |
| **Successor Designee** | Backup person for all credential rotation (per ADR-012) |

---

## 4. Preconditions

- All responders have access to:
  - Kubernetes cluster via `kubectl`
  - Cloudflare Dashboard at `https://dash.cloudflare.com`
  - GitHub organisation settings
  - Keycloak Admin UI at `https://auth.zarishsphere.com`
  - Vault CLI for secret management
- GitGuardian is configured on all repositories.
- Trivy is running on all container builds.
- The escalation contact list is current (see §7).

---

## 5. Steps

### 5.1 Recognise security incident triggers

Declare a security incident immediately if ANY of these occur:

- GitGuardian detected a committed secret in any repository.
- Unusual access patterns in `audit.events` — access by unknown IP or user.
- Keycloak logs show mass login failures (> 50 failures in 5 minutes).
- Database accessed outside normal hours by an admin account.
- Staff reports patient data shown to wrong person.
- Trivy found a CRITICAL CVE in a deployed container.

**Checklist item:** ☐ Security incident trigger identified and confirmed.

### 5.2 First 30 minutes — contain, preserve, assess

#### Minute 0–5: Contain the threat

**If a credential is compromised — revoke immediately:**

| Credential Type | Revocation Method |
|---|---|
| GitHub personal access token | `https://github.com/settings/tokens` → Revoke |
| Cloudflare API token | Cloudflare Dashboard → API Tokens → Revoke |
| PostgreSQL credential | `vault lease revoke {lease_id}` |
| Keycloak user session | Admin UI → Sessions → Revoke all sessions for affected user |

**If active intrusion is detected — block the source:**

1. Open **Cloudflare Dashboard** at `https://dash.cloudflare.com` in a browser.
2. Navigate to **Security** → **Firewall rules**.
3. Click **"Create firewall rule"**.
4. Set **Field** = `IP Source Address`, **Operator** = `equals`, **Value** = `{suspicious_ip}`.
5. Set **Action** = `Block`.
6. Click **"Save"**.

Alternatively, apply a Cilium network policy via `kubectl`:
```bash
kubectl apply -f - <<EOF
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: block-suspicious-ip
  namespace: zs-gateway
spec:
  egressDeny:
  - toCIDR:
    - {suspicious_ip}/32
EOF
```

#### Minute 5–15: Preserve forensic evidence

**Before making further changes**, capture the following:

1. **Keycloak access logs** (last 1000 lines):
   ```bash
   kubectl logs -n zs-auth deployment/keycloak --tail=1000 > /tmp/keycloak-logs.txt
   ```

2. **FHIR audit events** (last 2 hours):
   ```bash
   kubectl exec -n zs-data postgres-0 -- psql -U zarish zarishsphere -c \
     "SELECT * FROM audit.events WHERE recorded_at > NOW() - INTERVAL '2 hours' ORDER BY recorded_at;" \
     > /tmp/audit-events.txt
   ```

3. **Network connections** from the FHIR engine:
   ```bash
   kubectl exec -n zs-core deployment/zs-core-fhir-engine -- ss -tulnp > /tmp/connections.txt
   ```

**Checklist item:** ☐ Forensic evidence captured before any remediation.

#### Minute 15–30: Assess scope and notify

**Determine the scope of the incident:**

- Which tenant(s) are affected?
- Which resource types were accessed?
- Was data read only, or was data modified or deleted?
- Is the attacker still active?

**If PHI was accessed without authorisation:**
1. Notify `@arwazarish` IMMEDIATELY (call, not message).
2. Prepare for GDPR notification within 72 hours (Art. 33).
3. Do NOT notify users until full scope is determined.

**Checklist item:** ☐ Incident scope assessed and documented.

**Checklist item:** ☐ PHI breach notifications initiated if applicable.

### 5.3 Scenario: Committed secret

1. **Revoke the secret immediately:**
   - GitHub token: `https://github.com/settings/tokens` → Revoke.
   - API key: Invalidate in the issuing system.
   - Cloudflare token: Dashboard → API Tokens → Revoke.
2. **Generate a new secret** and update in Vault or GitHub organisation secrets:
   ```bash
   vault kv put secret/zarishsphere/prod/{secret-name} key={new-value}
   ```
3. **Remove the secret from git history:**
   ```bash
   git filter-repo --sensitive-data-removal --path {file-containing-secret}
   ```
4. **Notify all owners** in the platform owner group.
5. **Document** in the security incident log.

**Checklist item:** ☐ Secret revoked, rotated, and git history cleaned.

### 5.4 Scenario: Mass failed authentication

1. **Verify Keycloak brute force protection is active:**
   - Open **Keycloak Admin UI** at `https://auth.zarishsphere.com`.
   - Navigate to **Realm Settings** → **Security Defenses**.
   - Confirm **Brute Force Detection** is enabled.
2. **Block the attacking IP** in Cloudflare firewall:
   - Open **Cloudflare Dashboard** → **Security** → **Firewall rules**.
   - Create rule: Block `{attacking_ip}`.
3. **Review compromised accounts:**
   ```sql
   SELECT * FROM audit.events WHERE event_type = 'login_failure' ORDER BY recorded_at DESC LIMIT 100;
   ```
4. If any account was compromised → follow the committed secret procedure (§5.3).

**Checklist item:** ☐ Brute force protection verified and attacking IP blocked.

### 5.5 Scenario: Unexpected data export

1. **Review export events in the audit log:**
   ```sql
   SELECT * FROM audit.events WHERE event_type = 'export' ORDER BY recorded_at DESC;
   ```
2. **Identify the user** who initiated the export from the audit log.
3. **Suspend the user account** in Keycloak:
   - Open **Keycloak Admin UI** → **Users** → search for user → **Actions** → **Disable account**.
4. **Contact the user's organisation** for explanation.
5. If unauthorised: notify affected patients per GDPR Art. 34.

**Checklist item:** ☐ User account suspended and organisation contacted.

### 5.6 Regulatory notification templates

**Country focal point notification (P0 breach):**

> "ZarishSphere Security Notice: We have detected a potential security incident affecting {country} data. We have taken immediate containment measures. We will provide a full report within 24 hours. Please change your ZarishSphere password immediately at {url}."

**Regulatory authority notification (GDPR Art. 33, within 72 hours):**

Use the supervisory authority's official notification form. Include:
- Nature of the breach.
- Categories of data subjects affected.
- Approximate number of records affected.
- Likely consequences of the breach.
- Measures taken or proposed to address the breach.

---

## 6. Expected outcome

- Security incident is detected and classified within 2 minutes.
- Threat is contained within 5 minutes (credential revocation or IP blocking).
- Forensic evidence is preserved within 15 minutes.
- Incident scope is assessed within 30 minutes.
- Credential secrets are revoked, rotated, and removed from git history within 1 hour.
- Brute force attacks are blocked and compromised accounts are suspended.
- Regulatory notifications are prepared within 72 hours (GDPR Art. 33).
- A security incident report is filed in the incident log.

---

## 7. Escalation

| Issue | Action |
|---|---|
| PHI breach confirmed | Immediately notify `@arwazarish` via phone. Engage legal counsel. Prepare GDPR Art. 33 notification. Do not discuss scope publicly. |
| Attacker still active after containment | Isolate the entire affected namespace: `kubectl label ns {namespace} istio-injection- --overwrite` and apply deny-all network policy. |
| Credential rotated but new one also compromised | Verify the rotation channel is not compromised. Generate the new credential on a separate, trusted machine. Use Vault to distribute. |
| GitGuardian alert but secret cannot be found | Run full git history scan: `git log --all --diff-filter=A -- '*secret*'`. Use `trufflehog` or `ggshield` for deep scanning. |
| Country regulatory authority demands immediate notification | Follow the GDPR Art. 33 template in §5.6. Notify within 72 hours of becoming aware of the breach. |
| Security incident and P0 service outage occur simultaneously | The Security Incident Commander handles the security incident. A separate Incident Commander is assigned for the service outage. Both teams coordinate through the Communications Lead. |
| Successor designee is unavailable for credential rotation | Use the emergency contact method documented in → **[008-credential-inventory.md](009-zs-operations/008-credential-inventory.md)**. The maintainer holds the final break-glass access. |

---

## Cross-references

- → **[006-sop-credential-succession.md](009-zs-operations/006-sop-credential-succession.md)** — SOP-006: Credential documentation and rotation
- → **[008-credential-inventory.md](009-zs-operations/008-credential-inventory.md)** — Central credential inventory
- → **[010-sop-incident-response.md](009-zs-operations/010-sop-incident-response.md)** — SOP-010: General incident response
- → **[009-sop-deployment.md](009-zs-operations/009-sop-deployment.md)** — SOP-009: Deployment and rollback
- → **[011-adr-privacy-by-architecture.md](008-zs-adrs/011-adr-privacy-by-architecture.md)** — ADR-011: Privacy by architecture
- → **[012-adr-no-single-person-dependency.md](008-zs-adrs/012-adr-no-single-person-dependency.md)** — ADR-012: No single-person dependency
- → **[007-security-architecture.md](006-zs-infrastructure/007-security-architecture.md)** — Security architecture overview
- → **[008-security-policies.md](006-zs-infrastructure/008-security-policies.md)** — Security policies
- → **[009-compliance-controls.md](006-zs-infrastructure/009-compliance-controls.md)** — Compliance controls (HIPAA, GDPR)
- → **[010-threat-models.md](006-zs-infrastructure/010-threat-models.md)** — Threat models

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
