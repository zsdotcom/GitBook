# 006-sop-credential-succession.md
## SOP-006: Credential documentation, rotation, and succession procedures
### Standard Operating Procedure — credential succession

**Document type:** SOP
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Table of contents

1. [Purpose](#1-purpose)
2. [Scope](#2-scope)
3. [Roles & Responsibilities](#3-roles--responsibilities)
4. [Preconditions](#4-preconditions)
5. [Procedure](#5-procedure)
6. [Expected Outcome](#6-expected-outcome)
7. [Escalation](#7-escalation)

---

## 1. Purpose

To establish mandatory procedures for documenting, rotating, and succeeding all credentials, API keys, secrets, and access rights used across the ZarishSphere ecosystem. Every credential must have a documented rotation schedule (maximum 90 days), a designated successor who can rotate it, and a recovery path that does not depend on any single person's availability. This SOP implements Constitution Law 11 and ADR-012.

> **Constraint:** No ZarishSphere function, workflow, API key, credential, or operational process may depend on a single person's access, knowledge, or continued availability.

---

## 2. Scope

**In scope:** All ZarishSphere ecosystem credentials including:

- GitHub organisation owner and member access
- Cloudflare API tokens and Global API keys
- Domain registrar access and DNS management credentials
- Email relay and SMTP credentials
- CI/CD secrets stored in GitHub Actions
- Database connection strings and credentials
- Signing keys (GPG, SSH, code signing)
- Cloud provider access keys (AWS, GCP, Azure — if used)
- Any other secrets used in production, staging, or development environments
- The `008-credential-inventory.md` file in this folder

**Out of scope:** Personal credentials of individual contributors (e.g., personal GitHub accounts, personal email). End-user authentication credentials handled by the ZarishSphere Platform IAM system.

---

## 3. Roles & Responsibilities

| Role | Who | Responsibility |
|---|---|---|
| **Repository Owner** | Mohammad Ariful Islam | Primary holder of root credentials; ensures credential inventory is current |
| **Successor Designee** | Designated by Repository Owner | Authorised to rotate any credential if the Repository Owner is unavailable |
| **GitHub Org Members** | All contributors | Maintain their own personal access tokens and SSH keys; do not hold shared secrets |
| **Auditor** | Reviewer Agent (AI) or designated reviewer | Quarterly credential inventory review; annual succession test observer |

---

## 4. Preconditions

- → **[012-adr-no-single-person-dependency.md](008-zs-adrs/012-adr-no-single-person-dependency.md)** — ADR-012 is adopted and in effect.
- The `zarishsphere` GitHub organisation has been created (→ **[002-github-org-architecture.md](006-zs-infrastructure/002-github-org-architecture.md)**).
- The credential inventory file exists at `009-zs-operations/008-credential-inventory.md` or will be created in Step 5.1.
- All repository maintainers have read this SOP.
- A designated Successor Designee has been identified and has accepted the role.

---

## 5. Procedure

### 5.1 Maintain the credential inventory

Every credential used by the ZarishSphere ecosystem must be recorded in a single inventory file at:

```
009-zs-operations/008-credential-inventory.md
```

**To create or update the inventory:**

1. Open **VS Code → Explorer pane → expand 009-zs-operations/**.
2. If the file does not exist, right-click **009-zs-operations/** → **New File** and name it `008-credential-inventory.md`.
3. Use the following template for each credential entry:

```markdown
## Credential: [Name]

| Field | Value |
|---|---|
| **Purpose** | What this credential is used for |
| **Location** | Where it is stored (e.g., GitHub Actions secret, Cloudflare dashboard, env file) |
| **Rotation schedule** | Every N days (max 90) |
| **Last rotated** | YYYY-MM-DD |
| **Next rotation due** | YYYY-MM-DD |
| **Successor** | Name / GitHub handle of person who can rotate |
| **Rotation procedure** | Link to step number in this SOP |
| **Emergency contact** | Successor Designee contact info |
```

4. **Save the file** (`` Ctrl+S ``).
5. After saving, run the validation pipeline to ensure the inventory is properly formatted:
   ```bash
   python3 scripts/010-refresh-files.py
   ```

### 5.2 Define credential attributes

For each credential in the inventory, the following attributes must be defined:

1. **Purpose** — A clear, one-sentence description of what this credential unlocks.
   - Example: *"GitHub organisation owner token — used to manage org settings, branch protection, and repository creation."*
2. **Location** — The exact storage mechanism and path.
   - Example: *"Stored as `GH_ORG_OWNER_TOKEN` in GitHub Actions encrypted secrets for `zarishsphere/zs-docs`."*
3. **Rotation schedule** — Maximum interval between rotations: **90 days**.
   - Shorter intervals (30 or 60 days) are recommended for production-critical credentials.
4. **Successor** — The GitHub handle or full name of the person authorised to rotate this credential.
   - The successor must have acknowledged this responsibility in writing (GitHub issue or documented meeting note).

### 5.3 GitHub organisation credentials

GitHub organisation credentials are the most critical — they control all repositories, branch protection, and CI/CD pipelines.

**Owner account:**
1. Open a browser and navigate to **https://github.com/organizations/zarishsphere/settings**.
2. Under **Access → Personal access tokens**, verify the organisation owner token exists.
3. Document in the credential inventory:
   - **Purpose:** GitHub org owner access — full administrative control.
   - **Location:** GitHub Actions encrypted secret `GH_ORG_OWNER_TOKEN`.
   - **Rotation:** Every 90 days via GitHub UI: **Settings → Developer settings → Personal access tokens → Regenerate token**.
   - **Backup owner:** A second GitHub account with organisation owner privileges, documented in → **[002-github-org-architecture.md](006-zs-infrastructure/002-github-org-architecture.md)**.

**To add a backup owner:**
1. In the browser, go to **https://github.com/organizations/zarishsphere/settings**.
2. Click **People** in the left sidebar.
3. Click **Invite member** and enter the backup owner's GitHub handle.
4. Set role to **Owner**.
5. Click **Send invitation**.
6. The backup owner accepts the invitation via email notification.

### 5.4 Cloudflare API tokens

Cloudflare controls DNS, WAF, CDN, and edge routing for all ZarishSphere domains.

**To document and rotate Cloudflare tokens:**

1. Open a browser and navigate to **https://dash.cloudflare.com/profile/api-tokens**.
2. Review the list of API tokens. Each token must have a descriptive name (e.g., `zarishsphere-dns-edit`, `zarishsphere-pages-deploy`).
3. For each token, verify:
   - ☐ Token name clearly describes its purpose.
   - ☐ Token permissions are restricted to the minimum required (least-privilege).
   - ☐ Token TTL is set (max 90 days) or uses the "Rotate" feature.
4. Document each token in the credential inventory with its token ID (not the secret value).
5. Store each token value as a GitHub Actions encrypted secret (→ Step 5.6).

**To rotate a Cloudflare token:**
1. In the browser, navigate to **https://dash.cloudflare.com/profile/api-tokens**.
2. Click the token name.
3. Click **Rotate** (or **Regenerate**).
4. Copy the new token value immediately (it is shown only once).
5. Update the corresponding GitHub Actions encrypted secret with the new value (→ Step 5.6).
6. Update the credential inventory: set `Last rotated` to today's date and `Next rotation due` to +90 days.

> **Constraint:** Never store Cloudflare API tokens in plain text files, environment files, or anywhere outside GitHub Actions encrypted secrets or a password manager approved by the Repository Owner.

### 5.5 Domain and DNS access

Domain registrar credentials control the `zarishsphere.com` domain and all subdomains.

**To document domain access:**

1. Open a browser and navigate to the domain registrar's console (e.g., Namecheap, GoDaddy, Cloudflare Registrar).
2. Verify the domain contact information is current and includes:
   - Primary email: `security@zarishsphere.com`
   - Backup email: A secondary email not tied to the primary domain
   - Phone contact (if required)
3. In the registrar console, add a **backup contact** if the registrar supports it:
   - **Registrar dashboard → Domain Settings → Contacts → Add backup contact**.
4. Document in the credential inventory:
   - **Purpose:** Domain management for `zarishsphere.com` and all subdomains.
   - **Location:** Registrar console (list the specific registrar).
   - **Backup contact:** Name, email, phone of the Successor Designee.
5. If the registrar supports **Transfer Lock**, ensure it is enabled.

### 5.6 CI/CD secrets

All CI/CD secrets must be stored in GitHub Actions encrypted secrets — never in repository files.

**To add or update a CI/CD secret:**

1. Open a browser and navigate to the target repository on GitHub (e.g., **https://github.com/zsdotcom/zs-docs**).
2. Click **Settings → Secrets and variables → Actions**.
3. Click **New repository secret**.
4. Enter the **Name** (e.g., `CLOUDFLARE_API_TOKEN`).
5. Enter the **Value** (the secret string).
6. Click **Add secret**.
7. Document in the credential inventory:
   - **Location:** GitHub Actions encrypted secret for `zarishsphere/[repo-name]`.

**To rotate a CI/CD secret:**
1. Generate a new value for the credential (e.g., rotate the Cloudflare API token).
2. Open **GitHub → [repo] → Settings → Secrets and variables → Actions**.
3. Click **Update** next to the existing secret name.
4. Paste the new value.
5. Click **Update secret**.
6. Update the credential inventory.

All CI/CD secrets are documented in → **[006-ci-cd-architecture.md](006-zs-infrastructure/006-ci-cd-architecture.md)**.

### 5.7 Obituary and transition document

This is the single most important succession artifact — a markdown file containing ALL credentials, locations, rotation procedures, and contacts, stored encrypted and accessible.

**To create the transition document:**

1. In **VS Code → Explorer pane → 009-zs-operations/** → right-click → **New File**.
2. Name it `credential-transition-plan.md.enc` (or `.gpg` if using GPG encryption).
3. Structure the document as follows:

```markdown
# ZarishSphere Credential Transition Plan
## To be used only if the Repository Owner is unavailable

## Emergency contacts
- Successor Designee: [Name], [Email], [Phone]
- Backup contact: [Name], [Email], [Phone]

## Quick-reference credential table
| Credential | Location | Rotate via | Successor |
|---|---|---|---|
| GitHub org owner | GH Actions secret | GitHub UI | [Name] |
| Cloudflare API token | GH Actions secret | Cloudflare dashboard | [Name] |
| Domain registrar | [Registrar] console | Registrar dashboard | [Name] |
| ... | ... | ... | ... |

## Full credential inventory
[Copy of each entry from 008-credential-inventory.md]

## Recovery procedures
Step-by-step for each credential:
1. How to access the system
2. How to rotate the credential
3. How to verify the new credential works
4. How to document the rotation
```

4. **Encrypt the file** using GPG or a password manager:
   ```bash
   gpg --symmetric --cipher-algo AES256 credential-transition-plan.md
   ```
5. Store the encrypted file in the repository at `009-zs-operations/credential-transition-plan.md.enc`.
6. Store the decryption passphrase in a physically separate location (e.g., printed and stored in a safe, or with a trusted third party).
7. Verify that the Successor Designee can decrypt and read the document.

### 5.8 Annual succession test

Once per year, the Repository Owner and Successor Designee must walk through the full credential recovery process.

**To conduct the succession test:**

1. Schedule a 2-hour block on the calendar (set a recurring annual event).
2. The Repository Owner simulates unavailability (does not participate in the exercise).
3. The Successor Designee performs the following, documented as a GitHub issue:
   - Retrieves the encrypted transition document from the repository.
   - Decrypts it using the stored passphrase.
   - Identifies all credentials in the inventory.
   - Selects **one production credential** (e.g., a Cloudflare API token) and rotates it following the documented procedure.
   - Verifies the rotated credential works (e.g., runs a Cloudflare API call).
   - Documents the rotation in the credential inventory.
4. After the test, the Repository Owner reviews the issue and updates any procedures that were unclear or incorrect.
5. File a GitHub issue with label `succession-test` and the year:
   - **Title:** `Succession test YYYY results`
   - **Body:** Summary of what was tested, what succeeded, what needs improvement.

**Checklist item:** ☐ Annual succession test completed and documented as a GitHub issue.

---

## 6. Expected Outcome

| Check | Frequency | Method |
|---|---|---|
| Credential inventory is current | Quarterly | Review → **[008-credential-inventory.md](009-zs-operations/008-credential-inventory.md)** — verify every entry has a successor and next rotation date |
| Rotation dates are enforced | Quarterly | Check `Next rotation due` for every credential — any overdue by more than 7 days triggers escalation |
| GitHub organisation has backup owner | Quarterly | Open **GitHub → org Settings → People** — verify at least 2 owners |
| CI/CD secrets are encrypted | Quarterly | Verify no plain-text secrets in repository files: `git grep -i "secret\|token\|password\|api_key" -- ':!.opencode/' ':!*.md.enc'` |
| Annual succession test | Yearly | Confirm the GitHub issue with label `succession-test-YYYY` is filed and reviewed |
| Transition document decryptable | Yearly | Successor Designee attempts decryption and confirms access |

---

## 7. Escalation

| Issue | Action |
|---|---|
| Credential rotation overdue by more than 7 days | Notify the Successor Designee immediately. If the credential's successor is also unavailable, escalate to the Repository Owner via email at `security@zarishsphere.com`. |
| Credential inventory entry missing successor | Do not create the credential until a successor is assigned. For existing entries, block any rotation until a successor is documented. |
| Transition document cannot be decrypted | File a GitHub issue with label `critical` and `credential`. Contact the Repository Owner and Successor Designee immediately. Recreate the document from the credential inventory. |
| Successor Designee becomes unavailable | The Repository Owner must designate a new successor within 7 days. Update all credential inventory entries and the transition document. |
| GitHub organisation has only one owner | Add a second owner immediately via **GitHub → Org Settings → People → Invite member**. |
| Credential exposed or compromised | 1. Rotate the credential immediately (follow the rotation procedure for that credential type). 2. File a security incident GitHub issue with label `security-incident`. 3. Review access logs for the compromised period. 4. Document the incident and remediation in the credential inventory. |
| Repository Owner permanently unavailable | The Successor Designee assumes the Repository Owner role. Decrypt the transition document. Rotate ALL credentials within 7 days. File a constitutional notification issue documenting the succession. |

---

## Cross-references

- → **[001-zarishsphere-constitution.md](001-zs-meta/001-zarishsphere-constitution.md)** — Law 11: "The platform outlives its creators"
- → **[012-adr-no-single-person-dependency.md](008-zs-adrs/012-adr-no-single-person-dependency.md)** — ADR-012: No single-person dependency
- → **[002-github-org-architecture.md](006-zs-infrastructure/002-github-org-architecture.md)** — GitHub organisation credentials and backup owner
- → **[003-cloudflare-architecture.md](006-zs-infrastructure/003-cloudflare-architecture.md)** — Cloudflare API token configuration
- → **[006-ci-cd-architecture.md](006-zs-infrastructure/006-ci-cd-architecture.md)** — CI/CD secrets management
- → **[003-zarishsphere-founder-profile.md](001-zs-meta/003-zarishsphere-founder-profile.md)** — §5.3: Founder infrastructure constraints
- → **[008-credential-inventory.md](009-zs-operations/008-credential-inventory.md)** — Credential inventory file (maintained by this SOP)

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
