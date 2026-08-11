# 013-sop-country-onboarding.md
## SOP-013: Onboarding a new country to the ZarishSphere Platform
### Standard Operating Procedure — new country deployment

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

To define the complete procedure for onboarding a new country to the ZarishSphere Platform — from initial preconditions (MOU signed, focal point identified) through GitHub repository setup, infrastructure deployment, Keycloak configuration, data loading, and focal point training. This SOP guides the CAMM L0 to L1 transition (initial deployment to first patient record).

---

## 2. Scope

**In scope:** Creating country-specific GitHub repositories (`zs-distro-{cc}`, `zs-infra-{cc}`). Forking the core distribution template. Adding the country focal point to the GitHub organisation. Setting up country infrastructure on Oracle Cloud Free Tier, Raspberry Pi 5, or Render.com. Deploying the ZarishSphere stack via Docker Compose. Configuring Keycloak with country realm, clients, roles, and users. Loading country facility registry, forms, and terminology. Verifying deployment with smoke tests. Training the country focal point. Updating CAMM country status.

**Out of scope:** Onboarding individual health facilities (see → **[014-sop-facility-onboarding.md](009-zs-operations/014-sop-facility-onboarding.md)**). Ongoing country support after onboarding. Custom domain configuration (see → **[004-domain-architecture.md](006-zs-infrastructure/004-domain-architecture.md)**). Security incident response during onboarding (see → **[011-sop-security-incident.md](009-zs-operations/011-sop-security-incident.md)**).

---

## 3. Roles

| Role | Who |
|---|---|
| **Onboarding Lead** | Owner executing the onboarding procedure (default: `@devopsariful`) |
| **Country Focal Point** | Designated technical contact from the new country |
| **Approver** | `@arwazarish` — provides approval to proceed with onboarding |
| **Trainer** | Person conducting focal point training (L1 curriculum) |

---

## 4. Preconditions

- [ ] MOU signed (see `zs-docs-camm/templates/MOU-TEMPLATE.md`).
- [ ] Country focal point has a GitHub account.
- [ ] Country 2-letter code determined (ISO 3166-1 alpha-2): e.g., BGD, IND, MMR.
- [ ] Target cloud region or hardware determined (Oracle Cloud / RPi / Render).
- [ ] `@arwazarish` approval obtained to proceed.
- [ ] The Onboarding Lead has admin access to the `zarishsphere` GitHub organisation.
- [ ] The Onboarding Lead has SSH access to the target server or cloud account credentials.

---

## 5. Steps

### 5.1 Create GitHub repositories (15 minutes)

All steps are performed in a browser via the GitHub UI.

1. Open a browser and navigate to `https://github.com/orgs/zarishsphere/repositories`.
2. Click **"New repository"**.
3. Create `zs-distro-{cc}`:
   - **Repository name:** `zs-distro-{cc}` (e.g., `zs-distro-bgd`)
   - **Description:** `"ZarishSphere distribution: {Country Name}"`
   - **Visibility:** Public
   - **License:** Apache 2.0
   - Click **"Create repository"**.
4. Click **"New repository"** again.
5. Create `zs-infra-{cc}`:
   - **Repository name:** `zs-infra-{cc}` (e.g., `zs-infra-bgd`)
   - **Description:** `"ZarishSphere country infrastructure: {Country Name}"`
   - **Visibility:** Public
   - **License:** Apache 2.0
   - Click **"Create repository"**.

**Checklist item:** ☐ `zs-distro-{cc}` created.
**Checklist item:** ☐ `zs-infra-{cc}` created.

### 5.2 Fork country distribution template (10 minutes)

1. In a browser, navigate to `https://github.com/zsdotcom/zs-distro-core`.
2. Click **"Use this template"** → **"Create a new repository"**.
3. Select owner: `zarishsphere`.
4. Repository name: `zs-distro-{cc}` (select the one created in §5.1).
5. Click **"Create repository from template"**.
6. In the new repository, edit `distro.yaml`:
   - Open the file and click the ✏️ (edit) icon.
   - Update the content:
     ```yaml
     metadata:
       name: zs-distro-{cc}
       country: {Country Name}
       countryCode: {CC}
       language: {language code}
       version: 1.0.0
     ```
   - Commit directly to `main`.
7. Edit `CODEOWNERS`:
   - Add the country focal point's GitHub username to the file.
   - Commit directly to `main`.

**Checklist item:** ☐ `distro.yaml` updated with country-specific values.

### 5.3 Add country focal point to GitHub organisation (5 minutes)

