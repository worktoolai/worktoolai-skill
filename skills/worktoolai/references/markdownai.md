# markdownai Command Reference

Agent-first CLI for structured markdown access. Read sections instead of whole files, search across documents, modify surgically.

## Synopsis

```
markdownai [OPTIONS] <COMMAND>

Commands:
  toc              Headings with section numbers
  read             Read file or section
  tree             Directory structure
  search           Full-text search
  frontmatter      YAML frontmatter fields
  links            Outgoing links
  backlinks        Incoming links (backlinks)
  graph            Link graph
  section-set      Replace section content
  section-add      Add new section
  section-delete   Delete section
  frontmatter-set  Set frontmatter field
  index            DB management
```

## Section Address Formats

| Format | Example | Use case |
|--------|---------|----------|
| TOC index | `"#1.1"` | Stable reference from `toc` output |
| Header path | `"## Setup > ### Prerequisites"` | Human-readable |
| Line range | `"L10-L25"` | Precise byte-level access (1-based, inclusive) |

TOC indices are assigned by appearance order. Duplicate header names get different indices:
```
## FAQ          → #1.1
### Question    → #1.1.1
### Question    → #1.1.2    ← same name, different index
## API          → #1.2
```

After modifying a document, use `--with-toc` to get updated indices in the response.

## Commands

### `toc`

Headings with section numbers.

```bash
markdownai toc docs/guide.md
markdownai toc docs/guide.md --depth 2    # h1+h2 only
markdownai toc docs/guide.md --flat       # no indentation
```

Output:
```
1   # Guide                          (L1)
1.1 ## Setup                         (L5)
1.1.1 ### Prerequisites              (L8)
1.2 ## Usage                         (L20)
```

| Flag | Description |
|------|-------------|
| `--depth <N>` | Max heading depth (1-6) |
| `--flat` | Flat output (no indentation) |

### `read`

Read file or section.

```bash
markdownai read docs/guide.md                           # whole file
markdownai read docs/guide.md --section "#1.1"          # by TOC index
markdownai read docs/guide.md --section "## Setup > ### Prerequisites"  # by header path
markdownai read docs/guide.md --section "L10-L25"       # by line range
markdownai read docs/guide.md --summary                 # first 3 lines per section
markdownai read docs/guide.md --summary 5               # first 5 lines per section
markdownai read docs/guide.md --meta                    # include frontmatter
markdownai read docs/guide.md --stats                   # size/structure statistics only
```

| Flag | Short | Description |
|------|-------|-------------|
| `--section` | `-s` | Section address: `"#1.1"`, `"## Heading"`, `"L10-L25"` |
| `--summary` | | Preview first N lines per section (default 3) |
| `--meta` | | Include frontmatter in output |

### `tree`

Directory structure.

```bash
markdownai tree ./docs
markdownai tree ./docs --depth 2          # limit depth
markdownai tree ./docs --files-only       # files only
markdownai tree ./docs --count            # show count only
```

| Flag | Description |
|------|-------------|
| `--depth <N>` | Max depth |
| `--files-only` | Show files only (no directories) |
| `--count` | Show count only |

### `search`

Full-text search across files.

