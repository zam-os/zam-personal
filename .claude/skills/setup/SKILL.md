---
name: setup
description: First-time setup for a ZAM personal instance. Reads .zam/config.yaml for non-secret config, detects the OS (macOS or Windows), installs tools, clones community repos, handles auth flows, and wires everything up. Run this once after creating your personal repo from the template.
user-invocable: true
---

# ZAM Setup

You are guiding the user through first-time setup of their ZAM personal instance. Be direct and practical. Run each step, confirm it worked, then move to the next.

---

## What you are setting up

- **This repo** (`beliefs/`, `goals/`) — the slow layer. Tracked in git. Changes require a commit to take effect.
- **ZAM database** — the fast layer. Local SQLite is the fallback; if `turso.url` is configured, the existing Turso DB is the source of truth.
- **`~/.zam/credentials.json`** — machine-local secrets such as Turso tokens. Never commit it.
- **Skill files** — copied from `zam-core` into `.claude/skills/zam/` and `.gemini/skills/zam/`. These unlock the `/zam` learning agent.
- **Community repos** — cloned alongside this repo, linked if they are source packages.

---

## Step 0 — Read instance config

Read `.zam/config.yaml`. Extract:
- `identity.user_id`
- `communities[]` — list of `{url, role}` entries (may be empty)
- `turso.url` and `turso.db`
- `connectors.ado.org_url` and `connectors.ado.project`

## Step 1 — Detect platform

```bash
uname -s
```

- **Darwin** → macOS. Package manager: `brew`.
- **MINGW***, **MSYS***, **CYGWIN***, or failure → Windows. Package manager: `winget`.

## Step 2 — Check Node.js

```bash
node --version
```

Must be v18 or later. If missing:

| Platform | Command |
|----------|---------|
| macOS | `brew install node` |
| Windows | `winget install OpenJS.NodeJS` |

## Step 3 — Install dependencies

```bash
npm install
```

Confirm with:
```bash
npx zam --version
```

## Step 4 — Distribute skills and initialize local fallback DB

```bash
npx zam setup
```

This copies `.claude/skills/zam/` and `.gemini/skills/zam/` from `node_modules/zam-core/` and initializes the local fallback DB if no cloud credentials are configured. If `turso.url` is set, do not treat a new local DB as useful state; connect to Turso in Step 7 and verify cloud stats.

To update existing skill files: `npx zam setup --force`

## Step 5 — Set identity

Use `identity.user_id` from config. If empty, ask:
> "What username would you like to use? (lowercase, no spaces)"

```bash
npx zam whoami --set <user_id>
```

## Step 6 — Clone community repos

**Skip if `communities` in config is empty.**

For each entry in `communities[]`:

**6a. Determine the parent directory** (sibling of this repo):
```bash
dirname $(pwd)
```

**6b. Clone the community repo** if not already present:
```bash
# Convert github:org/repo → https://github.com/org/repo
git clone https://github.com/<org>/<repo> <parent-dir>/<repo>
```

Skip with a message if the directory already exists.

**6c. Read the community's config:**

Read `<community-dir>/.zam/config.yaml`. Extract:
- `repos[]` — list of `{url, description, link}` entries

**6d. Clone each repo listed by the community:**

For each repo in community `repos[]`:
```bash
git clone https://github.com/<org>/<repo> <parent-dir>/<repo>
```

Skip if already present.

**6e. Link source packages:**

For each repo where `link: true`:
```bash
cd <parent-dir>/<repo>
npm install
npm link
```

Then link into this personal instance:
```bash
cd <this-repo>
npm link <package-name>
```

The package name comes from `<repo>/package.json` → `"name"` field.

Confirm the link worked:
```bash
npx zam --version
```

## Step 7 — Turso cloud database

**Skip if `turso.url` is empty.**

The configured Turso database is the source of truth for tokens, cards, review history, and sessions. The repo stores only the non-secret URL/database name. The machine-local token belongs in `~/.zam/credentials.json`.

First check whether credentials already exist:
```bash
npx zam connector sync
```

If this succeeds, continue to verification. If it says Turso is not configured, obtain a database token. Prefer the least invasive path:

- If Turso CLI is already installed and authenticated, run `turso db tokens create <db>`.
- If not authenticated, run `turso auth login`, let the user complete browser login, then run `turso db tokens create <db>`.
- On Windows, do **not** require WSL just to continue setup. If Turso CLI is unavailable, ask the user to generate a DB token from the Turso dashboard or from another machine.

Connect ZAM non-interactively:
```bash
npx zam connector setup turso --url "<turso.url from config>" --token "<captured token>"
```

The token is a secret. Do not echo it back, write it into the repo, or commit it.

Verify:
```bash
npx zam connector sync
npx zam stats --user <user_id>
```

You should see existing card counts and review history. If stats show an empty deck while a cloud DB was expected, stop and investigate the Turso connection instead of registering new tokens.

## Step 8 — Azure DevOps connector

**Skip if `connectors.ado.org_url` is empty.**

```bash
npx zam connector setup ado
```

When prompted, enter the org URL, project name, and PAT from config / user input.

Verify:
```bash
npx zam connector tasks
```

## Step 9 — Set goals directory

```bash
npx zam settings set --key personal.goals_dir --value "$(pwd)/goals"
```

## Step 10 — Commit setup artifacts

```bash
git add .claude/skills/zam/ .gemini/skills/zam/ .github/copilot-instructions.md CLAUDE.md README.md
git commit -m "chore: distribute zam-core skill files"
```

## Step 11 — Done

> "Setup is complete. Run `/zam` to start a learning session."

---

## Updating after a zam-core upgrade

```bash
npm install
npx zam setup --force
git add .claude/skills/zam/ .gemini/skills/zam/
git commit -m "chore: update zam-core skill files"
```