1. Open a browser and navigate to `https://github.com/orgs/zarishsphere/people`.
2. Click **"Invite member"**.
3. Enter the focal point's GitHub username.
4. Set **Role:** Member (not Owner).
5. Click **"Invite"**.
6. Create a team for the country:
   - Navigate to `https://github.com/orgs/zarishsphere/teams`.
   - Click **"New team"**.
   - Team name: `country-{cc}-team`
   - Add the focal point to the team.
7. Grant the team access to `zs-distro-{cc}` and `zs-infra-{cc}`.

**Checklist item:** ☐ Country focal point added to GitHub organisation and team.

### 5.4 Add country status file (10 minutes)

1. Navigate to `https://github.com/zsdotcom/zs-docs-camm`.
2. Open or create `countries/{CC}-STATUS.md`.
3. Update the content:
   - Set current CAMM level to L0.
   - Add focal point details (name, email, GitHub handle).
   - Add MOU signing date.
4. Commit with message: `docs(camm): add {Country Name} status at L0`.

**Checklist item:** ☐ CAMM status file updated to L0.

### 5.5 Set up country infrastructure (1–2 hours)

Choose one deployment option based on country requirements:

#### Option A: Oracle Cloud Free Tier (recommended for production pilots)

1. Sign up at `https://cloud.oracle.com` using the country focal point's email.
2. Provision 2x ARM Ampere A1 Compute instances (free forever):
   - 2 OCPUs, 12 GB RAM each.
3. Install Docker 29.x on each instance:
   ```bash
   ssh {user}@{instance-ip}
   sudo apt update && sudo apt install -y docker.io docker-compose-v2
   sudo systemctl enable docker && sudo systemctl start docker
   ```
4. Document the instance IP addresses in `zs-infra-{cc}/README.md`.

#### Option B: Raspberry Pi 5 (for offline / low-resource settings)

1. Flash Ubuntu Server 24.04 ARM64 to an SD card (use Raspberry Pi Imager).
2. Set a static IP address on the facility network.
3. Install Docker 29.x:
   ```bash
   sudo apt update && sudo apt install -y docker.io docker-compose-v2
   ```
4. Document the static IP in `zs-infra-{cc}/README.md`.

#### Option C: Render.com (for quick pilots only)

1. Create a Render account.
2. Deploy `zs-iac-dev-environment` Docker Compose to Render.
   - Note: 750 hours/month free — suitable for pilot only.
3. Document the Render service URL in `zs-infra-{cc}/README.md`.

**Checklist item:** ☐ Country infrastructure provisioned and documented.

### 5.6 Deploy ZarishSphere stack (30 minutes)

1. SSH to the provisioned server:
   ```bash
   ssh {user}@{server-ip}
   ```
2. Clone the development environment:
   ```bash
   git clone https://github.com/zsdotcom/zs-iac-dev-environment
   cd zs-iac-dev-environment
   ```
3. Configure the environment:
   ```bash
   cp .env.example .env
   nano .env
   ```
   Set country-specific values:
   - `COUNTRY_CODE={cc}`
   - `KEYCLOAK_ADMIN_PASSWORD={strong-password}`
   - `POSTGRES_PASSWORD={strong-password}`
4. Start the stack:
   ```bash
   docker compose up -d
   ```
5. Verify all services are running:
   ```bash
   docker compose ps
   ```
   Expected: All services showing `Up` status.

**Checklist item:** ☐ ZarishSphere stack deployed and all services running.

### 5.7 Configure Keycloak (30 minutes)

All steps are performed in a browser via the Keycloak Admin UI.

1. Open a browser and navigate to `https://{server-ip}:8443`.
2. Log in with the admin credentials set in `.env`.
3. **Create a realm:**
   - Hover over the realm dropdown (top-left) → Click **"Create realm"**.
   - **Realm name:** `zarishsphere-{cc}`
   - Click **"Create"**.
4. **Create clients:**
   - Navigate to **Clients** → **Create client**.
   - Create `zs-fhir-engine` (confidential, service account roles enabled).
   - Create `zs-ui-clinical` (public, standard flow).
5. **Create roles:**
   - Navigate to **Realm roles** → **Create role**.
   - Create: `clinician`, `nurse`, `chw`, `supervisor`, `admin`.
6. **Create initial admin user:**
   - Navigate to **Users** → **Add user**.
   - Create the country focal point's admin account.
   - Set a temporary password (force change on first login).
   - Assign the `admin` role.

**Checklist item:** ☐ Keycloak realm, clients, roles, and admin user created.

### 5.8 Load country data (30 minutes)

1. On the server, load the country facility registry:
   ```bash
   cd zs-distro-{cc}
   ./scripts/load-facilities.sh
   ```
2. Load country forms:
   ```bash
   ./scripts/load-forms.sh
   ```
3. Load country terminology mappings:
   ```bash
   ./scripts/load-terminology.sh
   ```

**Checklist item:** ☐ Country data loaded (facilities, forms, terminology).

