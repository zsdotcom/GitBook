# 014-sop-facility-onboarding.md
## SOP-014: Onboarding a new health facility to ZarishSphere
### Standard Operating Procedure — facility registration and activation

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

To define the complete procedure for onboarding a new health facility to an existing ZarishSphere country deployment — covering facility registration in the FHIR Location registry, Keycloak tenant creation, country distribution update, staff training, and CAMM status advancement. Each new facility moves the country deployment closer to full operational capability.

---

## 2. Scope

**In scope:** Registering any health facility (health post, sub-district clinic, district hospital, or referral hospital) in an active ZarishSphere country deployment. Creating the FHIR Location resource in `zs-data-facility-registry`. Creating a Keycloak tenant for the facility. Updating the country distribution (`zs-distro-{cc}`). Training facility staff in ZarishSphere usage. Updating the CAMM country status to reflect the new facility count.

**Out of scope:** Onboarding a new country (see → **[013-sop-country-onboarding.md](009-zs-operations/013-sop-country-onboarding.md)**). Infrastructure provisioning for a new country. Custom form creation. Advanced FHIR profiling (see → **[006-fhir-profiling-policy.md](005-zs-standards/006-fhir-profiling-policy.md)**).

---

## 3. Roles

| Role | Who |
|---|---|
| **Onboarding Operator** | Person executing the facility onboarding (country focal point or maintainer) |
| **Reviewer** | Maintainer who approves the GitHub PR for the facility registration |
| **Trainer** | Person conducting facility staff training (usually the country focal point) |
| **Approver** | `@arwazarish` or country focal point — authorises facility activation |

---

## 4. Preconditions

- The country is already onboarded to ZarishSphere (CAMM L0 or higher).
- The country focal point has a GitHub account with access to `zarishsphere` organisation.
- The Onboarding Operator has the facility information form completed (see §5.1).
- The facility has internet connectivity (4G/3G/2G/satellite) or a plan for offline operation.
- The facility has at least one device (Android phone, tablet, or desktop computer) for staff use.

---

## 5. Steps

### 5.1 Gather facility information

Collect the following information before starting. Use this checklist:

```
□ Facility Name (official):
□ Facility Type: [ ] Health Post  [ ] Sub-district  [ ] District  [ ] Referral
□ Country Code: BGD / IND / MMR / PAK / THA
□ Administrative Division: [Division/State/Province]
□ District:
□ Upazila/Sub-district:
□ Village/Ward:
□ GPS Coordinates (Lat, Long):
□ Facility Code (official government code, if any):
□ Manager Name:
□ Manager Email:
□ Number of Staff (approx):
□ Internet connectivity: [ ] 4G  [ ] 3G  [ ] 2G  [ ] Satellite  [ ] None
□ Device availability: [ ] Android phones  [ ] Tablets  [ ] Desktop computers
```

**Checklist item:** ☐ All facility information fields completed.

### 5.2 Register facility in FHIR Location registry

All steps are performed in a browser via the GitHub UI.

1. Open a browser and navigate to `https://github.com/zsdotcom/zs-data-facility-registry`.
2. Click **"Add file"** → **"Create new file"**.
3. Name the file: `facilities/{cc}/{FACILITY-CODE}.json`
   - Example: `facilities/bgd/00123.json`
4. Paste and fill in the FHIR Location template:

   ```json
   {
     "resourceType": "Location",
     "id": "{cc}-{FACILITY-CODE}",
     "meta": {
       "profile": ["https://zarishsphere.com/fhir/StructureDefinition/ZSLocation"]
     },
     "status": "active",
     "name": "{OFFICIAL FACILITY NAME}",
     "type": [{
       "coding": [{
         "system": "http://terminology.hl7.org/CodeSystem/v3-RoleCode",
         "code": "HOSP",
         "display": "Hospital"
       }]
     }],
     "address": {
       "country": "{CC}",
       "state": "{DIVISION}",
       "district": "{DISTRICT}",
       "city": "{UPAZILA}",
       "text": "{FULL ADDRESS}"
     },
     "position": {
       "longitude": {LONGITUDE},
       "latitude": {LATITUDE}
     },
     "identifier": [{
       "system": "https://zarishsphere.com/fhir/NamingSystem/{cc}-facility-code",
       "value": "{OFFICIAL-GOVT-CODE}"
     }]
   }
   ```

