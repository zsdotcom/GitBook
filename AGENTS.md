# AGENTS.md — zarishsphere-gitbook workspace

This file is read automatically by opencode every time it runs inside
`/home/codeandbrain/Desktop/zarishsphere-gitbook/` or any sub-folder of it. It is
the single source of truth for how agents must behave here. Do not duplicate
these rules into `opencode.json` — reference this file instead.

## 0. Workspace map

```
zarishsphere-gitbook/
├── AGENTS.md              ← this file (project rules, always loaded)
├── opencode.json           ← project config: agents, permissions, GitBook MCP
├── GETTING-STARTED.md      ← one-time toolchain setup (incl. live API token — see §6)
├── .gitignore              ← exists, but does NOT cover session-ses_*.md yet (see §6)
├── .opencode/
│   ├── agents/              ← subagent definitions: librarian, architect, editor
│   ├── skills/              ← task-specific SKILL.md packs (see §6)
│   └── package.json, node_modules/  ← dev deps: @opencode-ai/plugin + @gitbook/api; not part of the content workflow, leave alone
├── session-ses_*.md         ← opencode session transcripts at root — treat as secrets, not source (see §6)
├── _raw/                    ← SOURCE MATERIAL. Read-only. Never restructured in place.
│   └── docs/001-zs-meta/ … 010-zs-ecosystem/   (128 files, 10 domains)
└── gitbook-zarishsphere/     ← OUTPUT. The actual GitBook Git Sync project.
    ├── README.md, SUMMARY.md, .gitbook.yaml   (real content, not placeholders)
    ├── .env.example, .gitignore
    └── <restructured content lives here>
```

## 1. The one rule that overrides every other instruction

**`_raw/` is a reference corpus, not a draft.** Its files carry no guarantee of
correct structure, numbering, cross-references, or ZUSS compliance. Treat every
file as raw evidence to read, classify, and mine — never as something to edit,
move, rename, delete, or copy verbatim.

1. Agents may `read`, `grep`, and `glob` inside `_raw/` at any time.
2. Agents must never run `edit`, `write`, `rm`, `mv`, or `git` operations that
   change anything inside `_raw/`.
3. All generated output is written only inside `gitbook-zarishsphere/`.
4. Before relying on any fact from `_raw/`, state which source file(s) you drew
   from, so Ariful can verify the mapping.

Folder protection has two layers:

- **Active today:** `opencode.json` keeps `edit` on `ask`, denies `rm *`, and
  every subagent has `edit` restricted. This is the real guard — never bypass it
  by editing a file an agent declined.
- **Applied:** a filesystem lock, per `GETTING-STARTED.md` §1. Verified applied
  as of 2026-08-01 (`_raw/` dirs are `dr-x`, files `r--`). Re-verify if you rely
  on it, and re-lock with `chmod -R a-w _raw` after any temporary
  `chmod -R u+w _raw`.

## 2. Workflow — every restructuring task follows this order

