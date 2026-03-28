# ZAM Personal Kernel

This is a ZAM personal instance. ZAM builds lasting skills through spaced
repetition during real work — not separate study sessions.

## First time here?

Run `/setup` to complete first-time setup. It will:
- Install dependencies and distribute skill files (`zam setup`)
- Initialize the learning database (`~/.zam/zam.db`)
- Walk you through identity, connectors, and your first goals

## Regular use

Run `/zam` to start a learning session on whatever you are working on.

## What lives here

- `beliefs/` — your worldview, approved by git commit
- `goals/` — your objectives, decomposed into tasks and learning tokens

The git history is your approval trail. Every committed change represents a
conscious decision you made in conversation with the ZAM agent.

## Fast-changing data

Learning tokens, cards, review history, and sessions live in `~/.zam/zam.db`
(local SQLite, not committed to git). Use `zam connector setup turso` to
enable cloud sync across machines.

## Skill files

After running `zam setup`, you will find:
- `.claude/skills/zam/SKILL.md` — the ZAM learning agent skill for Claude Code
- `.gemini/skills/zam/SKILL.md` — the same for Gemini CLI

These are distributed from the `zam` npm package. To update them after a `zam`
upgrade: `npm install && zam setup --force`
