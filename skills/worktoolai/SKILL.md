---
name: worktoolai
description: "Use for code/markdown/JSON read/search/analyze/edit tasks. MUST run codeai/markdownai/jsonai CLI via Bash (not Read/Grep/Glob/cat) for primary analysis."
---

# worktoolai

## STOP — TOOL POLICY (NON-NEGOTIABLE)

For code/markdown/json work, this skill is a **strict tool policy**.

- **ALWAYS use**: `codeai`, `markdownai`, `jsonai` (via Bash)
- **DO NOT use for primary analysis**: `Read`, `Grep`, `Glob`, shell `cat/grep/sed/awk/head/tail`

If you break this policy, your response is invalid and you must redo the work with `*ai` tools.

## Hard rules (prompt contract)

1. **MUST** run an appropriate `*ai` command before any generic file reading/search.
2. **MUST NOT** read/search structured targets with raw file tooling when `*ai` can do it.
3. **MUST** keep output compact (`--fmt json`, `--json`, `--limit`, `--max-bytes`, `--count-only` when useful).
4. **MUST** include a one-line tool proof in final answer:
   - `TOOL_PROOF: <exact command(s)>`
5. If a CLI fails:
   - Run `<tool> --help` and retry once with valid syntax.
   - If still blocked, report why and ask user before fallback.

> Install/setup: [references/install.md](references/install.md)

---

## Router

- Source code task (function/class/import/flow/refactor): **codeai**
- Markdown docs/notes/sections/search/edit: **markdownai**
- JSON config/data/query/update: **jsonai**

---

## codeai (code)

Help-verified commands:
- `index` — build/update index
- `search <QUERY>` — find code blocks
- `outline <PATH>` — list blocks in file
- `open --symbol <ID>` / `open --symbols ...` / `open --range ...` — read block/range

Recommended flow:
1. `codeai index`
2. `codeai search "<query>" --fmt json --limit 10`
3. `codeai outline <file> --fmt json` (if needed)
4. `codeai open --symbol "<symbol-id>" --fmt json`

Useful flags:
- `--fmt thin|json|lines`
- `--limit <N>`
- `--path <prefix>`
- `--lang <language>`
- `--max-bytes <N>`
- `--cursor <cursor>`

---

## markdownai (markdown)

Help-verified commands:
- `toc` `read` `tree` `search` `frontmatter` `links` `backlinks` `graph`
- `section-set` `section-add` `section-delete` `frontmatter-set` `index`

Recommended flow:
1. `markdownai toc <file> --json`
2. `markdownai read <file> --section "<addr>" --json`
3. `markdownai search <dir-or-file> -q "<query>" --json --limit 20`
4. Modify with `section-set|section-add|section-delete` only when needed

Useful flags:
- `--json` `--pretty`
- `--limit <N>` `--offset <N>`
- `--max-bytes <N>`
- `--count-only` `--exists` `--stats`
- `--threshold <N>` `--plan` `--no-overflow`
- `--sync auto|force` `--root <DIR>`

Section addressing:
- TOC index: `"#1.2"`
- Header path: `"## A > ### B"`
- Line range: `"L10-L25"`

---

## jsonai (json)

Help-verified commands:
- `cat` `search` `fields` `query`
- `set` `add` `delete` `patch`

Recommended flow:
1. `jsonai cat <file> --pretty` (or compact by default)
2. `jsonai search -q "<term>" --all <file>`
3. `jsonai query -f '<jq-filter>' <file>`
4. Update with `set|add|delete|patch` as needed

Useful flags:
- `--pretty` `--compact`

---

## Minimal playbooks

### Find function implementation
```bash
codeai index
codeai search "<function-name or behavior>" --fmt json --limit 10
codeai open --symbol "<symbol-id>" --fmt json
```

### Read only one markdown section
```bash
markdownai toc <file> --json
markdownai read <file> --section "#1.3" --json
```

### Extract one JSON value
```bash
jsonai cat <file> --pretty
jsonai query -f '.path.to.value' <file>
```

---

## Response discipline

- Do not dump full files if block/section/value access is enough.
- Prefer targeted reads + small limits.
- State which *ai command was used when summarizing findings.
