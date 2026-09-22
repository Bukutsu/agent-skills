---
name: git-peek
description: >-
  Inspect external Git repositories when given a repository URL or owner/repo,
  or asked about remote source structure or symbols.
---

# Git Peek

Reuse shallow clones for source inspection. Keep cached repositories read-only except for authorized refreshes.

## 1. Resolve source and cache

Resolve the clone URL, host, owner, repository, and any requested branch, tag, or commit. Expand owner/repo to a clone URL for the intended host. Include the host in the cache key to separate identically named repositories.

Choose a permitted cache root:
- Default: `${XDG_CACHE_HOME:-$HOME/.cache}/git-peek`.
- Workspace-confined harness: `<workspace>/.git-peek`. Use this directly when outside access is restricted; respect the sandbox rather than seeking broader access just for caching.

For a cache inside a Git repository, resolve the repository root and exclude path with `git rev-parse --show-toplevel` and `git rev-parse --git-path info/exclude`. Add the cache's root-relative ignore pattern once and verify it with `git check-ignore`. In worktrees, the exclude file may be outside the permitted workspace: if it is inaccessible, request permission or an approved ignore location before cloning. Outside Git, no ignore setup is needed.

Set the cache path to `<cache-root>/<host>/<owner>/<repo>`. Treat URL components as path segments, rejecting traversal or paths outside the selected root. An older cache may be reused after verifying its origin and requested revision; migration or deletion requires authorization.

**Complete when:** source, requested revision, permitted cache path, and any required ignore rule are resolved.

## 2. Reuse or clone

- Cache hit: verify it is a Git repository and its origin matches the requested source. Use its current revision for ordinary inspection.
- Cache miss: create its parent directory and run `git clone --depth 1 <clone-url> <cache-path>` with quoted, resolved arguments.
- Freshness request: when asked for latest/current source or an update comparison, fetch the relevant ref. Otherwise refresh only when asked. Preserve local modifications and avoid changing a checkout another session is using; use a separate revision-specific checkout when needed.
- Refresh failure: preserve the cache and report the error. An old checkout may support a clearly labeled historical answer, not a claim about current source. A failed pull never authorizes deleting the cache.

For a requested tag or commit missing from the shallow clone, fetch that ref. Record the commit actually inspected with `git rev-parse HEAD`.

**Complete when:** the verified clone contains the requested source at an identified revision.

**Blocked:** report retrieval failures and pause claims that require unavailable source.

## 3. Inspect and answer

Search relevant symbols and read bounded slices around definitions and callers. Exclude cached foreign source from searches of the user's own project.

Answer with repository paths, line numbers, and the inspected commit. Distinguish code-backed facts from inference. If the target is absent, cite the search scope and limits instead of inventing a location.

**Complete when:** source claims have traceable evidence and missing or stale evidence is explicit.

## Retention

Reuse the cache across sessions. Remove cached repositories only when explicitly requested.
