---
id: master
title: Documentation Master Hub
version: 1.0.0
status: active
created: 2026-04-10
updated: 2026-04-10
owner: ai-zamurai
phase: mvp
tags: [docs, index, ai-agent]
references:
  - 01-context/PROJECT.md
  - 02-design/ARCHITECTURE.md
  - 05-operations/GIT_WORKFLOW.md
changeImpact: high
---

# Documentation Master Hub

Central navigation for `ai-zamurai/phalcon-sqlsrv` documentation. All AI agents and contributors **MUST** read this file first to understand where information lives.

## Quick Navigation

| Folder | Purpose | Primary Documents |
|--------|---------|-------------------|
| `01-context/` | Why this project exists | [PROJECT.md](./01-context/PROJECT.md), [CONSTRAINTS.md](./01-context/CONSTRAINTS.md) |
| `02-design/` | How the system is shaped | [ARCHITECTURE.md](./02-design/ARCHITECTURE.md), [DOMAIN.md](./02-design/DOMAIN.md) |
| `03-implementation/` | Coding rules and patterns | [PATTERNS.md](./03-implementation/PATTERNS.md) |
| `04-quality/` | Test strategy | [TESTING.md](./04-quality/TESTING.md) |
| `05-operations/` | Release and Git workflow | [DEPLOYMENT.md](./05-operations/DEPLOYMENT.md), [GIT_WORKFLOW.md](./05-operations/GIT_WORKFLOW.md) |
| `06-reference/` | Glossary and decision log | [GLOSSARY.md](./06-reference/GLOSSARY.md), [DECISIONS.md](./06-reference/DECISIONS.md) |

## AI Quick Nav (Task → Document)

| When you want to… | Read first |
|-------------------|------------|
| Understand what this package does | `01-context/PROJECT.md` |
| Check PHP / Phalcon version requirements | `01-context/CONSTRAINTS.md` |
| Add a new SQL Server dialect method | `02-design/ARCHITECTURE.md` → `03-implementation/PATTERNS.md` |
| Map a new SQL Server column type | `02-design/DOMAIN.md` |
| Release a new version | `05-operations/DEPLOYMENT.md` + skill `phalcon-release` |
| Verify Phalcon 4/5 compatibility | Skill `phalcon-compat-check` |
| Smoke-test a SQL Server connection | Skill `sqlsrv-smoke-test` |
| Find a term definition | `06-reference/GLOSSARY.md` |
| Understand a past design decision | `06-reference/DECISIONS.md` |

## Source of Truth Order

When documents conflict, trust in this order (top wins):

1. Source code under `Phalcon/` (actual behavior)
2. `composer.json` (dependency contract)
3. `06-reference/DECISIONS.md` (recorded intent)
4. `02-design/ARCHITECTURE.md`
5. Other documents

If a document drifts from source, open an issue and fix the document, not the code.

## Document Update Policy

- Bump `version` in frontmatter per semver (major = breaking concept, minor = new section, patch = fix).
- Update `updated` date in frontmatter.
- Record `changeImpact: high` changes in `06-reference/DECISIONS.md`.

## Related Configuration Files

| File | Audience |
|------|----------|
| `../CLAUDE.md` | Claude Code |
| `../AGENTS.md` | Generic AI agents |
| `../.github/copilot-instructions.md` | GitHub Copilot |
| `../.cursorrules` | Cursor |

These files point back to this hub. Keep them in sync when the hub changes.