```bash
markdownai search ./docs -q "authentication"
markdownai search ./docs -q "OAuth" --scope headers --match exact
markdownai search ./docs -q "OAuth" -q "JWT"             # multi-query
markdownai search ./docs -q "OAuth" --context 2           # context lines
markdownai search ./docs -q "OAuth" --count-only
markdownai search ./docs -q "OAuth" --exists              # exit code only
markdownai search ./docs -q "OAuth" --bare                # no envelope
```

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--query` | `-q` | Search query (repeatable) | required |
| `--scope` | | Search scope: `all`, `body`, `headers`, `frontmatter`, `code` | `all` |
| `--match` | `-m` | Match mode: `text`, `exact`, `fuzzy`, `regex` | `text` |
| `--context` | | Context lines around match | `0` |
| `--bare` | | Output bare results (no envelope) | |

### `frontmatter`

YAML frontmatter fields.

```bash
markdownai frontmatter ./docs --list                     # list all unique keys
markdownai frontmatter ./docs --filter 'tags contains "rust"'
markdownai frontmatter ./docs --facets tags               # value distribution
markdownai frontmatter docs/guide.md --field title        # extract specific field
```

| Flag | Description |
|------|-------------|
| `--list` | List all unique frontmatter keys |
| `--field <FIELD>` | Extract specific field |
| `--filter <EXPR>` | Filter by field value |
| `--facets <FIELD>` | Show value distribution |

### `links`

Outgoing links from a file.

```bash
markdownai links docs/index.md
markdownai links docs/index.md --broken                   # broken links only
markdownai links docs/index.md --resolved                 # resolved links only
markdownai links docs/index.md --type wiki                # wiki links only
```

| Flag | Description |
|------|-------------|
| `--type` | Link type filter: `wiki`, `markdown`, `all` |
| `--broken` | Show only broken links |
| `--resolved` | Show only resolved links |

### `backlinks`

Incoming links (backlinks) to a file.

```bash
markdownai backlinks docs/auth.md
```

### `graph`

Link graph visualization.

```bash
markdownai graph ./docs --format stats
markdownai graph ./docs --format adjacency
markdownai graph ./docs --format edges
markdownai graph ./docs --start docs/index.md --depth 2
markdownai graph ./docs --orphans                         # show orphan nodes
```

| Flag | Description | Default |
|------|-------------|---------|
| `--format` | Output: `adjacency`, `edges`, `stats` | `adjacency` |
| `--start <FILE>` | Start node for subgraph traversal | |
| `--depth <N>` | Max traversal depth | |
| `--orphans` | Show orphan nodes | |

### `section-set`

Replace section content.

```bash
markdownai section-set docs/guide.md -s "#1.1" -c "New content here"
markdownai section-set docs/guide.md -s "#1.1" --content-file patch.md
echo "Updated" | markdownai section-set docs/guide.md -s "#1.1" --content -
markdownai section-set docs/guide.md -s "#1.1" -c "New content" --dry-run
markdownai section-set docs/guide.md -s "#1.1" -c "New content" -o out.md
```

| Flag | Short | Description |
|------|-------|-------------|
| `--section` | `-s` | Section address (required) |
| `--content` | `-c` | Inline content |
| `--content-file` | | Content from file |
| `--dry-run` | | Preview without writing |
| `--output` | `-o` | Write to different file |
| `--with-toc` | | Include updated TOC in response |

### `section-add`

Add a new section.

```bash
markdownai section-add docs/guide.md -t "Troubleshooting" -c "Common issues..." --after "#1.2" --level 2
markdownai section-add docs/guide.md -t "### Sub" --before "#1.1"
```

| Flag | Short | Description |
|------|-------|-------------|
| `--title` | `-t` | Section title (required) |
| `--content` | `-c` | Section content |
| `--content-file` | | Content from file |
| `--after <ADDR>` | | Insert after this section |
| `--before <ADDR>` | | Insert before this section |
| `--level <N>` | | Heading level (1-6) |
| `--dry-run` | | Preview without writing |
| `--output` | `-o` | Write to different file |
| `--with-toc` | | Include updated TOC in response |

### `section-delete`

Delete a section.

```bash
markdownai section-delete docs/guide.md -s "#1.1.1"
markdownai section-delete docs/guide.md -s "## Old Section"
markdownai section-delete docs/guide.md -s "#1.1" --dry-run
```

| Flag | Short | Description |
|------|-------|-------------|
| `--section` | `-s` | Section address to delete (required) |
| `--dry-run` | | Preview without writing |
| `--output` | `-o` | Write to different file |
| `--with-toc` | | Include updated TOC in response |

### `frontmatter-set`

Set a YAML frontmatter field (auto-creates frontmatter if missing).

```bash
markdownai frontmatter-set docs/guide.md -k tags -v '["rust", "cli"]'
markdownai frontmatter-set docs/guide.md -k draft -v true
```

| Flag | Short | Description |
|------|-------|-------------|
| `--key` | `-k` | Field name (required) |
| `--value` | `-v` | Field value — JSON or plain text (required) |
| `--dry-run` | | Preview without writing |
| `--output` | `-o` | Write to different file |
| `--with-toc` | | Include updated TOC in response |

### `index`

DB management.

```bash
markdownai index ./docs                                  # sync
markdownai index ./docs --force                          # full rebuild
markdownai index ./docs --status                         # current status
markdownai index ./docs --check                          # verify SQLite <-> Tantivy consistency
markdownai index ./docs --dry-run                        # show what would change
```

| Flag | Description |
|------|-------------|
| `--force` | Full rebuild (delete DB first) |
| `--status` | Show status only |
| `--check` | Verify SQLite <-> Tantivy consistency |
| `--dry-run` | Show what would change |

## Global Flags

| Flag | Description | Default |
|------|-------------|---------|
| `--json` | JSON envelope output | raw markdown |
| `--pretty` | Pretty-print JSON (only with `--json`) | |
| `--max-bytes <N>` | Truncate to byte budget | |
| `--limit <N>` | Max result items | `20` |
| `--offset <N>` | Result start position (paging) | `0` |
| `--threshold <N>` | Overflow threshold triggering plan mode | `50` |
| `--no-overflow` | Bypass overflow protection | |
| `--plan` | Force plan mode: metadata only, no results | |
| `--count-only` | Return count only | |
| `--exists` | Check existence (exit code 0=exists, 1=not) | |
| `--stats` | Size/structure statistics only | |
| `--facets <FIELD>` | Return facet distribution for a field | |
| `--sync <MODE>` | Sync mode: `auto`, `force` | `auto` |
| `--root <DIR>` | Project root override | |

## Output Modes

**Raw (default)** — plain markdown with paging footer:
```
## Setup
Rust 1.75+ required...
--- 10/47 shown, next: --offset 10 ---
```

**JSON (`--json`)** — structured envelope:
```json
{
  "meta": {"total": 47, "returned": 10, "offset": 0, "has_more": true, "next_offset": 10},
  "results": [...]
}
```

**Overflow protection** — when results exceed `--threshold` (default 50):
```json
{
  "meta": {"total": 350, "overflow": true},
  "plan": {"suggestion": "add --scope headers or narrow query"}
}
```

## Input Types

- File path: `docs/guide.md`
- Directory (recursive `.md`): `./docs`
- Stdin: `-`

```bash
git show HEAD:docs/guide.md | markdownai toc -
curl -s https://example.com/doc.md | markdownai frontmatter -
```

## Storage

Index stored in `.worktoolai/` at project root (auto-detected via `.git/`):
```
.worktoolai/
  markdownai.db          ← SQLite
  markdownai_index/      ← Tantivy
```

Sync is automatic via content hash (xxh3).

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Not found |
| `2` | Error |
