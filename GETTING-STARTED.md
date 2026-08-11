# Getting started — zarishsphere-gitbook toolchain

This is the one-time setup for the folder. Everything here is free and open
source, verified against currently published versions as of 2026-08-01.
Re-check versions yourself before pinning anything long-term — tools update
often.

## 0. Where these files go

```
/home/codeandbrain/Desktop/zarishsphere-gitbook/
├── AGENTS.md
├── opencode.json
├── GETTING-STARTED.md
├── .gitignore
├── .opencode/
│   ├── agents/{librarian,architect,editor}.md
│   └── skills/{raw-doc-classifier,gitbook-restructure,zuss-compliance-audit,
│              gitbook-summary-builder,gitbook-api-client}/SKILL.md
├── _raw/                      ← your existing folder, untouched
└── gitbook-zarishsphere/
    ├── README.md
    ├── SUMMARY.md
    ├── .gitbook.yaml
    ├── .gitignore
    └── .env.example
```
This is already in place. If you ever re-deploy the kit elsewhere, copy it
with `cp -r /path/to/kit/. /home/codeandbrain/Desktop/zarishsphere-gitbook/`
then confirm with `tree -a -L 2 /home/codeandbrain/Desktop/zarishsphere-gitbook`.

## 1. Lock `_raw/` before anything else touches it — DONE, confirmed 2026-08-01

```bash
# 1. Make the raw corpus read-only for your user
chmod -R a-w /home/codeandbrain/Desktop/zarishsphere-gitbook/_raw

# 2. Verify (should show r-xr-xr-x style permissions, no w)
ls -ld /home/codeandbrain/Desktop/zarishsphere-gitbook/_raw
```
GUI equivalent: in your file manager, right-click `_raw` → Properties →
Permissions → set "Access" to Read Only for Owner/Group/Others, applied
recursively.

## 2. Install opencode — DONE (v1.18.10 confirmed installed)

```bash
# 3. Install (curl method — fastest on Ubuntu)
curl -fsSL https://opencode.ai/install | bash

# 4. Reload your shell profile
source ~/.bashrc

# 5. Verify
opencode --version
```
Or via npm (needs Node.js already installed):
```bash
# 3-alt. Install (npm method)
npm i -g opencode-ai@latest
```
GUI equivalent: `opencode-desktop` is available for Linux via `.deb`/`.rpm`
downloads from `opencode.ai/download` if you'd rather not use the terminal
UI at all.

## 3. Connect a model provider

```bash
# 6. Start the login flow (choose your provider interactively)
opencode auth login
```
Credentials land in `~/.local/share/opencode/auth.json` — never committed to
the repo.

## 4. Install the linting toolchain (free, open source)

```bash
# 7. Node-based Markdown structure linter — DONE (v0.23.2 confirmed installed)
npm i -g markdownlint-cli2
```

**Two more corrections, confirmed on your machine:**
1. The earlier `curl ... install.sh` command was wrong and failed silently
   (curl's `-f` flag fails quietly on a 404, so `sh` got empty input and
   installed nothing). There is no official `install.sh` for vale.
2. `sudo snap install vale` doesn't work either — this system has no
   `snapd`, and `vale` isn't in Ubuntu's `apt` repos.
3. The project also moved its GitHub org from `errata-ai` to `vale-cli` —
   using the canonical URL below instead of relying on a redirect.

Use the manual binary install instead — this fetches whatever release is
currently latest via the GitHub API, no hardcoded version:

