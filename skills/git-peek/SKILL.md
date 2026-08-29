---
name: git-peek
description: >-
  Inspect external Git repositories (owner/repo, GitHub/GitLab URLs) by shallow-cloning to a persistent cache.
  Use when the user says owner/repo, pastes a GitHub/GitLab URL, exploring remote codebases,
  investigating library internals, or checking third-party implementations.
  Not for local workspace files.
---

# Git Peek

Shallow-clone external Git repositories to a local cache to inspect structure, search symbols, and read source files without web API rate limits.

## Workflow

### 1. Ensure repository is available
Derive cache path: `/tmp/git-peek/<owner>/<repo>`

- **If already cached:**
  - Default: use the existing cache immediately.
  - If latest commit or refresh is requested:
    ```bash
    git -C "/tmp/git-peek/<owner>/<repo>" pull --ff-only || (rm -rf "/tmp/git-peek/<owner>/<repo>" && git clone --depth 1 <repo-url-or-slug> "/tmp/git-peek/<owner>/<repo>")
    ```
- **If not cached:**
  ```bash
  mkdir -p "/tmp/git-peek/<owner>"
  git clone --depth 1 <repo-url-or-slug> "/tmp/git-peek/<owner>/<repo>"
  ```

### 2. Inspect with available tools
Use your harness's search, directory listing, and file-reading tools on `/tmp/git-peek/<owner>/<repo>`:
- Check directory structure to understand architecture and entry points.
- Search for relevant symbols, function names, and configuration files.
- Read specific implementation files.

### 3. Answer
Answer the user's question, citing exact file paths and line numbers from the repository.

## Retention
Retain the cloned directory in `/tmp/git-peek/<owner>/<repo>` across turns so follow-up questions proceed without re-cloning. The OS `/tmp` cleans the cache across reboots, or remove with `rm -rf /tmp/git-peek/<owner>/<repo>` when explicitly requested.
