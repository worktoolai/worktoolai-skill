---
name: worktoolai
description: "Use for code/markdown/JSON read/search/analyze/edit tasks. Prefer codeai/markdownai/jsonai CLI via Bash; allow Read/Grep/Glob when they are clearly faster for narrow lookups."
---

# worktoolai

## taskai (task orchestration)

AI agent task orchestration CLI. Manages plans, tasks with dependencies, and agent assignment.

**Setup**: Requires a git repo. Run `taskai init` first. DB stored at `<git-root>/.worktoolai/taskai/taskai.db`.

Help-verified commands:
- `init` — initialize DB in git repo
- `plan create|list|show|activate|delete|load` — plan lifecycle
- `task add|list|show|start|done|fail|skip|cancel` — task lifecycle
- `task dep add|remove` — manage task dependencies
- `next` — get next ready task (highest priority, then sort order)
- `status` — show overall progress

Global flags: `--json`, `--plan <NAME|ID>`

### Orchestrator loop (primary use case)

```bash
# Load a plan from JSON
echo '{"name":"my-plan","title":"My Plan","tasks":[
  {"id":"t1","title":"Setup","agent":"claude-code","priority":10},
  {"id":"t2","title":"Build","agent":"claude-code","after":["t1"]},
  {"id":"t3","title":"Test","agent":"claude-code","after":["t2"]}
]}' | taskai plan load --json

# Agent loop: claim → execute → done
while true; do
  TASK=$(taskai next --claim --agent "my-agent" --json)
  # exit 0 + plan_completed=true → done
  # exit 2 → waiting (blocked/in_progress remain)
  TASK_ID=$(echo "$TASK" | jq -r '.data.task.id')
  # ... execute task ...
  taskai task done "$TASK_ID" --json
done
```

### Key concepts

- **`agent`** field: pre-assigned at creation (who *should* execute). Set via `task add --agent` or `"agent"` in `plan load` JSON.
- **`assigned_to`** field: set at runtime when claimed (who *actually* claimed). Set via `next --claim --agent` or `task start --agent`.
- **Status flow**: `blocked` → `ready` → `in_progress` → `done` / `cancelled` / `skipped`
- **Unblock rule**: only `done` unblocks dependents. `cancelled`/`skipped` do NOT.
- **Exit codes**: 0 = success/completed, 1 = error, 2 = waiting (blocked remaining)

### plan load JSON task fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | yes | Temporary ID for dependency references |
| `title` | yes | Task title |
| `description` | no | Task description |
| `priority` | no | Integer, default 0. Higher = picked first |
| `agent` | no | Pre-assigned agent name for routing |
| `after` | no | List of task IDs this depends on |
| `documents` | no | List of `{title, content}` |

### Task commands quick reference

```bash
taskai task add "Title" --agent "bot" --priority 5 --after <ID>
taskai task list --json
taskai task show <ID> --json
taskai task start <ID> --agent "bot"
taskai task done <ID>
taskai task fail <ID>          # in_progress → ready (or blocked)
taskai task skip <ID>          # ready|blocked → skipped
taskai task cancel <ID>
taskai task dep add <ID> <DEP_ID>
taskai task dep remove <ID> <DEP_ID>
taskai next --json             # read-only peek
taskai next --claim --agent "bot" --json  # atomic claim
taskai status --json
```

---

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

## When to use for initial project analysis

When first encountering an unfamiliar codebase or markdown knowledge base, use these tools to build a mental model quickly:

**Code projects** — use `codeai` to understand structure before diving into files:
1. `codeai index` — build the block index (incremental, fast on repeat)
2. `codeai project get --fmt thin` — infer entrypoints, shared modules, and orphans from the dependency graph.
   - **Multi-project / monorepo**: always split by subproject with `--path <subdir>`. Running on the whole repo mixes languages/frameworks and produces unusable results.
   - e.g. `codeai project get --path backend --fmt thin` → `codeai project get --path frontend --fmt thin`
3. `codeai graph <entry-file> --fmt thin --depth 2` — trace imports from a key entry point
4. `codeai outline <file> --fmt thin` — list functions/classes/structs in a file

**Markdown knowledge bases** — use `markdownai` to survey docs before reading:
1. `markdownai tree <dir> --depth 2` — directory structure of markdown files
2. `markdownai overview <dir> --json --limit 30` — file-level summary with frontmatter + structure metadata (line count, section count, etc.)
3. `markdownai search <dir> -q "<topic>" --scope headers --json --limit 20` — find relevant sections by header
4. `markdownai frontmatter <dir> --facets tags --json` — discover tag/category distribution

