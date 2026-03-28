# Your ZAM Personal Kernel

This is your personal ZAM repository. Create it from this template, make it yours, and start ZAM from here.

## What lives here

- **`beliefs/`** — Your personal worldview. What you hold to be true. The ZAM agent proposes changes through conversation; you approve them by committing.
- **`goals/`** — Your personal objectives. High-level goals decompose into tasks and learning tokens. Maintained collaboratively with the ZAM agent.
- **`.zam/config.yaml`** — Non-secret instance configuration. Turso URL, ADO org/project, identity. Secrets are never stored here — they are obtained via auth flows during setup.

## Getting started

### 1. Create your personal repo from this template

Click **"Use this template"** on GitHub and create a new **private** repository, e.g. `YourName/zam-YourName`.

Clone it:
```bash
git clone https://github.com/YourName/zam-YourName
cd zam-YourName
```

### 2. Edit `.zam/config.yaml`

Fill in your non-secret configuration:

```yaml
identity:
  user_id: yourname

turso:
  url: libsql://zam-yourname.aws-eu-west-1.turso.io
  db: zam

connectors:
  ado:
    org_url: https://dev.azure.com/yourorg
    project: YourProject
```

Leave sections empty if you don't use them. Commit the config:
```bash
git add .zam/config.yaml
git commit -m "chore: configure instance"
```

### 3. Run `/setup` in Claude Code or Gemini CLI

```bash
claude  # or: gemini
# then: /setup
```

The `/setup` skill reads your config, detects your OS (macOS or Windows), and handles everything:
- Installs Node.js if missing (`brew` on Mac, `winget` on Windows)
- Installs the `zam` CLI (`npm install`)
- Distributes skill files and initializes the database (`zam setup`)
- Sets your identity from config
- Installs and authenticates Turso CLI if cloud sync is configured
- Connects Azure DevOps if configured (prompts for PAT — the only secret)

### 4. Start learning

Run `/zam` to begin a learning session.

## Setting up a new machine

Clone your personal repo on the new machine and run `/setup`. The config is already in `.zam/config.yaml` — setup reads it and handles platform-specific tool installation and auth flows automatically.

```bash
git clone https://github.com/YourName/zam-YourName
cd zam-YourName
claude  # or: gemini
# then: /setup
```

## Supported platforms

| Platform | Package manager | Tested |
|----------|----------------|--------|
| macOS (Apple Silicon) | Homebrew (`brew`) | Yes |
| Windows 11 | WinGet (`winget`) | Yes |

## How it works

Your personal repo holds the slow-changing parts of your ZAM experience: beliefs and goals. These are markdown files tracked in git. The git history is your approval trail — every committed change represents a conscious decision.

Fast-changing data (learning tokens, cards, review history, sessions) lives in the ZAM database at `~/.zam/zam.db`. With Turso cloud sync enabled, this data is accessible from any machine.

The core learning kernel lives in [`zam-os/zam`](https://github.com/zam-os/zam). It provides the CLI, the FSRS scheduler, the bridge protocol, and all the learning science. This repo provides *your* context.

## Beliefs vs. Goals

- **Beliefs** are premises — things you hold to be true. They guide how you interpret the world and how the ZAM agent interacts with you.
- **Goals** are intentions — things you want to achieve. They decompose into tasks, which surface learning opportunities.

Both evolve through conversation with the ZAM agent. Both require your explicit approval (a git commit) to take effect.

## Updating after a zam upgrade

```bash
npm install
npx zam setup --force
git add .claude/skills/zam/ .gemini/skills/zam/
git commit -m "chore: update zam skill files"
```
