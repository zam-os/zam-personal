# Copilot instructions for ZAM personal repos

This repository is a private ZAM personal instance created from `zam-os/zam-personal`. Treat it as the slow, git-approved layer: beliefs, goals, identity, and non-secret configuration live here. Learning tokens, cards, reviews, and sessions live in the ZAM database.

## Bootstrap order on a fresh machine

1. Read `.zam/config.yaml` before running ZAM commands. It contains the user id, configured communities, Turso database URL/name, and non-secret connector metadata.
2. Install dependencies with `npm install` if `node_modules` is missing or stale.
3. If `.zam/config.yaml` has `communities`, clone or pull each community repo as a sibling of this repo. Read each community's `.zam/config.yaml`.
4. For community repos that list source repos with `link: true`, clone or pull those repos, run `npm install && npm link` in the source package, then run `npm link <package-name>` in this personal repo. For `github:zam-os/zam-dev`, the linked package is usually `zam-core` from the sibling `zam` repo.
5. If `turso.url` is set, the existing Turso database is the source of truth. Do not treat an empty local `~/.zam/zam.db` as useful state. If `~/.zam/credentials.json` lacks a Turso token, ask the user for a DB token or guide them through Turso login/token creation.
6. Configure cloud access with `npx zam connector setup turso --url "<turso.url>" --token "<token>"`, then verify with `npx zam stats --user "<identity.user_id>"`.
7. Only after cloud stats work should you register tokens, start sessions, or run `/zam`.

## Multi-repo expectations

ZAM may be spread across several sibling repositories:

- `zam-<Name>`: private personal instance; do not commit secrets.
- `zam`: `zam-core` source package and CLI implementation.
- `zam-dev` or other community repos: shared configuration that tells setup which repos to clone and which packages to link.
- `zam-personal`: public template for private personal instances.

Prefer the latest linked source package over a stale npm package when a configured community indicates a source link. Pull only with safe fast-forward commands, and never overwrite unrelated or uncommitted user work.

## Cloud database rule

When `turso.url` is configured, local machine state is just credentials and tooling. The cloud database contains the real learning history. Secrets belong in `~/.zam/credentials.json`, not in this repository. The repo may contain the non-secret Turso URL and DB name.

## Validation

Use existing commands only:

```bash
npm install
npx zam --version
npx zam connector sync
npx zam stats --user "<identity.user_id>"
```

On Windows, use PowerShell and Windows-style paths.
