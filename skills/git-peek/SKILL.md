---
name: git-peek
description: >-
  Verify external contracts from matching-version source or authoritative docs
  before implementation. Use whenever implementing, configuring, or fixing code
  that depends on an external library, SDK, API, service, framework, CLI, build
  tool, or system interface, even when the user does not ask for research.
  Also use for repository URLs or owner/repo, dependency tracing, and supported
  usage or prerequisite questions. Reuse this workflow while it remains in context.
---

# Git Peek

Use the smallest authoritative lookup that resolves the question. For open-source implementation questions, prefer matching-version source and examples. Keep cached repositories read-only except for authorized refreshes.

Read this workflow once while it remains in context. Reuse verified evidence while the version and contract still apply; reload after relevant context loss or a skill change.

## 1. Resolve the contract and evidence

Identify the behavior or interface needed and the version in use from lockfiles, installed metadata, or tool version output. Check relevant prerequisites before executing. Internal code questions start with the existing implementation and conventions.

Choose the lightest sufficient source:
- Installed source, types, headers, `--help`, or man pages when they answer the question directly.
- A matching upstream release or commit for deeper open-source tracing. Use the cache workflow below; default-branch behavior is not evidence for a different installed version.
- Official versioned documentation or specifications for closed-source services, unsupported/internal API questions, or gaps in source evidence.

Documentation establishes supported guarantees; implementation shows actual behavior. Resolve discrepancies explicitly and keep private implementation details out of integrations unless that dependency is intentional and authorized. A source trace need not be followed by redundant documentation reading when the contract is already established.

**Complete when:** target version, question, evidence source, and prerequisites are identified, or missing information is reported. If no repository is needed, skip to inspection.

## 2. Resolve source and cache

Resolve the clone URL, host, owner, repository, and any requested branch, tag, or commit. Expand owner/repo to a clone URL for the intended host. Include the host in the cache key to separate identically named repositories.

Choose a permitted cache root:
- Default: `${XDG_CACHE_HOME:-$HOME/.cache}/git-peek`.
- Workspace-confined harness: `<workspace>/.git-peek`. Use this directly when outside access is restricted; respect the sandbox rather than seeking broader access just for caching.

For a cache inside a Git repository, resolve the repository root and exclude path with `git rev-parse --show-toplevel` and `git rev-parse --git-path info/exclude`. Add the cache's root-relative ignore pattern once and verify it with `git check-ignore`. In worktrees, the exclude file may be outside the permitted workspace: if it is inaccessible, request permission or an approved ignore location before cloning. Outside Git, no ignore setup is needed.

Set the cache path to `<cache-root>/<host>/<owner>/<repo>`. Treat URL components as path segments, rejecting traversal or paths outside the selected root. An older cache may be reused after verifying its origin and requested revision; migration or deletion requires authorization.

**Complete when:** source, requested revision, permitted cache path, and any required ignore rule are resolved.

## 3. Reuse or clone

- Cache hit: verify it is a Git repository and its origin matches the requested source. Use its current revision for ordinary inspection.
- Cache miss: create its parent directory and run `git clone --depth 1 <clone-url> <cache-path>` with quoted, resolved arguments.
- Freshness request: when asked for latest/current source or an update comparison, fetch the relevant ref. Otherwise refresh only when asked. Preserve local modifications and avoid changing a checkout another session is using; use a separate revision-specific checkout when needed.
- Refresh failure: preserve the cache and report the error. An old checkout may support a clearly labeled historical answer, not a claim about current source. A failed pull never authorizes deleting the cache.

For a requested branch, tag, or commit missing from the shallow clone, fetch that ref. Resolve the requested or freshly fetched ref to a commit and inspect it with revision-qualified reads such as `git show <commit>:<path>` or a separate revision-specific checkout. Fetching alone does not select the inspected revision. Leave the shared checkout unchanged and cite the resolved commit; use `git rev-parse HEAD` only when inspecting the default cached checkout.

**Complete when:** the source is verified and inspection targets the resolved requested or refreshed commit, or the identified default checkout for ordinary inspection.

**Blocked:** report retrieval failures and pause claims that require unavailable source.

## 4. Inspect and apply

Trace the relevant public entry point into its implementation and callers using targeted searches and bounded reads. Consult examples where useful. Establish the inputs, outputs, errors, and constraints needed for the task. Exclude cached foreign source from searches of the user's own project.

When implementation is authorized, use supported entry points and the contracts established above. Follow configuration to actual calls or payloads when relevant. Keep scratch work outside cached source and the active checkout. Inspection alone does not authorize edits, installs, or live side effects.

Cite repository paths, line numbers, and the inspected commit, or the authoritative URL/installed definition and version. Distinguish supported contracts, observed implementation, and inference. If the target is absent or evidence unavailable, report the search scope and limits rather than guessing.

**Complete when:** the requested answer or implementation follows the inspected contract, with relevant uncertainty and limits explicit.

## Retention

Reuse the cache across sessions. Remove cached repositories only when explicitly requested.
