# worktoolai Installation Guide

Install jsonai, markdownai, and codeai CLI tools.

## Install

```bash
# jsonai
curl -fsSL https://raw.githubusercontent.com/worktoolai/jsonai/main/install.sh | sh

# markdownai
curl -fsSL https://raw.githubusercontent.com/worktoolai/markdownai/main/install.sh | sh

# codeai
curl -fsSL https://raw.githubusercontent.com/worktoolai/codeai/main/install.sh | sh
```

## Verify

```bash
jsonai --help
markdownai --help
codeai --help
```

## Tool Summary

| Tool | Purpose | Key Commands |
|------|---------|--------------|
| jsonai | Read, search, and modify JSON | `cat`, `search`, `query`, `set` |
| markdownai | Structured markdown read, search, and modify | `toc`, `read`, `search`, `section-set` |
| codeai | Block-level code exploration | `index`, `search`, `outline`, `open` |
