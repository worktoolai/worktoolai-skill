# worktoolai Installation Guide

Install taskai, jsonai, markdownai, and codeai CLI tools.

## Install

```bash
# taskai
curl -fsSL https://raw.githubusercontent.com/worktoolai/taskai/main/install.sh | sh

# jsonai
curl -fsSL https://raw.githubusercontent.com/worktoolai/jsonai/main/install.sh | sh

# markdownai
curl -fsSL https://raw.githubusercontent.com/worktoolai/markdownai/main/install.sh | sh

# codeai
curl -fsSL https://raw.githubusercontent.com/worktoolai/codeai/main/install.sh | sh
```

## Verify

```bash
taskai --help
jsonai --help
markdownai --help
codeai --help
```

## Tool Summary

| Tool | Purpose | Key Commands |
|------|---------|--------------|
| taskai | AI agent task orchestration | `plan load`, `next`, `task done`, `status` |
| jsonai | Read, search, and modify JSON | `cat`, `search`, `query`, `set` |
| markdownai | Structured markdown read, search, and modify | `toc`, `read`, `search`, `section-set` |
| codeai | Block-level code exploration | `index`, `search`, `outline`, `open` |