5. At the bottom, enter a commit message:
   ```
   content(facilities): add {FACILITY NAME} {CC}
   ```
6. Select **"Create a new branch for this commit and start a pull request"**.
7. Click **"Propose new file"**.
8. On the PR page, click **"Create pull request"**.
9. Assign the PR to a reviewer (maintainer).
10. Wait for the PR to be approved and merged.

**Checklist item:** ☐ Facility FHIR Location PR created and merged.

### 5.3 Create Keycloak tenant for the facility

#### Via Keycloak Admin UI (recommended for non-technical users):

1. Open a browser and navigate to `https://auth.zarishsphere.com`.
2. Log in as an admin (use the country realm: `zarishsphere-{cc}`).
3. Navigate to **Clients** → **Create client**.
4. Set:
   - **Client ID:** `fac-{cc}-{FACILITY-CODE}`
   - **Client authentication:** ON (confidential)
   - **Standard flow:** Enabled
5. Go to **Settings** → set **Home URL** and **Valid redirect URIs**.
6. Click **"Save"**.
7. Navigate to **Users** → **Add user**.
8. Create the facility manager's user account:
   - **Username:** `{facility-code}-manager`
   - **Email:** {manager email}
   - **First name:** {manager first name}
   - **Last name:** {manager last name}
9. Go to **Credentials** → set a temporary password (force change on first login).
10. Go to **Role mapping** → assign `facility-admin` role.

#### Via Keycloak CLI (if preferred):

```bash
kcadm.sh create clients -r zarishsphere-{cc} \
  -s clientId="fac-{cc}-{FACILITY-CODE}" \
  -s enabled=true \
  -s publicClient=false
```

**Checklist item:** ☐ Keycloak client and facility manager account created.

### 5.4 Update country distribution

1. Open a browser and navigate to `https://github.com/zsdotcom/zs-distro-{cc}`.
2. Open `facilities/facility-registry.json`.
3. Click the ✏️ (edit) icon.
4. Add the new facility to the JSON array:
   ```json
   {
     "id": "{cc}-{FACILITY-CODE}",
     "name": "{OFFICIAL FACILITY NAME}",
     "status": "active",
     "type": "{facility type}"
   }
   ```
5. At the bottom, enter a commit message:
   ```
   content(facilities): add {FACILITY NAME} to facility registry
   ```
6. Select **"Create a new branch for this commit and start a pull request"**.
7. Click **"Propose changes"** → **"Create pull request"**.
8. Assign the PR to a reviewer.
9. Wait for approval and merge.

**Checklist item:** ☐ Country distribution updated with new facility.

### 5.5 Train facility staff

Conduct training for facility staff using the ZarishSphere training materials at `https://docs.zarishsphere.com/training`.

| Training Module | Audience | Duration | Method |
|---|---|---|---|
| ZarishSphere Overview | All staff | 30 min | Video or in-person |
| Patient Registration | Registration clerks | 2 hours | Hands-on practice |
| Clinical Data Entry | Nurses, clinicians | 4 hours | Guided session |
| Mobile App (if CHW) | Community health workers | 3 hours | Hands-on |
| System Administration | Facility manager | 2 hours | Walkthrough |

**Checklist item:** ☐ Facility staff trained on all relevant modules.

### 5.6 Update CAMM country status

1. Open a browser and navigate to `https://github.com/zsdotcom/zs-docs-camm`.
2. Open `countries/{CC}-STATUS.md`.
3. Click the ✏️ (edit) icon.
4. Update:
   - Facility count (increment by 1).
   - Date of the new facility onboarding.
5. At the bottom, enter a commit message:
   ```
   docs(camm): add {FACILITY NAME} to {CC} facility count
   ```
