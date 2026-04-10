---
id: deployment
title: Release & Deployment
version: 1.1.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [release, packagist, composer]
references:
  - ../06-reference/DECISIONS.md
  - GIT_WORKFLOW.md
changeImpact: medium
---

# Release & Deployment

This package is distributed as a Composer library on Packagist. There is no server-side deployment.

## Release Channel

- Package: `ai-zamurai/phalcon-sqlsrv`
- Source: `https://github.com/ai-zamurai/phalcon-sqlsrv`
- Registry: Packagist (auto-updates from GitHub webhook)

Initial registration on Packagist is **manual and one-time** per package name. Submit the repository URL once at https://packagist.org/packages/submit; Packagist auto-configures a `push`-event webhook on success, and every subsequent tag push is picked up automatically (~1 minute). Confirm the package is registered with `curl -sS -o /dev/null -w "%{http_code}\n" "https://repo.packagist.org/p2/ai-zamurai/phalcon-sqlsrv.json"` — a `200` response means the package is registered on Packagist (`404` means initial submission is still required).

## Versioning

Semantic versioning (`MAJOR.MINOR.PATCH`):

- **MAJOR** — Breaking API change, Phalcon major version bump.
- **MINOR** — New SQL generation capability, new column type mapping, new configuration option.
- **PATCH** — Bug fix, docs, safe refactor.

Always tag with a leading `v` (e.g. `v1.2.3`).

## Pre-Release Checklist

1. All changes merged into `develop` via PR.
2. `composer validate` passes.
3. Manual smoke test (see `04-quality/TESTING.md`) passes against a real SQL Server instance.
4. `DECISIONS.md` updated if the change alters observable behavior.
5. Frontmatter `version` bumps applied to any edited docs.
6. Packagist registration confirmed (see [Release Channel](#release-channel) above) — required once per new package name.

## Release Steps

The `phalcon-release` skill automates most of this. Manual fallback:

```bash
# 1. Merge develop → master
git checkout master
git pull origin master
git merge --no-ff develop -m "chore: release vX.Y.Z"

# 2. Tag
git tag -a vX.Y.Z -m "Release vX.Y.Z"

# 3. Push
git push origin master
git push origin vX.Y.Z
```

Packagist picks up the new tag via webhook within ~1 minute.

## Rollback

If a tag turns out to be broken:

1. Do **not** delete the tag from GitHub (Packagist caches it, and deletion confuses dependents).
2. Instead, release a new patch (`vX.Y.Z+1`) that reverts or fixes the issue.
3. Mark the broken version in `DECISIONS.md` with an `avoid: vX.Y.Z` note.

## Distribution Verification

After release, run the canonical verification recipe in the [`phalcon-release` skill](../../.claude/skills/phalcon-release/SKILL.md) Step 5. The skill uses `ReflectionClass` so it succeeds silently — no spurious connect-time error to interpret.

If the autoloader cannot find the class, the release is broken — issue a patch.

## Out of Scope for Release

- Docker images
- Language bindings other than PHP
- Compiled Phalcon extension (`ext-phalcon`) — users install it separately
