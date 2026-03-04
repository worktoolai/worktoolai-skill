---
name: worktoolai
description: MUST invoke for ANY file read/search/edit task on code (.ts/.js/.py/.go/.rs/etc), markdown (.md), or JSON (.json). Hooks block raw Read/Grep/Glob for these types. Use codeai/markdownai/jsonai CLI via Bash. Also handles task orchestration (taskai). Only skip for non-code/non-md/non-json files.
---

# worktoolai

## Router

- Task orchestration (plan/task/dependency/agent/next/status): **taskai**
- Source code (function/class/import/flow/refactor): **codeai**
- Markdown docs/notes/sections/search/edit: **markdownai**
- JSON config/data/query/update: **jsonai**

## Tool Policy

**Default**: use `taskai`, `codeai`, `markdownai`, `jsonai` via Bash for code/markdown/json work.

**Fast path allowed**: use `Read`, `Grep`, `Glob` when clearly faster (exact path known, quick regex lookup, simple discovery).

**Never use**: `cat/grep/sed/awk/head/tail` for primary analysis.

**Rules**:
1. Choose fastest valid path
2. Prefer `*ai` CLIs for structured/deep analysis and updates
3. You MAY use `Read/Grep/Glob` when: exact path known, narrow lookup needed, or `*ai` overhead exceeds gain
4. Keep output compact: `--fmt json`, `--json`, `--limit`, `--max-bytes`, `--count-only`
5. Include proof: `TOOL_PROOF: <command>` or `FALLBACK_REASON: <why>`
6. If `*ai` fails: run `<tool> --help` and retry once, then fallback to generic tools

## taskai

Task orchestration CLI. Manages plans, tasks with dependencies, and agent assignment.

**Setup**: Requires git repo. Run `taskai init` first. DB stored at `<git-root>/.worktoolai/taskai/taskai.db`.

**Commands**:
- `init` — initialize DB in git repo
- `plan create|list|show|activate|delete|load` — plan lifecycle
- `task add|list|show|start|done|fail|skip|cancel` — task lifecycle
- `task dep add|remove` — manage dependencies
- `next` — get next ready task (highest priority, then sort order)
- `status` — show overall progress

**Global flags**: `--json`, `--plan <NAME|ID>`

**Key concepts**:
- **`agent`** field: pre-assigned at creation (who should execute). Set via `task add --agent` or `"agent"` in JSON.
- **`assigned_to`** field: set at runtime when claimed. Set via `next --claim --agent` or `task start --agent`.
- **Status flow**: `blocked` → `ready` → `in_progress` → `done` / `cancelled` / `skipped`
- **Unblock rule**: only `done` unblocks dependents. `cancelled`/`skipped` do NOT.
- **Exit codes**: 0 = success/completed, 1 = error, 2 = waiting (blocked remaining)

**plan load JSON fields**:

| Field | Required | Description |
|-------|----------|-------------|
| `id` | yes | Temporary ID for dependency references |
| `title` | yes | Task title |
| `description` | no | Task description |
| `priority` | no | Integer, default 0. Higher = picked first |
| `agent` | no | Pre-assigned agent name |
| `after` | no | List of task IDs this depends on |
| `documents` | no | List of `{title, content}` |

## codeai

Source code analysis, navigation, and editing. Supports: go, rust, python, typescript, tsx, javascript, jsx, java, kotlin, c, cpp, csharp, swift, scala, ruby, php, bash, hcl

**Commands**:
- `index` — build/update index. Flags: `--full`, `--path PREFIX`, `--lang LANG`, `--no-gitignore`, `--no-default-ignores`, `--ignore-file FILE`
- `search QUERY` — find code blocks. Flags: `--limit N`, `--path PREFIX`, `--lang LANG`, `--fmt json|thin|lines`, `--cursor CURSOR`, `--max-bytes N`
- `outline PATH` — list blocks in file. Flags: `--kind KIND`, `--limit N`, `--fmt json|thin|lines`, `--cursor CURSOR`, `--max-bytes N`
- `open` — read block/range. Flags: `--symbol ID`, `--symbols ID...`, `--range FILE:START:END`, `--preview-lines N`, `--offset N`, `--fmt json|thin|lines`, `--max-bytes N`
- `graph PATH` — show import/dependency graph. Flags: `--depth N`, `--limit N`, `--offset N`, `--external`, `--fmt tree|thin`, `--max-bytes N`
- `project get` — infer entrypoint/shared/orphan structure. Flags: `--path PREFIX`, `--fmt thin`, `--max-bytes N`
- `write FILE` — create/overwrite file. Flags: `-c CONTENT`, `--content-file FILE`, `--dry-run`. Escape: `\n`→newline, `\t`→tab
- `edit FILE` — replace line range. Flags: `--range L10-L15`, `-c CONTENT`, `--content-file FILE`, `--dry-run`. 1-based inclusive range

