---
name: worktoolai
description: "Use for code/markdown/JSON read/search/analyze/edit tasks. Prefer codeai/markdownai/jsonai CLI via Bash; allow Read/Grep/Glob when they are clearly faster for narrow lookups."
---

# worktoolai

## TOOL POLICY (STRICT DEFAULT, FAST EXCEPTIONS)

For code/markdown/json work, this skill is a strict **default** policy with practical exceptions.

- **Default path**: use `codeai`, `markdownai`, `jsonai` (via Bash)
- **Allowed fast path**: use `Read`, `Grep`, `Glob` when they are clearly faster (targeted lookup/discovery)
- **Never use shell text tools for primary analysis**: `cat/grep/sed/awk/head/tail`

## Hard rules (prompt contract)

1. Choose the fastest valid tool path for the task.
2. Prefer `*ai` CLIs for structured/deep analysis and updates.
3. You MAY use `Read/Grep/Glob` immediately when any of these apply:
   - exact file path or symbol location is already known
   - narrow regex/string lookup across files is needed
   - quick file discovery by pattern is needed
   - `*ai` indexing/startup overhead is higher than expected gain
4. Keep output compact (`--fmt json`, `--json`, `--limit`, `--max-bytes`, `--count-only` when useful).
5. Include one-line proof in final answer:
   - `TOOL_PROOF: <exact command(s) or tool call(s)>`
   - If generic tools were used: `FALLBACK_REASON: <why generic was faster>`
6. If a `*ai` CLI fails:
   - run `<tool> --help` and retry once
   - if still blocked, use `Read/Grep/Glob` with `FALLBACK_REASON`

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

### Search query guide (all tools, avoid agent confusion)

- Treat search as semantic keyword lookup, not regex/signature matching.
- Do **not** use escaped signatures/special chars as the primary query.
  - Bad: `func \(s \*Service\) Update`
  - Good: `service update method`
- Keep query short (2-5 tokens): `<target> <action> <context>`
  - Example: `index incremental generation`
- Narrow with tool flags, not punctuation:
  - `codeai`: `--path`, `--lang`, `--limit`
  - `markdownai`: `--limit`, `--offset`, `--threshold`
  - `jsonai`: use `query -f` for exact paths/filters
- Empty result means "no matches" (not command failure); broaden/normalize and retry.
- Before language-specific queries, verify relevant files exist first.

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