### 5.9 Verify deployment (15 minutes)

Run smoke tests on the deployed server:

1. Test the FHIR API:
   ```bash
   curl -s http://{server-ip}:8000/fhir/R5/metadata | jq .resourceType
   ```
   Expected: `"CapabilityStatement"`

2. Test authentication:
   ```bash
   curl -s http://{server-ip}:8443/realms/zarishsphere-{cc}/.well-known/openid-configuration | jq .issuer
   ```
   Expected: `"http://{server-ip}:8443/realms/zarishsphere-{cc}"`

**Checklist item:** ☐ FHIR API responding with CapabilityStatement.

**Checklist item:** ☐ Keycloak authentication working.

### 5.10 Train country focal point (2 hours)

Follow the L1 training track in `zs-docs-camm/templates/TRAINING-CURRICULUM.md`:

| Module | Duration | Topics |
|---|---|---|
| ZarishSphere overview | 30 min | Platform architecture, components, interfaces |
| Console navigation | 30 min | Login, dashboard, facility view, data entry |
| FHIR API basics | 30 min | REST API, resource types, search parameters |
| Troubleshooting | 30 min | Common issues, logging, support channels |

**Checklist item:** ☐ Country focal point completed L1 training.

---

## 6. Expected outcome

- GitHub repositories `zs-distro-{cc}` and `zs-infra-{cc}` are created and configured.
- Country focal point is added to the GitHub organisation as a member.
- CAMM country status is set to L0 in `zs-docs-camm`.
- Infrastructure is provisioned (Oracle Cloud, RPi, or Render) and documented.
- ZarishSphere stack is deployed and all services are running.
- Keycloak is configured with a country realm, clients, roles, and admin user.
- Country data (facilities, forms, terminology) is loaded.
- FHIR API and Keycloak respond correctly to smoke tests.
- Country focal point is trained (L1 curriculum complete).
- The country is ready to advance to CAMM L1 after the first patient record.

---

## 7. Escalation

| Issue | Action |
|---|---|
| Oracle Cloud Free Tier sign-up fails | Try a different email address. Oracle Free Tier requires a valid credit card for verification (no charges). If still failing, use Option B (Raspberry Pi 5). |
| Docker compose fails to start | Check `docker compose logs` for specific errors. Common issues: port conflicts, missing `.env` variables, incorrect Docker version. Verify Docker 29.x is installed. |
| Keycloak realm creation fails | Verify the Keycloak admin password in `.env`. Restart Keycloak: `docker compose restart keycloak`. |
| FHIR API not responding (HTTP 503) | Check that the FHIR engine container is running: `docker compose ps`. Check logs: `docker compose logs fhir-engine`. |
| Country focal point does not accept GitHub invitation | Resend the invitation. Ensure the email address associated with the GitHub account is correct. |
| Data load scripts fail | Verify the data files exist in `zs-distro-{cc}`. Check for missing or invalid JSON. Validate against the FHIR schema. |
| Country focal point training needs more time | Schedule a follow-up session. Provide access to training recordings and documentation at `https://docs.zarishsphere.com/training`. |
| MOU not yet signed | Do not proceed past preconditions. Notify `@arwazarish` to follow up on the MOU signing. |
| Infrastructure cost concerns | All three options (Oracle Free Tier, RPi 5, Render free tier) are zero-cost for initial pilot. If scale-up costs are a concern, contact `@arwazarish` for grant funding options. |

---

## Cross-references

- → **[014-sop-facility-onboarding.md](009-zs-operations/014-sop-facility-onboarding.md)** — SOP-014: Facility onboarding (next step after country onboarding)
- → **[002-sop-github-workflow.md](009-zs-operations/002-sop-github-workflow.md)** — SOP-002: GitHub repository and PR workflow
- → **[009-sop-deployment.md](009-zs-operations/009-sop-deployment.md)** — SOP-009: Deployment procedures
- → **[012-sop-backup-dr.md](009-zs-operations/012-sop-backup-dr.md)** — SOP-012: Backup and DR for country deployments
- → **[002-github-org-architecture.md](006-zs-infrastructure/002-github-org-architecture.md)** — GitHub organisation architecture
- → **[003-cloudflare-architecture.md](006-zs-infrastructure/003-cloudflare-architecture.md)** — Cloudflare architecture
- → **[004-domain-architecture.md](006-zs-infrastructure/004-domain-architecture.md)** — Domain architecture
- → **[003-deployment-planes.md](003-zs-platform/003-deployment-planes.md)** — Deployment planes (L0–L4)
- → **[006-adr-zero-cost-toolchain.md](008-zs-adrs/006-adr-zero-cost-toolchain.md)** — ADR-006: Zero-cost toolchain

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