6. Select **"Create a new branch for this commit and start a pull request"**.
7. Click **"Propose changes"** → **"Create pull request"**.
8. Assign the PR to the Approver.

**Checklist item:** ☐ CAMM status updated with new facility count.

### 5.7 Verify onboarding complete

Run this final verification checklist:

```
□ Facility appears in FHIR Location registry:
   curl -s https://api.zarishsphere.com/fhir/R5/Location/{cc}-{FACILITY-CODE} | jq .resourceType
   Expected: "Location"

□ Facility users can log in to ZarishSphere:
   (Test login with the facility manager account at https://{country-console-url})

□ Test patient successfully registered:
   (Register a test patient through the Console UI)

□ Test encounter successfully created:
   (Create a test encounter for the test patient)

□ AuditEvent generated for test encounter:
   Check Grafana audit log dashboard

□ Facility visible in Grafana monitoring:
   Open https://grafana.zarishsphere.com → Country dashboard → verify facility appears

□ Staff confirmed trained:
   Training attendance record signed by facility manager

□ CAMM country status updated:
   Verify updated facility count in zs-docs-camm/countries/{CC}-STATUS.md
```

**Checklist item:** ☐ All 8 verification checks pass.

---

## 6. Expected outcome

- A new FHIR Location resource is registered in the `zs-data-facility-registry` repository.
- A Keycloak client and facility manager account are created for the facility.
- The country distribution (`zs-distro-{cc}`) is updated with the new facility.
- Facility staff are trained on ZarishSphere usage.
- CAMM country status is updated with the new facility count.
- The facility can register patients, create encounters, and generate audit events.
- The facility appears in Grafana monitoring dashboards.

---

## 7. Escalation

| Issue | Action |
|---|---|
| FHIR Location PR not merged within 24 hours | Tag the reviewer in the PR comments. If no response within 48 hours, notify `@arwazarish`. |
| Keycloak admin login fails | Reset the admin password: `docker compose exec keycloak /opt/keycloak/bin/kcadm.sh update user/{admin-user} -r master -s enabled=true`. |
| Facility manager email bounces when sending Keycloak credentials | Use the Keycloak Admin UI to manually set a new temporary password. Provide it to the manager via phone or in-person. |
| Training session needs rescheduling | Coordinate with the facility manager. Use the online training materials as self-paced backup. |
| Facility has no internet connectivity | Deploy offline mode: configure PowerSync for local-first operation (see → **[021-adr-powersync-mobile-offline.md](008-zs-adrs/021-adr-powersync-mobile-offline.md)**). Sync data when connectivity is available. |
| Facility registry PR conflicts with another onboarding | Resolve conflicts in the GitHub PR UI. If the same file was modified, merge the changes manually. |
| Staff training takes longer than planned | Schedule follow-up sessions. Provide printed quick-reference guides. Assign a mentor from a nearby facility. |
| CAMM status file does not exist yet for the country | Create `countries/{CC}-STATUS.md` following the template in `zs-docs-camm/templates/COUNTRY-STATUS-TEMPLATE.md`. |

---

## Cross-references

- → **[013-sop-country-onboarding.md](009-zs-operations/013-sop-country-onboarding.md)** — SOP-013: Country onboarding (precedes facility onboarding)
- → **[002-sop-github-workflow.md](009-zs-operations/002-sop-github-workflow.md)** — SOP-002: GitHub PR workflow
- → **[005-fhir-r5-conventions.md](005-zs-standards/005-fhir-r5-conventions.md)** — FHIR R5 conventions for Location resources
- → **[006-fhir-profiling-policy.md](005-zs-standards/006-fhir-profiling-policy.md)** — FHIR profiling policy
- → **[016-terminology-governance.md](005-zs-standards/016-terminology-governance.md)** — Terminology governance
- → **[021-adr-powersync-mobile-offline.md](008-zs-adrs/021-adr-powersync-mobile-offline.md)** — ADR-021: PowerSync for offline capability
- → **[005-fhir-architecture.md](003-zs-platform/005-fhir-architecture.md)** — FHIR architecture overview

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