```bash
# 8. Download, verify, and install the current release
VALE_VER=$(curl -s https://api.github.com/repos/vale-cli/vale/releases/latest | grep -oP '"tag_name": "\K(.*)(?=")')
echo "Latest vale release: ${VALE_VER}"
curl -sL "https://github.com/vale-cli/vale/releases/download/${VALE_VER}/vale_${VALE_VER#v}_Linux_64-bit.tar.gz" -o /tmp/vale.tar.gz
sudo mkdir -p /usr/local/bin
sudo tar -xzf /tmp/vale.tar.gz -C /usr/local/bin vale
rm /tmp/vale.tar.gz

# 9. Verify both
markdownlint-cli2 --version
vale --version
```
No `sudo` available? Swap the destination to `~/.local/bin` instead of
`/usr/local/bin` (`mkdir -p ~/.local/bin`, confirm it's on your `PATH`), and
drop the two `sudo` prefixes.

## 5. GitBook API credentials — set once, use everywhere

This project has a real GitBook API token, organization, and site already
provisioned:

| Thing | Value |
|---|---|
| Organization | zarishsphere (ID `PFfkFFYv84Q0VSVmBl06`) |
| Site | zsdocs (ID `site_Ter8o`) |
| Token | a `gb_api_…` personal access token from GitBook Developer settings |

**opencode does not auto-load `.env` files as of the current release** (that
`{env:VAR}` syntax in `opencode.json` only resolves from variables already in
your shell's process environment — this is a confirmed, open gap, not
something I'm inferring). So the token has to be exported before you launch
opencode, not dropped in a `.env` file and forgotten:

```bash
# 10. Add the export to your shell profile (do this once)
cat >> ~/.bashrc << 'EOF'
export GITBOOK_API_TOKEN="gb_api_2mAh1OudyoEsip9yxq4Thlf8C4xzC8GV5QvKYLiS"
export GITBOOK_ORG_ID="PFfkFFYv84Q0VSVmBl06"
export GITBOOK_SITE_ID="site_Ter8o"
EOF

# 11. Reload
source ~/.bashrc

# 12. Confirm it's set (won't print the value)
echo "${GITBOOK_API_TOKEN:+GITBOOK_API_TOKEN is set}"
```

**Security note:** this token was pasted into a chat log, which is not a
secure storage channel. Treat it as already partly exposed — consider
generating a fresh token in GitBook Developer settings and revoking this one
once your workflow is confirmed working, rather than continuing to rely on
a token that passed through a conversation transcript. Never add the raw
value to any file inside `gitbook-zarishsphere/` — both `.gitignore` files
in this kit already exclude `.env*`, and `.env.example` in
`gitbook-zarishsphere/` only has a placeholder.

`opencode.json` at the project root now has GitBook MCP enabled and pointed
at `https://mcp.gitbook.com/mcp` using `{env:GITBOOK_API_TOKEN}` — no token
value lives in that file.

## 6. First opencode run

```bash
# 13. From inside the project folder
cd /home/codeandbrain/Desktop/zarishsphere-gitbook

# 14. Launch the interactive TUI — it auto-loads AGENTS.md and opencode.json
opencode

# 15. In a separate terminal, confirm the GitBook MCP connected
opencode mcp list
```
If `opencode mcp list` shows `gitbook` as failed rather than connected, the
first things to check are (a) `GITBOOK_API_TOKEN` is actually exported in
the same shell opencode was launched from, and (b) basic connectivity:
```bash
# 16. Quick reachability check
curl -s -o /dev/null -w "%{http_code}\n" https://api.gitbook.com/v1/user \
  -H "Authorization: Bearer ${GITBOOK_API_TOKEN}"
```
A `200` means the token and network path are both fine and the issue is
opencode-side; anything else means check the token or the network first.

Inside the opencode session, start with:
```
use the librarian agent to inventory _raw/docs/001-zs-meta
```
then progress domain by domain, followed by the `architect` agent to design
`SUMMARY.md`, and only then the `editor` agent to write pages — the full
sequence is in `AGENTS.md` §2, which opencode reads automatically.

## 7. GitBook — publishing target (GUI, free tier)

Publishing content is a Git Sync flow through the GitBook web app, not the
API — this is the intentional GUI-first half of the workflow:

1. Sign in at `gitbook.com` under the `zarishsphere` organization.
2. Open the `zsdocs` site's space settings → **Git Sync**, connect GitHub,
   and set the **Project directory** to wherever `gitbook-zarishsphere/`
   lives in the target repo.
3. Push a first commit containing `README.md`, `SUMMARY.md`, and
   `.gitbook.yaml` — GitBook picks them up on the next sync.
4. From then on, edit content only through the repository (via opencode or
   by hand) — never through the GitBook web editor once sync is on, to avoid
   conflicting writes.

### Two different ways to reach GitBook — don't mix them up

- **Git Sync** (above): how content is published. One-way source of truth:
  this repo → GitBook.
- **GitBook MCP / REST Ask API** (`.opencode/skills/gitbook-api-client/SKILL.md`):
  read-mostly querying of the *live* site — e.g. "is the Foundation section
  actually published", or asking a question against published docs. Not for
  writing content.
- **`@gitbook/cli`** (`npm install -g @gitbook/cli`): a separate tool for
  building GitBook *Integrations* (apps that extend GitBook itself, via
  `gitbook new` / `gitbook publish`). This project doesn't need it — it's
  not part of the documentation-publishing workflow. Don't install it unless
  a future task is specifically "build a GitBook integration."

## 8. Sanity checklist

```bash
# 17. Confirm _raw is locked
test -w /home/codeandbrain/Desktop/zarishsphere-gitbook/_raw && echo "WARNING: _raw is writable" || echo "OK: _raw is read-only"

# 18. Confirm opencode.json is valid JSON
cd /home/codeandbrain/Desktop/zarishsphere-gitbook && python3 -m json.tool opencode.json > /dev/null && echo "OK: opencode.json is valid JSON"

# 19. Confirm the token is exported and reachable
echo "${GITBOOK_API_TOKEN:+token set}" && curl -s -o /dev/null -w "%{http_code}\n" https://api.gitbook.com/v1/user -H "Authorization: Bearer ${GITBOOK_API_TOKEN}"
```
