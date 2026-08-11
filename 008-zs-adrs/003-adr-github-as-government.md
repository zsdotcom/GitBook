# 003-adr-github-as-government.md
## ADR-003: GitOps and GitHub as the Operational Control Plane
### GitHub as the single source of truth for governance, configuration, and decision records

**Document type:** ADR
**Date:** August 01, 2026
**Author:** Mohammad Ariful Islam / ZarishSphere Foundation
**License:** Apache 2.0 (code) · CC BY 4.0 (documentation)
**Status:** V1 — Accepted

---

## Table of Contents

1. [Decision](#1-decision)
2. [Context](#2-context)
3. [Alternatives Considered](#3-alternatives-considered)
4. [Reason for Decision](#4-reason-for-decision)
5. [Consequences](#5-consequences)
6. [Status](#6-status)
7. [Cross-references](#7-cross-references)

---

## 1. Decision

Use GitHub as the sole operational control plane for the ZarishSphere ecosystem. All governance decisions, infrastructure configuration, architecture decision records, deployment definitions, and operational procedures are committed to repositories under the `zarishsphere` GitHub organization. No decision is binding unless it exists as a git commit (Constitution Law 1). GitHub Actions is the CI/CD engine. GitHub Issues and Projects are the task tracking system. GitHub Pages serves the documentation site.

---

## 2. Context

The ZarishSphere ecosystem requires a source-of-truth system that:

- Provides immutable, timestamped, attributable history for every decision (Constitution Law 10)
- Works fully offline via local git clones for Plane 0 (air-gapped) deployments
- Is zero-cost with a genuine free tier (Constitution Law 5)
- Supports CI/CD for automated validation (ZUSS compliance, cross-reference checking)
- Enables AI agent integration (opencode agent ecosystem reads/writes files directly)
- Provides a collaboration surface for future contributors (Constitution Law 12 — borderless contribution)

Three candidate platforms were evaluated: GitHub, GitLab, and a self-hosted Gitea instance.

---

## 3. Alternatives Considered

| Alternative | Pros | Cons |
|---|---|---|
| **GitHub Free Org** | Free org with unlimited repositories and collaborators; GitHub Actions (2,000 min/month free); GitHub Pages for docs; GitHub Issues + Projects for task management; largest ecosystem (Copilot, opencode); community familiarity maximizes future contributor pool; 3rd-party integrations with Cloudflare Pages, Vercel | Internet required for remote sync (mitigated by git bundles for Plane 0); 2,000 min/month Actions limit may constrain CI; 500 MB GitHub Pages limit; private repos limited to 3 collaborators (but public repos are unlimited) |
| **GitLab Free (SaaS)** | Unlimited repos, 400 CI/CD minutes/month, integrated container registry, built-in CI/CD with more flexible runners | Heavier platform — slower UI on low-bandwidth connections; 400 CI/month minutes insufficient for active development; less community adoption (contributors less likely to have GitLab accounts); fewer AI/agent integrations; no direct Pages equivalent as polished as GitHub Pages |
| **Self-hosted Gitea** | Complete data sovereignty, no third-party dependency, no usage limits, works offline by default, extremely lightweight (runs on 1 GB RAM / RPi) | Additional server management burden for solo founder; requires open port / VPN for remote contributors; no built-in CI/CD (would need Woodpecker or Drone); no GitHub-native tooling (Actions, Copilot, opencode); 8 GB RAM development machine cannot spare resources for server; requires backup and maintenance — violates Law 11 (succession) if only the founder can maintain it |

---

## 4. Reason for Decision

1. **Constitutional mandate:** Constitution Law 1 (GitHub is the government) is a Tier I law — it can never be amended. The law explicitly states: "Every form, protocol, configuration, deployment, and governance decision in the ZarishSphere ecosystem is a git commit." GitHub is the implementation vehicle for this law. Choosing any other platform would contradict Tier I constitutional law, which constitutes a fork of the project.

2. **Free tier adequacy:** GitHub Free Organization provides all needed features: unlimited public repositories, GitHub Actions (2,000 min/month, sufficient for V1), GitHub Pages (500 MB, sufficient for zs-docs), GitHub Issues (unlimited), and GitHub Projects (basic Kanban). No credit card required. No time limit on free tier.

3. **Git-native offline support:** git is a distributed version control system. A Plane 0 deployment can receive updates via `git bundle` (a single file containing all commits), transferred by USB drive or local network. The machine running Plane 0 never needs direct internet access to GitHub. No other platform provides this offline distribution mechanism as naturally.

4. **AI agent ecosystem integration:** The `zarishsphere/zs-docs` repository is designed for AI agent consumption (`.opencode/` agent ecosystem, `llms.txt`, structured YAML front matter). GitHub's API and git-native architecture enable agents to read, create, and validate documents programmatically without human intervention. This integration is essential for a single-founder project scaling through AI augmentation.

5. **Community and contributor reach:** GitHub has over 100 million developers. Any future contributor (Constitution Law 12) is more likely to have a GitHub account than any other platform. Choosing GitHub reduces the contribution barrier to zero — a potential contributor with a browser and a GitHub account can open an issue or PR without creating yet another account.

---

## 5. Consequences

**Positive:**

- Complete, immutable audit trail of every decision from the first commit
- All ecosystem state can be recovered from git history (disaster recovery through `git clone`)
- Automated validation pipeline (ZUSS checks, cross-reference validation) runs on every PR via GitHub Actions
- AI agents operate on the same git-native content as human contributors
- Plane 0 deployments receive updates via git bundles — no internet required for sync

**Negative:**

- Internet required for remote collaboration and CI/CD execution
- GitHub Actions 2,000 min/month limit requires efficient workflow design (caching, conditional execution)
- GitHub Pages has a 500 MB publishing limit and 100 GB/month bandwidth cap
- If GitHub experiences downtime, PR reviews and CI/CD are blocked (but local development with `git` continues uninterrupted)
- Private repository collaboration limited to 3 users on free tier (all repos are public by design, so this is acceptable)

---

## 6. Status

Accepted. This is the operational foundation of the ecosystem — directly implementing Constitution Law 1. No future decision may contradict this ADR without amending a Tier I constitutional law (which is not permitted).

---

## 7. Cross-references

→ **001-zs-meta/001-zarishsphere-constitution.md** — Constitution Law 1 (GitHub is the government), Law 10 (auditability), Law 12 (borderless contribution)
→ **006-zs-infrastructure/002-github-org-architecture.md** — GitHub organization structure and access
→ **006-zs-infrastructure/006-ci-cd-architecture.md** — GitHub Actions CI/CD pipeline design
→ **008-zs-adrs/007-adr-markdown-first-documentation.md** — ADR-007: markdown-first documentation stored in git
→ **001-zs-meta/004-zarishsphere-writing-rules.md** — ZUSS validation rules enforced in CI on every PR

---

*ZarishSphere Foundation · V1 · August 01, 2026*
*License: Apache 2.0 (code) · CC BY 4.0 (documentation)*
*GitHub: https://github.com/zsdotcom*
