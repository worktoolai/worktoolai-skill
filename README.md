# worktoolai-skill

AI agent skill for structured code, markdown, and JSON processing — integrates **codeai**, **markdownai**, and **jsonai** into a single installable skill for Claude Code, OpenCode, and other agentskills.io-compatible tools.

## What's included

- **codeai** — Explore source code at block level (functions, classes). Index, search, and read individual blocks instead of whole files.
- **markdownai** — Read, search, and modify markdown documents by section. 80-90% token savings.
- **jsonai** — Read, search, and modify JSON with compact output, full-text search, and built-in jq filters.

## Install

### Claude Code

```bash
claude /plugin install https://github.com/worktoolai/worktoolai-skill
```

### OpenCode

```bash
cp -r skills/worktoolai ~/.config/opencode/skills/
```

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
├── SKILL.md                 # Main skill (core workflows + references)
└── references/
    ├── install.md           # Installation guide
    ├── codeai.md            # Full codeai command reference
    ├── markdownai.md        # Full markdownai command reference
    └── jsonai.md            # Full jsonai command reference
```

## License

MIT
