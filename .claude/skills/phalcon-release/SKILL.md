---
name: phalcon-release
description: Tag, push, and release a new version of ai-zamurai/phalcon-sqlsrv to Packagist. Use when the user says "release", "cut a version", "tag vX.Y.Z", or asks to publish to Packagist.
---

# Phalcon Release

Step-by-step release workflow for `ai-zamurai/phalcon-sqlsrv`. Follow exactly — do not skip the verification step.

## Prerequisites (check before running)

- Current branch is clean (`git status` shows nothing).
- `develop` is up to date with `origin/develop`.
- `docs/06-reference/DECISIONS.md` reflects any behavior change in this release.
- The target version has been agreed with the maintainer.
- **Package is registered on Packagist.** Verify with:
  ```bash
  curl -sS -o /dev/null -w "%{http_code}\n" "https://repo.packagist.org/p2/ai-zamurai/phalcon-sqlsrv.json"
  ```
  A `200` response means the package is registered and Step 5 verification will work. If the response is `404`, this is the first release under this package name — you must **manually submit the repository once** at https://packagist.org/packages/submit (enter `https://github.com/ai-zamurai/phalcon-sqlsrv` and click Submit) before completing the release. Packagist auto-configures a GitHub webhook on successful submission, so every subsequent tag push is picked up automatically.

## Step 1: Decide the Version

Ask the user (or infer from changes) which semver level applies:

- **MAJOR** — breaking API change, Phalcon major version bump.
- **MINOR** — new dialect method, new column mapping, new descriptor option.
- **PATCH** — bug fix, docs, refactor with no observable change.

Never guess. If unclear, stop and ask.

## Step 2: Merge develop → master

```bash
git checkout master
git pull origin master
git merge --no-ff develop -m "chore: release vX.Y.Z"
```

If the merge has conflicts, stop and escalate — do not resolve blindly.

## Step 3: Tag

```bash
git tag -a vX.Y.Z -m "Release vX.Y.Z"
```

Always use `vX.Y.Z` (leading `v`). Never tag without the `v` — Packagist parses it.

## Step 4: Push

```bash
git push origin master
git push origin vX.Y.Z
```

## Step 5: Verify Packagist Pickup

Wait ~1 minute, then verify from a clean directory:

```bash
mkdir /tmp/verify-$(date +%s) && cd /tmp/verify-*
composer require ai-zamurai/phalcon-sqlsrv:vX.Y.Z
php -r 'require "vendor/autoload.php"; class_exists("Phalcon\\Db\\Adapter\\Pdo\\Sqlsrv", false) || (new ReflectionClass("Phalcon\\Db\\Adapter\\Pdo\\Sqlsrv"));'
```

The command should exit silently. If it errors with "class not found", the release is broken — issue a patch version and mark the broken tag in `docs/06-reference/DECISIONS.md` as `avoid:`.

## Step 6: Sync develop

```bash
git checkout develop
git merge master
git push origin develop
```

This keeps `develop` ahead of `master` for future work.

## Rollback Protocol

If a tag is broken:

1. **Do not delete the tag from GitHub.** Packagist caches it; deletion confuses dependents.
2. Ship a new patch (`vX.Y.Z+1`) that reverts or fixes the issue.
3. Record in `docs/06-reference/DECISIONS.md`:
   ```
   avoid: vX.Y.Z — reason, see issue #N
   ```

## References

- `docs/05-operations/DEPLOYMENT.md` — full release documentation
- `docs/05-operations/GIT_WORKFLOW.md` — branch and commit conventions
