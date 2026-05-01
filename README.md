# Your ZAM Personal Kernel

This is your private ZAM repository. Create it from this template, make it yours, and start ZAM from here.

## What lives here

- **`beliefs/`** — Your personal worldview. The ZAM agent proposes changes through conversation; you approve them by committing.
- **`goals/`** — Your personal objectives. High-level goals decompose into tasks and learning tokens.
- **`.zam/config.yaml`** — Non-secret instance configuration: identity, community repos, Turso URL/database name, and connector metadata.

Never commit secrets. Tokens, PATs, and login state belong in machine-local credential stores such as `~/.zam/credentials.json`.

## Multi-repo model

ZAM is intentionally distributed:

| Repo | Typical location | Purpose |
| --- | --- | --- |
| Personal instance | `~/src/zam-YourName` | Private beliefs, goals, identity, and non-secret config |
| Core | `~/src/zam` | `zam-core`: CLI, database layer, scheduler, bridge protocol, skills |
| Community repos | `~/src/zam-dev`, etc. | Shared community config and source package links |

On a fresh machine, read `.zam/config.yaml` first. If `communities` includes a repo such as `github:zam-os/zam-dev`, clone or pull that community repo and follow its `.zam/config.yaml`. Repos marked `link: true` are source packages; install and `npm link` them before trusting the npm package version in this personal repo.

For the developer community, that usually means:

```bash
cd ..
git clone https://github.com/zam-os/zam-dev  # if missing
git clone https://github.com/zam-os/zam      # if missing

cd zam
git pull --ff-only
npm install
npm link

cd ../zam-YourName
npm install
npm link zam-core
npx zam --version
```

If the community repos are already present, pull with `--ff-only` and do not overwrite uncommitted work.

## Getting started

### 1. Create your personal repo from this template

Click **Use this template** on GitHub and create a new **private** repository, e.g. `YourName/zam-YourName`.

```bash
git clone https://github.com/YourName/zam-YourName
cd zam-YourName
```

### 2. Edit `.zam/config.yaml`

Fill in your non-secret configuration:

```yaml
identity:
  user_id: yourname

communities:
  - url: github:zam-os/zam-dev
    role: developer

turso:
  url: libsql://zam-yourname.aws-eu-west-1.turso.io
  db: zam

connectors:
  ado:
    org_url: https://dev.azure.com/yourorg
    project: YourProject
```

Leave sections empty if you do not use them. Commit the config:

```bash
git add .zam/config.yaml
git commit -m "chore: configure instance"
```

### 3. Install dependencies and update linked repos

```bash
npm install
```

If `.zam/config.yaml` has community repos, clone or pull them now and link any source packages as described above.

### 4. Connect the existing cloud database, if configured

If `turso.url` is set, the Turso database is the source of truth for learning tokens, cards, reviews, and sessions. Do not treat a freshly-created local `~/.zam/zam.db` as useful state.

Obtain a database token through the Turso CLI or dashboard, then run:

```bash
npx zam connector setup turso --url "<turso.url from .zam/config.yaml>" --token "<database token>"
npx zam connector sync
npx zam stats --user "<identity.user_id>"
```

The token is stored outside the repo in `~/.zam/credentials.json`.

### 5. Run `/setup` in your agent CLI

Open this repo in Claude Code, Gemini CLI, Copilot CLI, or another agent that understands this repository's instructions, then run `/setup` where available. Setup should install missing tools, distribute skills, set identity, and ask only for missing secrets.

### 6. Start learning

Run `/zam` to begin a learning session.

## Setting up a new machine

The fresh-machine rule is: **repo config first, latest linked repos second, secrets only when missing, cloud stats before local assumptions.**

1. Clone the private personal repo.
2. Read `.zam/config.yaml`.
3. Install dependencies.
4. Clone or pull configured community/source repos and `npm link` source packages.
5. If `turso.url` is configured and `~/.zam/credentials.json` has no token, ask for Turso login/token and run `npx zam connector setup turso --url ... --token ...`.
6. Verify with `npx zam stats --user <user_id>` before registering tokens or starting sessions.

## Supported platforms

| Platform | Package manager |
| --- | --- |
| macOS | Homebrew (`brew`) |
| Windows 11 | WinGet (`winget`) |

## How it works

This repo holds the slow-changing parts of your ZAM experience: beliefs, goals, identity, and non-secret setup config. Git history is your approval trail.

Fast-changing data lives in the ZAM database. By default that can be local SQLite; when `turso.url` is configured, the Turso database is shared across machines and local secrets only enable access to it.

The core learning kernel lives in [`zam-os/zam`](https://github.com/zam-os/zam). It provides the CLI, FSRS scheduler, bridge protocol, and learning science. This repo provides your personal context.

## Beliefs vs. Goals

- **Beliefs** are premises — things you hold to be true. They guide how you interpret the world and how the ZAM agent interacts with you.
- **Goals** are intentions — things you want to achieve. They decompose into tasks, which surface learning opportunities.

Both evolve through conversation with the ZAM agent. Both require your explicit approval through a git commit.

## Updating after a ZAM upgrade

```bash
npm install
npx zam setup --force
git add .claude/skills/zam/ .gemini/skills/zam/ CLAUDE.md
git commit -m "chore: update zam skill files"
```