**Block kinds**: function, method, class, struct, interface, trait, enum, impl, module, namespace, block, object, protocol

**Search tips**:
- Semantic keyword lookup (not regex): `"service update method"` not `func \(s \*Service\) Update`
- Keep queries short (2-5 words): `"index incremental generation"`
- Narrow with flags (`--path`, `--lang`, `--limit`), not punctuation
- Stemmer + identifier split: `"authentication"` matches `Authenticate`, `useAuth`, `ValidateAccessToken`
- Empty result = no matches (not error); broaden and retry

**Important**: Run `codeai index --full` once if you indexed before stemmer/identifier split feature was added.

## markdownai
Markdown analysis, search, and editing.

**Commands**:
- `toc FILE` — show table of contents. Flags: `--depth N`, `--flat`, `--json`, `--pretty`, `--max-bytes N`
- `read FILE` — read file/section. Flags: `--section ADDR`, `--summary [N]`, `--meta`, `--json`, `--pretty`, `--max-bytes N`
- `tree DIR` — directory structure. Flags: `--depth N`, `--files-only`, `--count`, `--json`, `--max-bytes N`
- `search PATH -q QUERY` — search. Flags: `--match text|exact|fuzzy|regex`, `--scope all|body|headers|frontmatter|code`, `--context N`, `--bare`, `--limit N`, `--offset N`, `--threshold N`, `--count-only`, `--json`, `--max-bytes N`
- `frontmatter PATH` — query frontmatter. Flags: `--field FIELD`, `--filter EXPR`, `--list`, `--facets FIELD`, `--json`, `--max-bytes N`
- `overview PATH` — file-level summary. Flags: `--field FIELD`, `--filter EXPR`, `--sort FIELD|name|lines|sections`, `--reverse`, `--limit N`, `--offset N`, `--json`, `--max-bytes N`
- `links FILE` — show links. Flags: `--type wiki|markdown|all`, `--resolved`, `--broken`, `--json`, `--max-bytes N`
- `backlinks FILE` — show backlinks. Flags: `--root DIR`, `--json`, `--max-bytes N`
- `graph PATH` — link graph. Flags: `--format adjacency|edges|stats`, `--start FILE`, `--depth N`, `--orphans`, `--root DIR`, `--json`, `--max-bytes N`
- `write FILE` — create/overwrite file. Flags: `-c CONTENT`, `--content-file FILE`, `--dry-run`. Escape: `\n`→newline, `\t`→tab
- `section-set FILE --section ADDR` — replace section body only (headings forbidden in content). Flags: `-c TEXT`, `--content-file FILE`, `--dry-run`, `--output FILE`, `--with-toc`
- `section-replace FILE --section ADDR` — replace entire section including heading (content must start with heading). Flags: `-c TEXT`, `--content-file FILE`, `--dry-run`, `--output FILE`, `--with-toc`
- `section-add FILE --title TITLE` — add new section. Flags: `-c TEXT`, `--content-file FILE`, `--after ADDR`, `--before ADDR`, `--level N`, `--dry-run`, `--output FILE`, `--with-toc`
- `section-delete FILE --section ADDR` — delete section. Flags: `--dry-run`, `--output FILE`, `--with-toc`
- `frontmatter-set FILE` — set frontmatter field. Flags: `-k KEY`, `-v VALUE`, `--dry-run`, `--output FILE`
- `renum FILE` — renumber headings. Flags: `--dry-run`, `--output FILE`
- `chars` — count characters. Input: file, directory, or stdin. Flags: `--json`, `--max-bytes N`
- `index DIR` — build search index. Flags: `--force`, `--status`, `--dry-run`, `--check`, `--sync auto|force`, `--root DIR`

**Section mutation rules**:
- `section-set`: body only, content with `#` headings → error
- `section-replace`: heading+body, content must start with `#` heading → error if not
- `section-add`: creates new section with heading
- `section-delete`: removes heading+body

**Section addressing**: `"#1.2"` (TOC index), `"## A > ### B"` (header path), `"L10-L25"` (line range)

**Shell quoting for edits**:
- Prefer `--content-file FILE` for multi-line content or content with backticks
- If inline: use single quotes or ANSI-C quotes (`$'content\n'`)
- Never use double quotes with backticks (command substitution)
- Keep list bullets as one argument (quote the whole payload)
## jsonai

JSON query and editing.

