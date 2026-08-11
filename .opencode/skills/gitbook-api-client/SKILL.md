---
name: gitbook-api-client
description: Query the live, published ZarishSphere GitBook site through GitBook's REST "Ask" endpoint or the GitBook MCP server, using the project's configured API token. Use when Ariful wants to check what's actually live on GitBook, ask a question against the published docs, or verify a sync landed correctly — never for writing content (that stays Git Sync's job).
---

# GitBook API client

## Identity for this project
- Organization ID: `PFfkFFYv84Q0VSVmBl06`
- Site ID: `site_Ter8o`
- Auth: personal access token, read from the `GITBOOK_API_TOKEN` environment
  variable — **never hardcode the token value in any file in this
  workspace.** See `GETTING-STARTED.md` §8 for how it's set.

## Two ways to talk to GitBook from here

### 1. GitBook MCP (already configured, `enabled: true` in `opencode.json`)
Since `oauth: false` is set with a bearer header, opencode authenticates
using `GITBOOK_API_TOKEN` automatically once that variable is exported in
your shell before launching opencode. Inside an opencode session you can
just ask, e.g. "ask GitBook MCP whether the Foundation section is published"
— no manual curl needed.

### 2. Raw REST "Ask" endpoint (for scripts, or outside opencode)
Useful for a quick one-off check without opening opencode at all.

```bash
# 1. Make sure GITBOOK_API_TOKEN is exported in your current shell
echo "${GITBOOK_API_TOKEN:+token is set}"

# 2. Ask a question against the published site
curl -s -X POST \
  "https://api.gitbook.com/v1/orgs/PFfkFFYv84Q0VSVmBl06/sites/site_Ter8o/ask" \
  -H "Authorization: Bearer ${GITBOOK_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"question": "How do I get started?", "scope": {"mode": "default"}}'
```

Python (using `requests`, no extra GitBook SDK needed):
```python
import os, requests

response = requests.post(
    "https://api.gitbook.com/v1/orgs/PFfkFFYv84Q0VSVmBl06/sites/site_Ter8o/ask",
    headers={
        "Authorization": f"Bearer {os.environ['GITBOOK_API_TOKEN']}",
        "Content-Type": "application/json",
    },
    json={"question": "How do I get started?", "scope": {"mode": "default"}},
)
print(response.json())
```

## What this is NOT for
- Not for publishing or editing content — that flow is Git Sync
  (`gitbook-zarishsphere/` → commit → push → GitBook syncs). Using the API to
  write pages directly would create the exact conflicting-write problem
  `AGENTS.md` §4 warns about.
- `@gitbook/cli` (`npm install -g @gitbook/cli`, commands like `gitbook new` /
  `gitbook publish`) is a **different tool** for building GitBook
  *Integrations* (apps that extend the GitBook product itself) — it is not
  part of this documentation-publishing workflow and this project does not
  need it. Don't install it unless a future task is actually "build a GitBook
  integration."

## Hard constraints
- Never print, log, or write the literal value of `GITBOOK_API_TOKEN` into
  any file, commit message, or published page.
- If a tool call using this token fails with 401/403, tell Ariful to re-check
  the export in his shell rather than trying alternate tokens or hardcoding
  one.
