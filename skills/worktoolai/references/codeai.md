# codeai Command Reference

Agent-first code exploration. Block-level access — treats functions/classes as the unit, not files.

## Synopsis

```
codeai <COMMAND>

Commands:
  index    Index the codebase
  search   Search for code blocks
  outline  List blocks in a file
  open     Open (read) code blocks by symbol ID

Workflow: index → search/outline → open
Output: --fmt thin (default, compact JSON) | json (pretty) | lines (one per line)
Exit: 0=ok, 1=error
```

## Commands

### `index`

Index the codebase. Incremental by default (skips unchanged files).

```bash
codeai index                           # incremental
codeai index --full                    # full reindex from scratch
codeai index --lang rust               # only Rust files
codeai index --path src/               # only files under src/
codeai index --no-gitignore            # include gitignored files
```

| Flag | Description | Default |
|------|-------------|---------|
| `--full` | Full reindex (delete existing index first) | |
| `--path <PATH>` | Filter by path | |
| `--lang <LANG>` | Filter by language | |
| `--no-gitignore` | Disable .gitignore respect | |
| `--no-default-ignores` | Disable built-in ignore patterns | |
| `--ignore-file <FILE>` | Additional ignore file | |
| `--max-bytes <N>` | Max output bytes | `12000` |
| `--fmt <FMT>` | Output format: `thin`, `json`, `lines` | `thin` |

### `search <query>`

Search for code blocks by name, content, doc comments, or string literals.

```bash
codeai search "validate payment"
codeai search "TODO" --path "src/services/"
codeai search "connection refused" --limit 5
codeai search "validate" --lang go     # only Go blocks
codeai search "error" --path src/      # only in src/
```

| Flag | Description | Default |
|------|-------------|---------|
| `--limit <N>` | Max results | `10` |
| `--path <PATH>` | Filter by path prefix | |
| `--lang <LANG>` | Filter by language | |
| `--max-bytes <N>` | Max output bytes | `12000` |
| `--cursor <CURSOR>` | Pagination cursor | |
| `--fmt <FMT>` | Output format | `thin` |

### `outline <path>`

List blocks in a file — the code equivalent of a table of contents.

```bash
codeai outline src/main.rs                    # all blocks
codeai outline src/main.rs --kind function    # functions only
codeai outline src/main.rs --kind struct      # structs only
```

| Flag | Description | Default |
|------|-------------|---------|
| `--kind <KIND>` | Filter by block kind | |
| `--limit <N>` | Max results | `100` |
| `--max-bytes <N>` | Max output bytes | `12000` |
| `--cursor <CURSOR>` | Pagination cursor | |
| `--fmt <FMT>` | Output format | `thin` |

Block kinds: `function`, `method`, `class`, `struct`, `interface`, `trait`, `enum`, `impl`, `module`, `namespace`, `block`, `object`, `protocol`

### `open`

Open (read) code blocks by symbol ID. The core operation.

```bash
# Single block
codeai open --symbol "src/auth/handler.go#function#Login"

# Batch: multiple blocks in one call
codeai open --symbols "src/a.rs#function#foo,src/b.rs#struct#Bar"

# Raw range (no index needed)
codeai open --range "src/main.rs:10:0-25:0"
```

| Flag | Description | Default |
|------|-------------|---------|
| `--symbol <ID>` | Single symbol ID | |
| `--symbols <IDs>` | Comma-separated symbol IDs (batch read) | |
| `--range <RANGE>` | Range: `path:L:C-L:C` | |
| `--preview-lines <N>` | Preview lines per block | `80` |
| `--max-bytes <N>` | Max output bytes | `16000` |
| `--fmt <FMT>` | Output format | `thin` |

Symbol ID format: `path#kind#name` or `path#kind#name#N` (N=occurrence index).
Obtained from: search results (`i[][0]`), outline results (`i[][0]`).

## Symbol IDs

Stable, human-readable identifiers for code blocks:

```
<path>#<kind>#<name>
src/auth/handler.go#function#Login
utils/helpers.py#class#ConfigManager
```

- Survive code edits (no line numbers)
- If a symbol goes stale, `open` returns structured recovery hints

## Output Format (Thin JSON)

All output is tuple-based minimal JSON:

```json
{
  "v": 1,
  "m": ["search", 12000, 843, 0, null],
  "i": [["symbol_id", "name", "file", "range", score, ["match_fields"], "preview"]],
  "h": [["open", {"symbol_id": "..."}]],
  "e": null
}
```

| Field | Description |
|-------|-------------|
| `v` | Schema version (currently `1`) |
| `m` | Meta tuple: `[cmd, max_bytes, byte_count, truncated, next_cursor]` |
| `i` | Items (tuples, format varies by command) |
| `h` | Hints (suggested next actions) |
| `e` | Error with `code`, `message`, `recovery` |

### Structured errors

```json
{
  "v": 1,
  "e": {
    "code": "SYMBOL_NOT_FOUND",
    "message": "symbol_id 'old_id' not found in current index",
    "recovery": [["search", {"query": "ValidatePayment"}]]
  }
}
```

## Supported Languages

18 languages with Tree-sitter grammars:

Go, Rust, Python, TypeScript, TSX, JavaScript, JSX, Java, Kotlin, C, C++, C#, Swift, Scala, Ruby, PHP, Bash, HCL

## Agent Workflow

```
1. index    →  build/update block index
2. search   →  find candidate blocks (even with fuzzy recall)
3. open     →  read just those blocks
4. repeat with refined search if needed
```

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Error |