These commands give high-level context without reading full file contents.

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
- `index`: `--full`, `--path <prefix>`, `--lang <language>`, `--no-gitignore`, `--no-default-ignores`, `--ignore-file <FILE>`
- `search`: `--limit <N>`, `--path <prefix>`, `--lang <language>`, `--cursor <cursor>`
- `outline`: `--kind <KIND>`, `--limit <N>`, `--cursor <cursor>`
- `open`: `--symbol`, `--symbols`, `--range`, `--preview-lines <N>` (default 80), `--offset <N>`
- `graph`: `--depth <N>`, `--limit <N>`, `--offset <N>`, `--external`
- `project get`: `--path <prefix>`, `--fmt thin`, `--max-bytes <N>`

**Important: `codeai index --full` for schema updates**
- If you already indexed code before the stemmer/identifier split feature was added, run `codeai index --full` once.
- This recreates the search index to pick up the new tokenizer configuration.

Supported languages: go, rust, python, typescript, tsx, javascript, jsx, java, kotlin, c, cpp, csharp, swift, scala, ruby, php, bash, hcl
Block kinds: function, method, class, struct, interface, trait, enum, impl, module, namespace, block, object, protocol

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

### codeai search capabilities (what works, what doesn't)

**What's supported** (English stemmer + identifier split):
- Word-form matching via English stemmer: `validate` matches `ValidateAccessToken`, `ValidationHelper`, `validates`
- CamelCase/PascalCase/snake_case word splitting: `AccessToken` indexed as `Access Token`
  - `"authentication"` matches `Authenticate`, `useAuth` (stems to `authent`)
  - `"validation"` matches `ValidateAccessToken`, `Valid` (stems to `valid`)
- Multi-word conjunctive queries: `"auth user"` matches both `auth` AND `user` (conjunction mode)

**What's NOT supported** (use other tools):
- True natural language semantic search (no embeddings, no LLM understanding)
  - `"usage user resolve email unknown"` → 0 results (use simpler keyword queries instead)
- Literal exact string search across all files (e.g., `admin@tokenai.local`, SQL queries)
  - Use `Grep` with exact pattern: `Grep --path /Users/bjm/work/ai/tokenai --pattern admin@tokenai.local`
- Cross-language import tracking (Python → Go → SQL → TypeScript)
  - `graph` command tracks imports within one language only
  - Cross-language relationships are runtime contracts (HTTP, DB), not static imports

---

## markdownai (markdown)

Help-verified commands:
- `toc` `read` `tree` `search` `frontmatter` `overview` `links` `backlinks` `graph`
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
- `toc`: `--depth <N>`, `--flat`
- `read`: `--section`, `--summary [N]`, `--meta`
- `search`: `--match text|exact|fuzzy|regex`, `--scope all|body|headers|frontmatter|code`, `--context <N>`, `--bare`
- `tree`: `--depth <N>`, `--files-only`, `--count`
- `overview`: `--field <FIELD>` (repeatable), `--filter '<expr>'`, `--sort <FIELD|name|lines|sections>`, `--reverse`
- `frontmatter`: `--field <FIELD>`, `--filter '<expr>'`, `--list`
- `links`: `--type wiki|markdown|all`, `--resolved`, `--broken`
- `graph`: `--format adjacency|edges|stats`, `--start <FILE>`, `--depth <N>`, `--orphans`
- `index`: `--force`, `--status`, `--dry-run`, `--check`
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

### Initial code project analysis
```bash
codeai index
# Single project:
codeai project get --fmt thin --max-bytes 12000
# Multi-project / monorepo — split by subdir:
codeai project get --path backend --fmt thin
codeai project get --path frontend --fmt thin
# then drill into key entry points:
codeai graph <entry-file> --fmt thin --depth 2
codeai outline <entry-file> --fmt thin
```

### Initial markdown knowledge base survey
```bash
markdownai tree <dir> --depth 2
markdownai overview <dir> --json --limit 30
markdownai frontmatter <dir> --facets tags --json
markdownai search <dir> -q "<topic>" --scope headers --json --limit 20
```

### Find function implementation
```bash
codeai index --path src/
codeai search "<function-name or behavior>" --fmt json --path src/ --limit 10
codeai open --symbol "<symbol-id>" --fmt json --preview-lines 80
```

### Inspect dependency graph from entry file
```bash
codeai graph <entry-file> --fmt thin --depth 2 --limit 20
```

### Infer project structure from index
```bash
codeai project get --fmt thin --max-bytes 12000
```

### Analyze a subproject in a monorepo
```bash
# Use --path to filter by directory prefix (avoids mixed-language noise)
codeai project get --path tokenai-proxy --fmt thin        # Go subproject only
codeai project get --path tokenai-admin-front --fmt thin   # TS subproject only
codeai project get --fmt thin                              # full monorepo (default)
```
When analyzing a monorepo, always use `--path <subdir>` to scope results to one subproject at a time. Without it, entrypoints/shared/orphan lists mix all languages and the output becomes too large to be useful.

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
