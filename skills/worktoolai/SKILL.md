---
name: worktoolai
description: Agent tools for structured code, markdown, and JSON access — read, search, and modify source code blocks, markdown documents, and JSON files with minimal token cost using codeai, markdownai, and jsonai CLIs
---

# worktoolai

Three CLI tools optimized for AI agents working with code, markdown, and JSON.

## When to use

- **codeai** — source code exploration. Index a codebase, search by function name or content, read individual blocks (functions/classes) instead of whole files.
- **markdownai** — markdown collections (docs, Obsidian vaults, knowledge bases). Read sections instead of whole files. Search across documents. Modify surgically.
- **jsonai** — JSON files (configs, API responses, data). Compact output by default. Full-text search returns only matching objects. jq filters built-in.

All tools share the same design principles: minimal token output, overflow protection, byte budgets, and structured envelopes.

## References

| Document | Description |
|----------|-------------|
| [install.md](references/install.md) | Installation guide for all three tools |
| [codeai.md](references/codeai.md) | codeai full command reference, output format, symbol ID system |
| [markdownai.md](references/markdownai.md) | markdownai full command reference, flags, and examples |
| [jsonai.md](references/jsonai.md) | jsonai full command reference, flags, and examples |

## Core Workflows

### codeai — Block-Level Code Exploration

```
1. index                       → parse and index the codebase
2. search "query"              → find blocks by name, content, doc comments
3. outline FILE                → list all blocks in a file (table of contents)
4. open --symbol ID            → read just that function/class
5. open --symbols id1,id2,id3  → batch read multiple blocks
```

Symbol IDs are stable (`path#kind#name`) — they survive code edits without stale line numbers.

### markdownai — Structured Markdown Access

```
1. toc FILE                    → discover structure
2. read FILE --summary         → preview all sections (first 3 lines each)
3. read FILE --section "#N.M"  → read specific section
4. search DIR -q "keyword"     → find across files
5. section-set / section-add   → modify (use --dry-run first)
```

Section addressing: TOC index (`"#1.1"`), header path (`"## Setup > ### Prerequisites"`), line range (`"L10-L25"`).

### jsonai — JSON for AI Agents

```
1. cat FILE                    → read compact JSON
2. cat -p /path FILE           → extract specific value
3. search -q "term" --all FILE → find matching objects
4. query -f 'FILTER' FILE      → jq filter (no jq needed)
5. set/add/delete              → modify by JSON Pointer
```

## Common Patterns

### Global flags

| Flag | codeai | markdownai | jsonai |
|------|--------|------------|--------|
| `--pretty` | | x | x |
| `--compact` | | | x |
| `--max-bytes <N>` | x (default: 12000/16000) | x | x |
| `--limit <N>` | x (default: 10/100) | x (default: 20) | x (default: 20) |
| `--offset <N>` | | x | x |
| `--count-only` | | x | x |
| `--fmt <FMT>` | x (thin/json/lines) | | |
| `--json` | | x | |
| `--exists` | | x | |
| `--stats` | | x | |
| `--bare` | | x (search) | x |
| `--select` | | | x |
| `--sync <MODE>` | | x (auto/force) | |
| `--root <DIR>` | | x | |

### Overflow protection

markdownai and jsonai prevent flooding agent context when results are large:

- Results exceeding `--threshold` (default: 50) trigger **plan mode**
- Plan mode returns metadata and suggestions for narrowing the query
- Use `--plan` to force plan mode, `--no-overflow` to bypass

codeai uses `--max-bytes` with cursor-based pagination instead.

### Exit codes

| Code | codeai | markdownai / jsonai |
|------|--------|---------------------|
| `0` | Success | Success / matches found |
| `1` | Error | Not found (not an error) |
| `2` | | Error (parse, runtime) |

### Stdin support

markdownai and jsonai accept `-` for stdin:

```bash
git show HEAD:docs/guide.md | markdownai toc -
curl https://api.example.com/data | jsonai search -q "error" --all -
```

### Storage

codeai and markdownai store their indices in `.worktoolai/` at the project root:
```
.worktoolai/
  markdownai.db          ← SQLite
  markdownai_index/      ← Tantivy
```

jsonai uses in-memory indexing — no disk storage required.
