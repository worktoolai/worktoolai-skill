# worktoolai-skill

Claude Code skill plugin that wraps the worktoolai CLI tools (`codeai`, `markdownai`, `jsonai`).

## Language

All documentation, comments, and commit messages must be written in English.

## Repository Structure

```
skills/worktoolai/
  SKILL.md            # Skill definition loaded by Claude Code
  references/         # Supporting docs (install, etc.)
```

## Editing Rules

- `SKILL.md` is the single source of truth for the skill plugin.
- All commands and flags documented in SKILL.md must be verified against `<tool> --help` before updating.
- Keep SKILL.md concise — agents read it on every invocation; avoid verbose prose.
- Do not add commands or flags that do not exist in the current CLI binaries.

## CLI Binary Locations

- Install path: `~/.worktoolai/bin/{codeai,markdownai,jsonai}`
- Source repos (sibling directories under `~/work/ai/`):
  - `jsonai` — JSON manipulation
  - `markdownai` — Markdown processing
  - `codeai` — Code analysis
- Local build & install: `cargo build --release && cp target/release/<tool> ~/.worktoolai/bin/`

## Release

This repo does not have automated releases. Changes are committed directly to `main`.

## Validation

Before committing SKILL.md changes, verify that every documented command/flag is accurate:
```bash
~/.worktoolai/bin/codeai --help
~/.worktoolai/bin/markdownai --help
~/.worktoolai/bin/jsonai --help
```
