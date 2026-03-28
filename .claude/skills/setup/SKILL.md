---
name: setup
description: First-time setup for a ZAM personal instance. Guides through dependency installation, skill distribution, identity configuration, and optional connector setup. Handles both new accounts and existing accounts with a local or cloud database. Run this once after creating your personal repo from the template.
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

## Step 1 — Check Node.js

```bash
node --version
```

Must be v18 or later. If missing, direct the user to nodejs.org.

## Step 2 — Install dependencies

```bash
npm install
```

This installs the `zam` CLI into `node_modules/`. Confirm with:

```bash
npx zam --version
```

## Step 3 — Distribute skills

```bash
npx zam setup --skip-init --skip-claude-md
```

This copies `.claude/skills/zam/SKILL.md` and `.gemini/skills/zam/SKILL.md` from `node_modules/zam/` into this repo.

If you see `skip` instead of `copy`, run with `--force`:
```bash
npx zam setup --skip-init --skip-claude-md --force
```

## Step 4 — New account or existing?

Ask the user:

> "Do you have an existing ZAM account with learning history you want to connect to, or is this a fresh start?"

Branch here:

---

### Branch A — Fresh start (new account)

**4A-1. Initialize the database:**
```bash
npx zam init
```

**4A-2. Set identity:**

Ask: "What username would you like to use? (lowercase, no spaces — e.g. your first name)"

```bash
npx zam whoami --set <chosen-id>
```

**4A-3. Optional — Azure DevOps connector:**

Ask: "Do you use Azure DevOps for work items?"

If yes:
```bash
npx zam connector setup ado
```

**4A-4. Optional — Cloud sync:**

Ask: "Do you want to sync your learning history across machines via Turso?"

If yes:
```bash
npx zam connector setup turso
```

---

### Branch B — Existing account (migrating or connecting)

The user already has ZAM data — either in a local SQLite file on another machine, or already in Turso.

Ask: "Where does your existing data live — in a local database file on another machine, or already in Turso cloud?"

**If local database (migrating to cloud):**

The user needs to export from their old machine first, then connect here.

Tell the user:
> "On your old machine, run: `zam connector setup turso` — this migrates your local `~/.zam/zam.db` to Turso and gives you connection credentials. Then come back here and we will connect this machine to the same Turso database."

Once they have the Turso credentials:

```bash
npx zam connector setup turso
```

Follow the prompts with the existing Turso database URL and auth token.

**If already in Turso:**

```bash
npx zam connector setup turso
```

Use the existing database URL and auth token.

**4B-2. Set identity to match existing account:**

Ask: "What is your ZAM username on the existing account?"

```bash
npx zam whoami --set <existing-id>
```

**4B-3. Verify data is accessible:**

```bash
npx zam stats --user <existing-id>
```

You should see the existing card counts and review history. If it shows zeros, the Turso connection may be pointing to a different database — double-check the URL.

---

## Step 5 — Commit the distributed skill files

```bash
git add .claude/skills/zam/ .gemini/skills/zam/
git commit -m "chore: distribute zam skill files"
```

## Step 6 — Done

Tell the user:

> "Setup is complete. Run `/zam` to start a learning session on whatever you are working on."

---

## Updating skill files after a zam upgrade

When a new version of `zam` is published:

```bash
npm install
npx zam setup --skip-init --skip-claude-md --force
git add .claude/skills/zam/ .gemini/skills/zam/
git commit -m "chore: update zam skill files to vX.Y.Z"
```
