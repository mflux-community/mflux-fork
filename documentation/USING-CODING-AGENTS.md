# Using coding agents in this repo

This project supports multiple coding agents (Cursor, Claude Code, etc.) from a shared, single source of truth.

## Agent instructions

- Canonical instructions: `AGENTS.md` (repo root).
- `.cursor/rules/RULE.md` is a **symlink** to `../../AGENTS.md`.
- Edit only `AGENTS.md`; Cursor reads the same content through the symlink.

## Skills

- Canonical location: `skills/` (repo root).
- `.claude/skills` and `.cursor/skills` are **symlinks** to `../skills`.
- Edit skills only under `skills/`; both tools pick up changes automatically since they read through the symlink.