1. **Classify** — `raw-doc-classifier` skill inventories `_raw/docs/**` and
   writes `gitbook-zarishsphere/.internal/raw-inventory.md` (working artifact,
   never linked from SUMMARY.md). The `librarian` agent gathers this read-only;
   the primary agent writes the index (librarian's `edit` is denied).
2. **Design** — the `architect` subagent proposes the README.md + SUMMARY.md
   hierarchy mapping every item to a destination page. It presents the map for
   approval before any file is written.
3. **Restructure & rewrite** — the `editor` subagent writes approved pages into
   `gitbook-zarishsphere/`, applying ZUSS (Section 5) and GitBook conventions
   (Section 4) together. Never copy a `_raw/` paragraph verbatim before checking
   it against current ZUSS + banned-word rules.
4. **Lint** — run `markdownlint-cli2` and `vale` against `gitbook-zarishsphere/`
   and fix every report before considering a page done. **Current-state caveat:**
   `vale` 3.17.0 is installed but has no `.vale.ini` / ZUSS style; `markdownlint-cli2`
   is not installed; no markdownlint config exists. So the lint step cannot run
   yet — report pages as **unverified**, not as linted, until the toolchain is
   set up per `GETTING-STARTED.md` §4.
5. **Audit** — the `zuss-compliance-audit` skill re-checks finished output
   against `004-zarishsphere-writing-rules.md` and reports a pass/fail score per
   document.

Never skip a step — a page reaching step 5 without steps 1–4 done in order is not
complete.

## 3. Standing technical rules (all agents, all tasks)

- **Never assume.** If a fact (version, URL, folder decision, naming choice) is
  not confirmed in `_raw/`, this file, or a live web search, stop and ask Ariful.
- **Verify current versions before writing them anywhere** — check the tool's
  registry/release page, not training memory.
- **Never use `latest` as a Docker image tag** in generated config, examples, or
  ADRs. Pin exact versions, matching `_raw/docs/007-zs-tech-stack/001-tech-stack-master.md`.
- **GUI-first bias.** When a task has both a GUI and a CLI path, name the GUI
  option and prefer it for exploratory work; use CLI for scripted/repeatable work.
- **CLI instructions to Ariful must be exact, numbered, and copy-paste ready.**
- **Offline-first / low-resource.** Ariful's build machine is a Lenovo i3, 8 GB
  RAM, Ubuntu 26.04, on variable-quality mobile broadband. Prefer lightweight
  offline-capable tools; batch web research; never require a live connection to
  lint or build.
- **Banned words** in any published ZarishSphere document: "genuinely",
  "honestly", "straightforward". Grep before finishing any page.
- **No employment/affiliation references.** Never describe Ariful by current or
  past employer or organizational role in generated content.

## 4. GitBook Git Sync conventions the output must follow

`gitbook-zarishsphere/` is a real GitBook Git Sync project directory. Current
state: **not yet `git init`-ed and not yet connected to a GitBook space** —
but the content is real and far along: README.md is a ZUSS Class B landing page,
SUMMARY.md is a 159-line ToC covering all 10 domains, `.gitbook.yaml` has live
redirects, and 130 output pages exist. Connection steps are in
`GETTING-STARTED.md` §7.

- `README.md` — the space's landing page. Once Git Sync is on, edit it only
  from the repository, never from the GitBook web UI (avoids conflicting writes).
- `SUMMARY.md` — the literal table of contents. A file on disk that is not
  linked here will not appear in the space. `##` headings for groups, nested
  bullets for hierarchy:
  ```markdown
  # Summary

  ## Foundation
  * [Foundation Charter](002-zs-foundation/001-foundation-charter.md)
  ```
- **Landing-page convention.** Each domain folder carries a `000-*.md`
  aggregate page (e.g. `003-zs-platform/000-platform-architecture.md`) that
  supersedes the raw `index.md`. New pages should follow this pattern when a
  domain needs an entry-point page.
- `.gitbook.yaml` — defines `root`/`readme`/`summary` and `redirects`. Every
  ZUSS renumbering or page move must add a redirect entry so old links keep
  working (the existing entries map each `NNN-zs-domain/index` to its `000-*`
  page or README).
- Folder names mirror the `_raw/` domain numbering (`001-zs-meta/` …
  `010-zs-ecosystem/`) so the source↔output crosswalk stays traceable.
- Never rely on GitBook's silent SUMMARY inference from folder structure —
  always ship an explicit `SUMMARY.md`.

## 5. ZUSS formatting rules the output must follow

(Full definition: `_raw/docs/001-zs-meta/004-zarishsphere-writing-rules.md` —
read it, don't paraphrase from memory. It is also loaded into every session via
`opencode.json` `instructions`.) Every generated page must carry:

- A mandatory header block (per ZUSS §6.1)
- A 3-digit zero-padded index prefix in its filename
- A lowercase-hyphenated filename
- A Table of Contents, mandatory once the document has more than 5 sections
- A license footer
- The "V1 until launch" version marker where ZUSS calls for one
- No banned words (Section 3)

## 6. Tools, lint state, and credentials

One-time setup is in `GETTING-STARTED.md` (root) — not `gitbook-zarishsphere/README.md`,
which is the GitBook landing page, not a setup guide. See
`.opencode/skills/*/SKILL.md` for when to invoke each skill.

| Tool | Purpose | Current state |
|---|---|---|
| `opencode` | agent runtime for this workflow | Installed |
| `markdownlint-cli2` | Markdown structure linting | **Not installed**; no config |
| `vale` | prose linting | **Installed v3.17.0**; no `.vale.ini` / ZUSS style yet |
| `git` | version control, feeds GitBook Git Sync | Not `git init`-ed yet |
| GitBook web app + Git Sync | publishing target | Free tier; connect per `GETTING-STARTED.md` §7 |
| GitBook MCP (`mcp.gitbook.com`) | query/manage the live site (org `PFfkFFYv84Q0VSVmBl06`, site `site_Ter8o`) | Configured (see below) |
| GitBook CLI (`@gitbook/cli`) | builds GitBook *Integrations* — **not needed for this doc workflow** | Do not install unless building an integration |
| `@gitbook/api` (in `.opencode/`) | typed JS client, usable by scripts | Installed as dev dep |

**GitBook MCP — two configs, don't mix them up.** The `gitbook` MCP is declared
in BOTH places:

- Global `~/.config/opencode/opencode.jsonc` — hardcoded bearer token (the live
  PAT). Its comment explicitly forbids redeclaring a conflicting `gitbook` entry.
- Project `opencode.json` — also declares `gitbook` using
  `{env:GITBOOK_API_TOKEN}`. That syntax resolves only from shell environment
  variables: **opencode does not auto-load `.env`** (confirmed gap, see
  `GETTING-STARTED.md` §5). `GITBOOK_API_TOKEN` must be exported in the shell
  before launching opencode, or the project MCP entry fails to connect.

If `opencode mcp list` shows `gitbook failed` with an SSE/cert error, the mobile
broadband dropped the TLS handshake — check the route with
`curl -s -o /dev/null -w "%{http_code}" https://api.gitbook.com/v1/user` and
retry; prefer the CLI or `@gitbook/api` for scripted access.

**GitBook credentials — treat these as secrets, not source material.**

- The PAT lives machine-locally (`~/.config/opencode/opencode.jsonc`,
  `~/.config/gitbook-nodejs/config.json`) and must never appear in committed
  content.
- **Known plaintext exposure:** the live token is written verbatim inside
  `GETTING-STARTED.md` §5 (a repo file, the `.bashrc` export snippet). It was
  also pasted into a past chat transcript. Ariful should rotate it (revoke in
  GitBook, update the two config files and the bashrc export) — do not spread
  the value further.
- Root `session-ses_*.md` transcripts: the last one (as of 2026-08-01,
  `session-ses_0441.md`) verified **clean** of the token, but treat them all as
  secrets. Never read them as content evidence, and never commit them — the root
  `.gitignore` exists but does not yet cover `session-ses_*.md`; add that glob
  before the first commit.

**Skill notes:** `.opencode/skills/open-api-spec/` is a GitBook-integration
skill for rendering interactive API blocks — unrelated to the doc restructuring
workflow, do not invoke it for restructuring tasks.

## 7. Subagents

| Agent | Mode | Role |
|---|---|---|
| `librarian` | subagent | Read-only. Classifies `_raw/` content, never writes. |
| `architect` | subagent | Proposes SUMMARY.md / folder structure for approval. Read + plan only. |
| `editor` | subagent | Writes/rewrites pages inside `gitbook-zarishsphere/` only. |

Invoke by name, e.g. "use the librarian agent to inventory `003-zs-platform`".
Definitions live in `.opencode/agents/*.md`; the permission restrictions are
enforced in `opencode.json`.
