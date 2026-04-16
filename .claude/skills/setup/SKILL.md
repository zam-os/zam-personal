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
- **`~/.zam/zam.db`** — the fast layer. Tokens, cards, review history, sessions. Not committed to git.
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

## Step 4 — Distribute skills and initialize DB

```bash
npx zam setup
```

This copies `.claude/skills/zam/` and `.gemini/skills/zam/` from `node_modules/zam-core/`, initializes `~/.zam/zam.db`, and skips CLAUDE.md (already present).

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

## Step 7 — Turso cloud sync

**Skip if `turso.url` is empty.**

The goal: obtain a Turso auth token, then run `zam connector setup turso` interactively.

### macOS

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

> "Please complete the Turso login in your browser — let me know when it's done."

**Wait for user confirmation before continuing.**

Create a token using `turso.db` from config:
```bash
turso db tokens create <db>
```

Capture the output — this is the token.

### Windows — WSL with a full Linux distro

The Turso CLI requires a real Linux distro in WSL (Docker Desktop does not count).

> **IMPORTANT — Git Bash path mangling:** Every `wsl` command must be prefixed
> with `MSYS_NO_PATHCONV=1` to prevent Git Bash from rewriting Unix paths.

**7w-1. Check for a usable WSL distro:**

```bash
MSYS_NO_PATHCONV=1 wsl -l -v
```

Look for a distro like **Ubuntu** (not `docker-desktop`). If none exists:

> Open an admin PowerShell and run: `wsl --install -d Ubuntu`
> Complete the Ubuntu first-run setup (username/password), then re-run `/setup`.

**Stop here if no usable distro is available.**

For the remaining commands, use `-d <distro>` (e.g. `-d Ubuntu`) to target the
correct distro.

**7w-2. Install the Turso CLI:**

```bash
MSYS_NO_PATHCONV=1 wsl -d Ubuntu -- sh -c 'curl -sSfL https://get.tur.so/install.sh | sh'
```

The installer puts the binary at `~/.turso/turso`. Ensure it is executable:

```bash
MSYS_NO_PATHCONV=1 wsl -d Ubuntu -- sh -c 'chmod +x $HOME/.turso/turso && $HOME/.turso/turso --version'
```

**7w-3. Authenticate (headless):**

WSL has no browser, so use headless mode:

```bash
MSYS_NO_PATHCONV=1 wsl -d Ubuntu -- sh -c '$HOME/.turso/turso auth login --headless'
```

This prints a URL. Tell the user:

> "Open this URL in your browser and log in. The page will display a token
> (starts with `turso config set token ...`). Copy the **token value** (the
> long string after `token`) and paste it here."

**Wait for the user to paste the token.**

Then set it:

```bash
MSYS_NO_PATHCONV=1 wsl -d Ubuntu -- sh -c '$HOME/.turso/turso config set token "<pasted token>"'
```

**7w-4. Create a database token:**

```bash
MSYS_NO_PATHCONV=1 wsl -d Ubuntu -- sh -c '$HOME/.turso/turso db tokens create <db>'
```

Capture the output — this is the DB token used in the next step.

### Connect ZAM to Turso

Once you have the token, pass it directly (no interactive prompts):
```bash
npx zam connector setup turso --url "<turso.url from config>" --token "<captured token>"
```

Verify:
```bash
npx zam stats --user <user_id>
```

You should see existing card counts and review history.

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
git add .claude/skills/zam/ .gemini/skills/zam/ CLAUDE.md
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
