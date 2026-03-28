# Your ZAM Personal Kernel

This is your personal ZAM repository. Create it from this template, make it yours, and start ZAM from here.

## What lives here

- **`beliefs/`** — Your personal worldview. What you hold to be true. The ZAM agent will propose changes through conversation; you approve them by committing.
- **`goals/`** — Your personal objectives. High-level goals decompose into tasks and learning tokens. Maintained collaboratively with the ZAM agent.

## Getting started

### 1. Create your personal repo from this template

Click **"Use this template"** on GitHub and create a new **private** repository, e.g. `YourName/zam-YourName`.

Clone it:
```bash
git clone https://github.com/YourName/zam-YourName
cd zam-YourName
```

### 2. Open in Claude Code or Gemini CLI

```bash
claude  # or: gemini
```

### 3. Run `/setup`

The `/setup` skill guides you through the rest interactively:
- Installs the `zam` CLI (`npm install`)
- Distributes skill files into `.claude/skills/zam/` and `.gemini/skills/zam/` (`zam setup`)
- Initializes your learning database at `~/.zam/zam.db`
- Sets your identity (`zam whoami --set <your-id>`)
- Optionally connects to Azure DevOps and/or Turso cloud sync

After setup, commit the distributed skill files:
```bash
git add .claude/skills/zam/ .gemini/skills/zam/
git commit -m "chore: distribute zam skill files"
```

### 4. Start learning

Run `/zam` to begin a learning session on any task you are working on.

## How it works

Your personal repo holds the slow-changing parts of your ZAM experience: beliefs and goals. These are markdown files tracked in git. The git history is your approval trail — every committed change represents a conscious decision.

Fast-changing data (learning tokens, cards, review history, sessions) lives in the ZAM database at `~/.zam/zam.db`.

The core learning kernel lives in [`zam-os/zam`](https://github.com/zam-os/zam). It provides the CLI, the FSRS scheduler, the bridge protocol, and all the learning science. This repo provides *your* context.

## Beliefs vs. Goals

- **Beliefs** are premises — things you hold to be true. They guide how you interpret the world and how the ZAM agent interacts with you.
- **Goals** are intentions — things you want to achieve. They decompose into tasks, which surface learning opportunities.

Both evolve through conversation with the ZAM agent. Both require your explicit approval (a git commit) to take effect.

## Updating skill files

When a new version of `zam` is published, refresh your distributed skill files:

```bash
npm install
npx zam setup --force
git add .claude/skills/zam/ .gemini/skills/zam/
git commit -m "chore: update zam skill files to vX.Y.Z"
```

## Skill distribution model

Skill files belong to the packages that define them. The `zam` package ships `.claude/skills/zam/SKILL.md` and `.gemini/skills/zam/SKILL.md` as part of its npm tarball. `zam setup` copies them into this repo so Claude Code and Gemini CLI can find them.

Future skill groups from other packages follow the same pattern: install the package, run its setup command, commit the distributed files.
