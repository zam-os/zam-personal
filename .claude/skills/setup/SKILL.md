---
name: setup
description: First-time setup for a ZAM personal instance. Reads .zam/config.yaml for non-secret config, detects the OS (macOS or Windows), installs tools, handles auth flows, and wires everything up. Run this once after cloning your personal repo.
user-invocable: true
---

# ZAM Setup

You are guiding the user through first-time setup of their ZAM personal instance. Be direct and practical. Run each step, confirm it worked, then move to the next.

---

## What you are setting up

- **This repo** (`beliefs/`, `goals/`) — the slow layer. Tracked in git. Changes require a commit to take effect.
- **`~/.zam/zam.db`** — the fast layer. Tokens, cards, review history, sessions. Not committed to git.
- **Skill files** — copied from the `zam` npm package into `.claude/skills/zam/` and `.gemini/skills/zam/`. These unlock the `/zam` learning agent.

---

## Step 0 — Read instance config

Read `.zam/config.yaml` in the repo root. This file contains non-secret instance configuration. Extract:
- `identity.user_id` — the ZAM username
- `turso.url` — Turso database URL (empty if no cloud sync)
- `turso.db` — Turso database name (for CLI token creation)
- `connectors.ado.org_url` — Azure DevOps org (empty if not used)
- `connectors.ado.project` — ADO project name

If any required value is empty, you will prompt the user for it at the relevant step.

## Step 1 — Detect platform

```bash
uname -s
```

Determine the platform:
- **Darwin** → macOS (Apple Silicon). Package manager: `brew`.
- **MINGW***, **MSYS***, **CYGWIN***, or if `uname` fails → Windows. Package manager: `winget`.

Remember the platform for all subsequent install commands.

## Step 2 — Check Node.js

```bash
node --version
```

Must be v18 or later. If missing or too old, install it:

| Platform | Command |
|----------|---------|
| macOS | `brew install node` |
| Windows | `winget install OpenJS.NodeJS` |

## Step 3 — Install dependencies

```bash
npm install
```

This installs the `zam` CLI into `node_modules/`. Confirm with:

```bash
npx zam --version
```

## Step 4 — Distribute skills and initialize DB

```bash
npx zam setup
```

This copies skill files into `.claude/skills/zam/` and `.gemini/skills/zam/`, initializes `~/.zam/zam.db`, and generates `CLAUDE.md`.

If skill files already exist and need updating, run with `--force`:
```bash
npx zam setup --force
```

## Step 5 — Set identity

Use `identity.user_id` from config. If empty, ask the user:
> "What username would you like to use? (lowercase, no spaces — e.g. your first name)"

```bash
npx zam whoami --set <user_id>
```

## Step 6 — Turso cloud sync

**Skip this step if `turso.url` in config is empty and the user doesn't want cloud sync.**

The goal of this step is to obtain a Turso auth token, then hand off to `zam connector setup turso` which stores the credentials and verifies the connection interactively.

Have the Turso database URL from config ready: `<turso.url>`

### 6a — Get a Turso auth token

The method depends on the platform and what's available.

---

**macOS:**

Check if Turso CLI is installed:
```bash
turso --version
```

If missing:
```bash
brew install tursodatabase/tap/turso
```

Authenticate:
```bash
turso auth login
```

This opens a browser. Tell the user:
> "Please complete the Turso login in your browser — let me know when it's done."

**Wait for the user to confirm before continuing.**

Create a token using `turso.db` from config:
```bash
turso db tokens create <db>
```

Capture the output — this is the token.

---

**Windows — check for WSL first:**

```bash
wsl --version
```

**If WSL is available** (command succeeds):

Install Turso CLI inside WSL:
```bash
wsl -- curl -sSfL https://get.tur.so/install.sh | bash
```

Authenticate inside WSL:
```bash
wsl -- turso auth login
```

This opens a browser. Tell the user:
> "Please complete the Turso login in your browser — let me know when it's done."

**Wait for the user to confirm before continuing.**

Create a token inside WSL using `turso.db` from config:
```bash
wsl -- /root/.turso/bin/turso db tokens create <db>
```

Capture the output — this is the token.

**If WSL is not installed** (command fails):

Offer two options:

> **Option A — Install WSL** (requires admin rights and a reboot):
> Run in an admin PowerShell: `wsl --install`
> After reboot and WSL first-run setup, re-open this repo and run `/setup` again.

> **Option B — Get the token another way** (no WSL needed):
> - From the **Turso web dashboard**: go to app.turso.tech → Databases → `<db>` → "Generate Token"
> - **Or from another machine** where the Turso CLI is already installed: `turso db tokens create <db>`
>
> Once you have the token, let me know and I'll continue.

Ask the user which they prefer and wait for them to either reboot (Option A) or provide the token (Option B).

---

### 6b — Connect ZAM to Turso

Once you have the token, run the interactive connector setup:

```bash
npx zam connector setup turso
```

When prompted:
- **URL**: paste `turso.url` from config
- **Token**: paste the token from Step 6a

### 6c — Verify

```bash
npx zam stats --user <user_id>
```

You should see existing card counts and review history. If it shows zeros, the Turso connection may be pointing to a different database — double-check the URL.

## Step 7 — Azure DevOps connector

**Skip this step if `connectors.ado.org_url` in config is empty and the user doesn't use ADO.**

If ADO config values are present, store the non-secret parts and prompt for the PAT:

```bash
npx zam settings set --key ado.org_url --value "<org_url from config>"
npx zam settings set --key ado.project --value "<project from config>"
```

Then prompt the user for their PAT:
> "Please provide your Azure DevOps Personal Access Token. It needs Work Items (Read) scope. You can create one at {org_url}/_usersSettings/tokens"

```bash
npx zam settings set --key ado.pat --value "<user-provided PAT>"
```

Verify:
```bash
npx zam connector tasks
```

## Step 8 — Set goals directory

```bash
npx zam settings set --key personal.goals_dir --value "$(pwd)/goals"
```

## Step 9 — Commit setup artifacts

```bash
git add .claude/skills/zam/ .gemini/skills/zam/ CLAUDE.md
git commit -m "chore: distribute zam skill files"
```

## Step 10 — Done

Tell the user:

> "Setup is complete. Run `/zam` to start a learning session."

---

## Updating after a zam upgrade

```bash
npm install
npx zam setup --force
git add .claude/skills/zam/ .gemini/skills/zam/
git commit -m "chore: update zam skill files"
```
