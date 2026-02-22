# jsonai Command Reference

Agent-first JSON full-text search CLI. Compact output by default, full-text search returns only matching objects, overflow protection, built-in jq filters.

## Synopsis

```
jsonai [OPTIONS] <COMMAND>

Commands:
  cat     Output JSON file as compact JSON (no search)
  search  Search JSON files by value
  fields  List searchable fields from a JSON file or schema
  set     Set/update a field value at a JSON Pointer path
  add     Add a value at a JSON Pointer path (append to arrays)
  delete  Delete a value at a JSON Pointer path
  patch   Apply a JSON Patch (RFC 6902) document
  query   Run a jq filter on JSON input

Options:
  --pretty   Pretty-print JSON output (default for stdout: compact, for file writes: pretty)
  --compact  Compact JSON output (override pretty default for file writes)
```

## Commands

### `cat`

Output JSON file as compact JSON (no search).

```bash
jsonai cat data.json                        # whole file, compact
jsonai cat -p /0 data.json                  # extract by JSON Pointer
jsonai cat -p /database/host config.json    # drill into nested value
curl ... | jsonai cat -                     # compact stdin
```

| Flag | Short | Description |
|------|-------|-------------|
| `--pointer` | `-p` | JSON Pointer path to extract a subtree |

### `search`

Search JSON files by value.

```bash
jsonai search -q "john" --all users.json
jsonai search -q "developer" -f role users.json
jsonai search -q "timeout" --all ./configs/
curl ... | jsonai search -q "error" --all -
```

#### Search options

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--query` | `-q` | Search query string | required |
| `--field` | `-f` | Search in specific field (repeatable) | |
| `--all` | `-a` | Search across all values | default if no `-f` |
| `--match` | `-m` | Match mode: `text` `exact` `fuzzy` `regex` | `text` |

#### Output options

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--output` | `-o` | Output mode: `match` `hit` `value` | `match` |
| `--limit` | `-l` | Max results | `20` |
| `--offset` | | Skip first N results | `0` |
| `--count-only` | | Return count only | |
| `--select` | | Project specific fields (comma-separated) | |
| `--bare` | | Bare JSON array, no envelope | |
| `--max-bytes` | | Max output bytes (truncated to fit, JSON stays valid) | |
| `--schema` | | JSON Schema file for structure awareness | |

#### Match modes

```bash
# text (default) — tokenized full-text search
jsonai search -q "john doe" --all data.json

# exact — exact value match
jsonai search -q "admin" -f role -m exact data.json

# fuzzy — edit distance tolerance
jsonai search -q "jon" --all -m fuzzy data.json

# regex — regular expression
jsonai search -q "^j.*@example" --all -m regex data.json
```

#### Overflow protection

When results exceed `--threshold` (default: 50), returns a **plan** instead of all results.

| Flag | Description | Default |
|------|-------------|---------|
| `--threshold` | Result count that triggers plan mode | `50` |
| `--plan` | Force plan mode | |
| `--no-overflow` | Bypass overflow, always return results | |

Plan mode output includes:
- **fields**: field names with distinct value counts (sorted by cardinality)
- **facets**: value distributions for low-cardinality fields (top 5 values)
- **commands**: ready-to-run narrowing commands

```bash
jsonai search -q "error" --all ./logs/            # auto plan if >50 results
jsonai search -q "error" --all --plan ./logs/     # force plan mode
jsonai search -q "error" --all --no-overflow ./logs/  # bypass overflow
```

### `query`

Run a jq filter on JSON input. No jq installation required.

```bash
jsonai query -f '.[] | select(.status == "open")' issues.json
jsonai query -f 'map(.name)' users.json
jsonai query -f 'group_by(.type) | map({key: .[0].type, count: length})' data.json
jsonai query -f 'keys' config.json
curl ... | jsonai query -f '[.items[] | {id, title}]' -
```

