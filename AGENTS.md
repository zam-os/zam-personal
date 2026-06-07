# ZAM Personal Kernel

This is a ZAM personal instance. Beliefs, goals, identity, and non-secret
configuration are the slow, git-approved layer. Learning tokens, cards, reviews,
and sessions live in the configured ZAM database.

## First Time Here

Invoke `$setup` or select `setup` through `/skills`. The setup workflow reads
`.zam/config.yaml`, installs dependencies, distributes the core ZAM skills,
configures identity and connectors, and clones configured community repositories.

## Regular Use

Invoke `$zam` or select `zam` through `/skills` to start a learning session.
Codex repository skills live under `.agents/skills/`; they are not custom slash
commands.

## Repository Contents

- `beliefs/` contains the user's worldview, approved by git commit.
- `goals/` contains objectives decomposed into tasks and learning tokens.
- `.zam/config.yaml` contains non-secret instance configuration.
- `~/.zam/credentials.json` contains machine-local secrets and must never be committed.

Run `npm install && npx zam setup --force` after upgrading `zam-core` to refresh
the distributed skills.
