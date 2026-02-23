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
- `outline <PATH>` — list blocks in a file
- `open --symbol <ID>` / `open --symbols ...` / `open --range ...` — read block/range
- `graph <PATH>` — show import/dependency graph from entry file
- `project get` — infer entrypoint/shared/orphan project structure

Recommended flow:
1. `codeai index`
2. `codeai search "<query>" --fmt json --limit 10`
3. `codeai open --symbol "<symbol-id>" --fmt json`
4. (Optional) `codeai outline <file> --fmt json` / `codeai graph <file> --fmt thin`

Useful flags (by command):
- Common: `--fmt thin|json|lines` (`graph`: `tree|thin`, `project get`: `thin`), `--max-bytes <N>`
- `index`: `--full`, `--path <prefix>`, `--lang <language>`, `--ignore-file <FILE>`
- `search`: `--limit <N>`, `--path <prefix>`, `--lang <language>`, `--cursor <cursor>`
- `outline`: `--kind <KIND>`, `--limit <N>`, `--cursor <cursor>`
- `open`: `--symbol`, `--symbols`, `--range`, `--preview-lines <N>`, `--offset <N>`
- `graph`: `--depth <N>`, `--limit <N>`, `--offset <N>`, `--external`
- `project get`: `--fmt thin`, `--max-bytes <N>`

### Search query guide (all tools, avoid agent confusion)

- Treat search as semantic keyword lookup, not regex/signature matching.
- Do **not** use escaped signatures/special chars as the primary query.
  - Bad: `func \(s \*Service\) Update`
  - Good: `service update method`
- Keep query short (2-5 tokens): `<target> <action> <context>`
  - Example: `index incremental generation`
- Narrow with tool flags, not punctuation:
  - `codeai`: `--path`, `--lang`, `--limit`
  - `markdownai`: `--limit`, `--offset`, `--match`, `--scope`, `--threshold`
  - `jsonai`: `search --field/--match` or `query -f` for exact paths/filters
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
4. Modify with `section-set|section-add|section-delete|frontmatter-set` when needed

Useful flags (common):
- `--json` `--pretty` `--max-bytes <N>`
- `--limit <N>` `--offset <N>` `--threshold <N>`
- `--count-only` `--exists` `--stats` `--plan` `--no-overflow`
- `--facets <FIELD>` `--sync auto|force` `--root <DIR>`

Command-specific highlights:
- `read`: `--section`, `--summary [N]`, `--meta`
- `search`: `--match text|exact|fuzzy|regex`, `--scope all|body|headers|frontmatter|code`, `--context <N>`, `--bare`
- `tree`: `--depth <N>`, `--files-only`, `--count`
- `links`: `--type wiki|markdown|all`, `--resolved`, `--broken`
- `graph`: `--format adjacency|edges|stats`, `--start <FILE>`, `--depth <N>`, `--orphans`
- Write commands (`section-*`, `frontmatter-set`): `--dry-run`, `--output <FILE>`, `--with-toc` (where supported), `--content-file <FILE>` (set/add)

Shell quoting safety (for `section-add` / `section-set`):
- Prefer `--content-file <FILE>` for multi-line content or content containing backticks (`` ` ``).
- If inline content is required, wrap `--content` with single quotes (`'...'`) or ANSI-C quotes (`$'...'` for `\n`).
- Do **not** use double quotes for `--content` when payload includes backticks; zsh may run command substitution.
- Never leave markdown list bullets (`- item`) unquoted; keep the entire payload as one argument.
- If you see `unexpected argument '- '`, check broken quoting first and retry with `--content-file`.

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
1. `jsonai cat <file> --pretty` (or `--pointer /path/to/node` for subtree)
2. `jsonai search -q "<term>" --all <file>`
3. `jsonai query -f '<jq-filter>' <file>`
4. Update with `set|add|delete|patch` as needed (`--dry-run` first when changing files)

Useful flags (common):
- `--pretty` `--compact`

Command-specific highlights:
- `cat`: `--pointer <JSON-Pointer>`
- `search`: `--field <FIELD>` (repeatable), `--all`, `--match text|exact|fuzzy|regex`, `--output match|hit|value`, `--select <FIELDS>`, `--limit`, `--offset`, `--count-only`, `--bare`, `--max-bytes`, `--threshold`, `--plan`, `--no-overflow`, `--schema <FILE>`
- `fields`: `--schema`
- `query`: `-f|--filter '<jq-filter>'`
- Mutating commands (`set|add|delete|patch`): `--output <FILE>`, `--dry-run` (`set|add|delete` use `--pointer`; `patch` uses `--patch <DOC|->`)

---

## Minimal playbooks

### Find function implementation
```bash
codeai index --path src/
codeai search "<function-name or behavior>" --fmt json --path src/ --limit 10
codeai open --symbol "<symbol-id>" --fmt json --preview-lines 120
```

### Inspect dependency graph from entry file
```bash
codeai graph <entry-file> --fmt thin --depth 2 --limit 20
```

### Infer project structure from index
```bash
codeai project get --fmt thin --max-bytes 12000
```

### Read only one markdown section
```bash
markdownai toc <file> --json --limit 100
markdownai read <file> --section "#1.3" --json --summary 5 --meta
```

### Search markdown with scope/match
```bash
markdownai search <dir-or-file> -q "<query>" --scope headers --match text --json --limit 20
```

### Extract one JSON value or subtree
```bash
jsonai cat <file> --pointer /path/to/node --pretty
jsonai query -f '.path.to.value' <file>
```

### Safely update JSON (dry run first)
```bash
jsonai set --pointer /path/to/key '<value-json>' <file> --dry-run --pretty
jsonai set --pointer /path/to/key '<value-json>' <file> --pretty
```

---

## Response discipline

- Do not dump full files if block/section/value access is enough.
- Prefer targeted reads + small limits.
- State which *ai command was used when summarizing findings.
