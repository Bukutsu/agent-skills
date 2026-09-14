---
name: git-peek
description: >-
  Inspect external Git repositories by shallow-cloning to a persistent cache.
  Use when the user gives owner/repo or a GitHub/GitLab URL,
  or asks to inspect remote structure or symbols.
  Not for local workspace files.
---

# Git Peek

Shallow-clone external Git repositories to a local cache to inspect structure, search symbols, and read source files without web API rate limits.

## Workflow

### 1. Ensure repository is available
Cache path: `/tmp/git-peek/<owner>/<repo>`.

- **Cache hit:** use the cache immediately. Refresh only when asked:
  ```bash
  git -C "/tmp/git-peek/<owner>/<repo>" pull --ff-only || (rm -rf "/tmp/git-peek/<owner>/<repo>" && git clone --depth 1 <repo-url-or-slug> "/tmp/git-peek/<owner>/<repo>")
  ```
- **Cache miss:**
  ```bash
  mkdir -p "/tmp/git-peek/<owner>"
  git clone --depth 1 <repo-url-or-slug> "/tmp/git-peek/<owner>/<repo>"
  ```

**Complete when:** the repo is present at the cache path.

### 2. Inspect with available tools
Use your harness's search, directory listing, and file-reading tools on the cache path:
- Check directory structure to understand architecture and entry points.
- Search for relevant symbols, function names, and configuration files.
- Read specific implementation files.

**Complete when:** architecture, entry points, and target symbols are located with paths noted.

### 3. Answer
Answer the user's question, citing exact file paths and line numbers from the repository.

**Complete when:** the answer cites file:line for every claim.

## Retention
Retain the cache path across turns so follow-up questions proceed without re-cloning. The OS `/tmp` cleans the cache across reboots; remove the cache path only when explicitly requested.