**Commands**:
- `cat FILE` — display JSON. Flags: `--pointer /path`, `--pretty`, `--compact`
- `search FILE -q TERM` — search. Flags: `--field FIELD` (repeatable), `--all`, `--match text|exact|fuzzy|regex`, `--output match|hit|value`, `--select FIELDS`, `--limit N`, `--offset N`, `--count-only`, `--bare`, `--max-bytes N`, `--threshold N`, `--schema FILE`
- `fields FILE` — list fields. Flags: `--schema FILE`
- `query FILE -f FILTER` — jq filter. Flags: `--filter JQ_EXPR`
- `set FILE --pointer /path VALUE` — set value. Flags: `--output FILE`, `--dry-run`, `--pretty`, `--compact`
- `add FILE --pointer /path VALUE` — add value. Flags: `--output FILE`, `--dry-run`, `--pretty`, `--compact`
- `delete FILE --pointer /path` — delete value. Flags: `--output FILE`, `--dry-run`, `--pretty`, `--compact`
- `patch FILE --patch DOC` — apply JSON patch. Flags: `--output FILE`, `--dry-run`, `--pretty`, `--compact`

## Best Practices

### Orchestrator loop
```bash
# Load plan from JSON
echo '{"name":"my-plan","title":"My Plan","tasks":[
  {"id":"t1","title":"Setup","agent":"claude-code","priority":10},
  {"id":"t2","title":"Build","agent":"claude-code","after":["t1"]},
  {"id":"t3","title":"Test","agent":"claude-code","after":["t2"]}
]}' | taskai plan load --json

# Agent loop: claim → execute → done
while true; do
  TASK=$(taskai next --claim --agent "my-agent" --json)
  TASK_ID=$(echo "$TASK" | jq -r '.data.task.id')
  # ... execute task ...
  taskai task done "$TASK_ID" --json
done
```

### Initial code project analysis
```bash
codeai index
# Single project:
codeai project get --fmt thin --max-bytes 12000
# Multi-project / monorepo — split by subdir:
codeai project get --path backend --fmt thin
codeai project get --path frontend --fmt thin
# Drill into entry points:
codeai graph src/main.ts --fmt thin --depth 2
codeai outline src/main.ts --fmt thin
```

### Initial markdown knowledge base survey
```bash
markdownai tree docs/ --depth 2
markdownai overview docs/ --json --limit 30
markdownai frontmatter docs/ --facets tags --json
markdownai search docs/ -q "authentication" --scope headers --json --limit 20
```

### Find and read a function
```bash
codeai index --path src/
codeai search "validateToken" --fmt json --path src/ --limit 10
codeai open --symbol "src/auth.ts#function#validateToken" --fmt json --preview-lines 80
```

### Inspect dependency graph
```bash
codeai graph src/index.ts --fmt thin --depth 2 --limit 20
```

### Analyze monorepo subproject
```bash
# Use --path to filter by directory (avoids mixed-language noise)
codeai project get --path tokenai-proxy --fmt thin        # Go only
codeai project get --path tokenai-admin-front --fmt thin   # TS only
```

### Read one markdown section
```bash
markdownai toc docs/guide.md --json --limit 100
markdownai read docs/guide.md --section "#1.3" --json --summary 5 --meta
```

### Search markdown with scope
```bash
markdownai search docs/ -q "deployment" --scope headers --match text --json --limit 20
```

### Extract JSON value
```bash
jsonai cat config.json --pointer /database/host --pretty
jsonai query -f '.database.host' config.json
```

### Update JSON safely
```bash
jsonai set --pointer /database/port '5432' config.json --dry-run --pretty
jsonai set --pointer /database/port '5432' config.json --pretty
```

### Search code across files
```bash
codeai search "authentication middleware" --path src/ --lang typescript --fmt json --limit 10
```

### Edit markdown section
```bash
echo "New content here" > /tmp/new_content.txt
markdownai section-set docs/guide.md --section "## Installation" --content-file /tmp/new_content.txt --dry-run
markdownai section-set docs/guide.md --section "## Installation" --content-file /tmp/new_content.txt
```

### Write new files
```bash
# Create new code file
codeai write src/utils/helper.ts -c "export function greet(name: string): string {\n  return \`Hello \${name}\`;\n}"

# Create new markdown file
markdownai write docs/setup.md -c "# Setup Guide\n\n## Prerequisites\n\nNode.js 18+\n\n## Installation\n\nnpm install"

# Edit specific lines in code
codeai edit src/main.rs --range L10-L15 -c "    let result = compute();\n    println!(\"{}\", result);"
```

## Response Discipline

- Do not dump full files if block/section/value access is enough
- Prefer targeted reads + small limits
- State which *ai command was used when summarizing findings
