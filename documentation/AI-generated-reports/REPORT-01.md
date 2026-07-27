# Code Review Report 01

**Scope:** `git diff HEAD` — staged/working-tree changes on `main` prior to commit.
**Date:** 2026-07-27

## Summary

This diff is primarily repo/tooling cleanup: rebranding fork references (`filipstrand/mflux` → `mflux-community/mflux-fork`), consolidating agent skill files into a shared `skills/` directory with symlinks for `.claude/` and `.cursor/`, converting `.cursor/rules/RULE.md` into a symlink pointing at a new canonical `AGENTS.md`, and a couple of `.gitignore`/doc tweaks. No application/model code changed except the one-line `github_repo` default in `release_manager.py`.

Verified as consistent (no issues): symlinks resolve correctly, no stale `filipstrand` references remain, `pyproject.toml` URLs are fully updated, `README.md` badge target (`tests.yml`) exists in the fork, and the `.gitignore` `*.lock` wording change is inert since `uv.lock` is already git-tracked.

## Findings

### 1. Release script would target upstream's PyPI package name (Medium)

**File:** `src/mflux/release/release_manager.py:17`

```python
github_repo: str = "mflux-community/mflux-fork",
package_name: str = "mflux",
```

`github_repo` is repointed to the fork, but `package_name` still defaults to `"mflux"` — the upstream PyPI project name. `release_script.py` (run by `.github/workflows/release.yml`) calls `create_release()` without overriding `package_name`, so a release run on this fork would check/publish against the real upstream PyPI project using the fork's `PYPI_API_TOKEN`. Best case: auth failure (treated as non-critical, swallowed). Worst case: it actually has access and publishes under the canonical `mflux` name.

**Suggested fix:** decide whether this fork should publish to PyPI at all; if so, use a distinct package name or gate PyPI publishing off.

**Status:** `package_name` default changed to `"mflux2"`. Note: this only affects the PyPI existence-check URL — the actual `twine upload` uploads whatever `uv build` produces, which is still named from `pyproject.toml`'s `[project] name = "mflux"`. To fully stop this fork's release workflow from touching the real upstream PyPI project, `pyproject.toml`'s package `name` also needs to change (or PyPI publishing needs to be disabled/gated for this fork).

### 2. `AGENTS.md` still references a Cursor-only path (Low)

**File:** `AGENTS.md:60`

```
- **Plan Mode Enforcement**: ... save your plan to `.cursor/plans/YYYY-MM-DD-feature-name.md` ...
```

`documentation/USING-CODING-AGENTS.md` (added here) declares `AGENTS.md` canonical and tool-agnostic ("Cursor, Claude Code, etc."). This diff strips "Cursor" from the doc title and from `skills/mflux-model-porting/SKILL.md`, but line 60 still directs every agent to `.cursor/plans/` — inconsistent with the rest of the de-Cursor-ifying pass.

**Suggested fix:** point at a tool-neutral location or scope the bullet explicitly to Cursor.

**Status:** Fixed — plans now save to `documentation/plans/YYYY-MM-DD-feature-name.md`.

### 3. Duplicate `.venv` entry in `.gitignore` (Cosmetic)

**File:** `.gitignore:4` (new) and `:11` (pre-existing)

`.venv` now listed twice — harmless, but dead duplication.

**Suggested fix:** remove one entry.

**Status:** Not fixed (out of scope — `.gitignore` changes predate this pass and weren't part of the requested fixes).

## Not flagged (checked and ruled out)

- `uv.lock` / `*.lock` gitignore wording fix — inert, `uv.lock` is already tracked.
- README CI badge — `tests.yml` exists in this fork.
- `pyproject.toml` — `homepage` is the only URL, correctly points at the fork.
- All three symlinks resolve correctly.
- No CLAUDE.md in this repo, so no conventions findings apply.
