---
id: git-workflow
title: Git Workflow
version: 1.0.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [workflow, git, process]
references:
  - ../MASTER.md
  - DEPLOYMENT.md
changeImpact: high
---

# Git Workflow

Branching model, commit conventions, and PR rules for this repository. Follow this document for every change — no direct commits to `master` or `develop`.

## Branch Model

```
master   ← release tags only (Packagist source)
  ↑
develop  ← integration branch (default for PRs)
  ↑
<type>/#<issue>-<slug>   ← working branches
```

| Prefix | Use for |
|--------|---------|
| `feature/` | New functionality (new dialect method, new column type) |
| `fix/` | Bug fixes |
| `docs/` | Documentation-only changes |
| `chore/` | Tooling, composer metadata, non-user-visible work |
| `hotfix/` | Emergency patch from `master` (rare) |

Every branch name contains the issue number: `feature/#42-datetime2-mapping`.

## Commit Convention

Use Conventional Commits with the issue number in the header:

```
<type>: #<issue> <short summary>

<optional body — what and why, not how>

Refs: docs/<path>.md
Closes #<issue>
```

Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`.

Example:

```
feat: #42 Map datetime2 to TYPE_DATETIME

SQL Server datetime2 was previously falling through to the VARCHAR
default. Phalcon apps expect these columns to surface as DateTime
instances, which requires TYPE_DATETIME.

Refs: docs/02-design/DOMAIN.md
Closes #42
```

## Standard Flow

1. **Create an issue** first. No silent work. Japanese text is fine for issues and PRs (global rule).
2. **Sync develop**: `git checkout develop && git pull origin develop`.
3. **Branch**: `git checkout -b feature/#42-datetime2-mapping`.
4. **Implement** with atomic commits. Reference docs in commit bodies.
5. **Self-review** before pushing — re-read the diff, run `composer validate`, run relevant smoke tests from `04-quality/TESTING.md`.
6. **Push** and **open a PR** against `develop`.
7. **Address reviews** by pushing new commits. Do not force-push a branch after a human has left a review comment (rebasing before any review is fine).
8. **Merge style.** Default to a merge commit. Use squash only for single-purpose PRs whose intermediate history is noise.
9. **Delete the branch** after merge.

## Pull Request Rules

- Base: `develop` (never `master`).
- Title: same format as the commit header.
- Body must include:
  - Summary of the change
  - Linked issue (`Closes #N`)
  - Manual test notes if the change touches SQL generation or the adapter
- Block merging until:
  - At least one reviewer approves, OR
  - It is a `docs/` branch and the author is a maintainer.

## Release Branches

This repo does not use long-lived `release/*` branches. Releases happen by merging `develop` → `master` and tagging. See `DEPLOYMENT.md`.

## Forbidden Actions

- `git push --force` to `master` or `develop`.
- Committing secrets, `.env` files, or real SQL Server credentials.
- Bypassing PRs by pushing directly to `master`.
- Skipping commit hooks (`--no-verify`) unless a maintainer has told you to.

## Working with Upstream

The repo tracks `upstream` remote (`bakaphp/phalcon-sqlsrv`). When pulling upstream changes:

```bash
git fetch upstream
git checkout develop
git merge upstream/<branch>
```

Resolve conflicts preserving this fork's decisions from `06-reference/DECISIONS.md` (e.g., `TrustServerCertificate=true` defaults).
