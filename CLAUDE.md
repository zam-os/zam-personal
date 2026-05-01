# ZAM Personal Kernel

This is a ZAM personal instance. ZAM builds lasting skills through spaced
repetition during real work — not separate study sessions.

## Agent bootstrap

Before doing ZAM work, read `.zam/config.yaml`. It is the non-secret source of
truth for identity, communities, Turso URL/database name, and connector config.

If `communities` contains entries such as `github:zam-os/zam-dev`, this repo is
part of a multi-repo workspace. Clone or pull those sibling repos first. If a
community config lists repos with `link: true`, install and `npm link` those
source packages before relying on the npm package installed in this repo.

If `turso.url` is set, the existing Turso database is the source of truth for
learning tokens, cards, reviews, and sessions. Do not treat an empty local
`~/.zam/zam.db` as useful state. Ask only for missing secrets, then run:

```bash
npx zam connector setup turso --url "<turso.url from .zam/config.yaml>" --token "<database token>"
npx zam stats --user "<identity.user_id>"
```

Secrets must stay out of the repo. Turso credentials belong in
`~/.zam/credentials.json`.

Run `/setup` to complete first-time setup. It should:
- Install dependencies and distribute skill files (`npx zam setup`)
- Pull/link configured community source repos where needed
- Set identity from `.zam/config.yaml`
- Connect to the configured cloud database when Turso credentials are present
- Ask for missing secrets only

## Regular use

Run `/zam` to start a learning session on whatever you are working on.

## What lives here

- `beliefs/` — your worldview, approved by git commit
- `goals/` — your objectives, decomposed into tasks and learning tokens

The git history is your approval trail. Every committed change represents a
conscious decision you made in conversation with the ZAM agent.

## Fast-changing data

Learning tokens, cards, review history, and sessions live in the ZAM database.
Local SQLite is the fallback. If `.zam/config.yaml` has `turso.url`, use that
cloud database across machines and store only its secret token locally.

## Skill files

After running `zam setup`, you will find:
- `.claude/skills/zam/SKILL.md` — the ZAM learning agent skill for Claude Code
- `.gemini/skills/zam/SKILL.md` — the same for Gemini CLI

These are distributed from the `zam` npm package. To update them after a `zam`
upgrade or linked source update: `npm install && npx zam setup --force`
