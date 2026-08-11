# 008-credential-inventory.md
## ZarishSphere ecosystem — credential inventory
### Maintained per SOP-006: Credential documentation, rotation, and succession procedures

**Document type:** Reference
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Stable

---

## Template entry

Each credential entry follows this structure:

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
| **Rotation procedure** | See SOP-006 §5. [step] |
| **Emergency contact** | Successor Designee contact info |
```

---

## Credential entries

*No entries yet. Add entries following the template above.*

---

## Cross-references

- → **[006-sop-credential-succession.md](009-zs-operations/006-sop-credential-succession.md)** — SOP-006: Credential succession procedures
- → **[012-adr-no-single-person-dependency.md](008-zs-adrs/012-adr-no-single-person-dependency.md)** — ADR-012: No single-person dependency
- → **[002-github-org-architecture.md](006-zs-infrastructure/002-github-org-architecture.md)** — GitHub organisation credentials
- → **[003-cloudflare-architecture.md](006-zs-infrastructure/003-cloudflare-architecture.md)** — Cloudflare API token configuration
- → **[006-ci-cd-architecture.md](006-zs-infrastructure/006-ci-cd-architecture.md)** — CI/CD secrets management

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
