# worktoolai-skill

AI agent skill for structured code, markdown, and JSON processing — integrates **codeai**, **markdownai**, and **jsonai** into a single installable skill for Claude Code, OpenCode, and other agentskills.io-compatible tools.

## What's included

- **codeai** — Explore source code at block level (functions, classes). Index, search, and read individual blocks instead of whole files.
- **markdownai** — Read, search, and modify markdown documents by section. 80-90% token savings.
- **jsonai** — Read, search, and modify JSON with compact output, full-text search, and built-in jq filters.

## Install

### Claude Code (marketplace)

```bash
/plugin marketplace add https://github.com/worktoolai/worktoolai-skill
/plugin install worktoolai
```

### Claude Code (direct)

```bash
claude /plugin install https://github.com/worktoolai/worktoolai-skill
```

### OpenCode

```bash
cp -r skills/worktoolai ~/.config/opencode/skills/
```

Also listed on [awesome-opencode](https://github.com/awesome-opencode/awesome-opencode).

### Other agentskills.io-compatible tools

Copy the `skills/worktoolai` directory to your tool's skills directory.

## Prerequisites

The CLI tools must be installed separately:

```bash
curl -fsSL https://raw.githubusercontent.com/worktoolai/codeai/main/install.sh | sh
curl -fsSL https://raw.githubusercontent.com/worktoolai/markdownai/main/install.sh | sh
curl -fsSL https://raw.githubusercontent.com/worktoolai/jsonai/main/install.sh | sh
```

## Structure

```
skills/worktoolai/
├── SKILL.md                 # Main skill (compact usage contract)
└── references/
    └── install.md           # Installation guide (install-only reference)
```

All command usage guidance is intentionally consolidated in `SKILL.md`.

### Search tips (all tools, quick)

- Use semantic keywords, not escaped signatures/regex.
- Prefer 2-5 tokens: `<target> <action> <context>` (e.g., `service update method`).
- Narrow with tool-specific flags (path/lang/limit-style options).
- Empty result means "no matches" (not a command error); broaden/normalize and retry.
- Confirm relevant files/data exist before highly specific queries.

## License

MIT