| Flag | Short | Description |
|------|-------|-------------|
| `--filter` | `-f` | jq filter expression (required) |

Single results output as a value; multiple results as an array.

### `fields`

List searchable fields from a JSON file or schema.

```bash
jsonai fields data.json
jsonai fields schema.json --schema    # use as JSON Schema
```

| Flag | Description |
|------|-------------|
| `--schema` | Treat input as JSON Schema |

### `set`

Set/update a value at a JSON Pointer path.

```bash
jsonai set -p /0/name '"New Name"' users.json
jsonai set -p /database/port '5433' config.json
jsonai set -p /0/name '"Test"' users.json --dry-run
jsonai set -p /0/name '"Test"' users.json -o out.json
```

| Flag | Short | Description |
|------|-------|-------------|
| `--pointer` | `-p` | JSON Pointer path (required) |
| `--dry-run` | | Preview without writing |
| `--output` | `-o` | Write to different file |

### `add`

Add a value at a JSON Pointer path (append to arrays, insert at index, add to objects).

```bash
jsonai add -p /users/- '{"id":6,"name":"New User"}' data.json    # append to array
jsonai add -p /users/0 '{"id":0,"name":"First"}' data.json       # insert at index 0
jsonai add -p /settings/theme '"dark"' config.json                # add to object
```

| Flag | Short | Description |
|------|-------|-------------|
| `--pointer` | `-p` | JSON Pointer path (required). `/arr/-` appends, `/arr/0` inserts at index |
| `--dry-run` | | Preview without writing |
| `--output` | `-o` | Write to different file |

### `delete`

Delete a value at a JSON Pointer path.

```bash
jsonai delete -p /0/email users.json
jsonai delete -p /users/2 data.json
```

| Flag | Short | Description |
|------|-------|-------------|
| `--pointer` | `-p` | JSON Pointer path to delete (required) |
| `--dry-run` | | Preview without writing |
| `--output` | `-o` | Write to different file |

### `patch`

Apply a JSON Patch (RFC 6902). Supports: `test`, `add`, `remove`, `replace`, `move`, `copy`.

```bash
jsonai patch -p patch.json target.json
echo '[{"op":"replace","path":"/0/name","value":"Updated"}]' | jsonai patch -p - target.json
```

| Flag | Short | Description |
|------|-------|-------------|
| `--patch` | `-p` | Patch document file or `-` for stdin (required) |
| `--dry-run` | | Preview without writing |
| `--output` | `-o` | Write to different file |

## Global Flags

| Flag | Description | Default |
|------|-------------|---------|
| `--pretty` | Pretty-print JSON | stdout: compact |
| `--compact` | Force compact JSON | file writes: pretty |

Defaults are agent-optimized: stdout is compact (save tokens), file writes are pretty (human readable).

## Output Format

### Default: envelope

```json
{"meta":{"total":1,"returned":1,"limit":20,"truncated":false,"files_searched":1},"results":[...]}
```

`meta.truncated` tells the agent if there are more results beyond the limit or byte budget.

### `--bare`

Bare JSON array, no envelope:
```json
[{"id":1,"name":"John Doe"}]
```

### `--output hit`

Includes file path, JSON Pointer (RFC 6901), and relevance score:
```json
{"meta":{...},"hits":[{"file":"users.json","pointer":"/0","record":{...},"score":1.906}]}
```

### `--output value`

Returns only matched values.

### `--count-only`

```json
{"meta":{"total":5,"returned":0,"limit":20,"truncated":false}}
```

### `--max-bytes`

Truncate results to byte budget. JSON remains valid; `meta.truncated` indicates overflow.

## Input Types

- File path: `data.json`
- Directory (recursive `*.json`): `./configs/`
- Glob pattern: `./**/*.json`
- Stdin: `-`

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success / matches found |
| `1` | No matches (not an error) |
| `2` | Error (parse, runtime) |

Errors go to stderr. stdout is always clean JSON (or empty).
