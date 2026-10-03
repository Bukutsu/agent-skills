---
name: docs-first
description: >-
  Ground external work in authoritative documentation, specifications, or installed definitions
  before writing code or running commands. Never guess from memory or trial-and-error.
  Use whenever a task touches external libraries, SDKs, APIs, third-party services,
  frameworks, CLI tools, package managers, build systems, or operating system/desktop configurations.
  Triggers include: calling or integrating APIs, auth, payments, or external services;
  configuring frameworks or software conventions; adding, updating, or building dependencies
  and toolchains; troubleshooting unfamiliar tools, device/system settings, or runtime errors;
  or whenever asked to check docs, follow conventions, or look up how something works.
---

# Docs First

LLM weights are ungrounded priors. Ground external APIs, CLI flags, version constraints, and framework conventions in primary documentation or installed definitions before writing code or running commands.

## Workflow

### 1. Resolve & Audit
- Check project manifests (`package.json`, `Cargo.lock`, `requirements.txt`) or tool `--version` for the exact target version.
- Audit all required headers, runtime tools, and permissions against the active environment in one pass before executing.
- **Complete when:** target version is resolved and all prerequisites are verified or reported as a single consolidated list.

### 2. Fetch Ground Truth
Choose the most direct authoritative source:
- **Local clone via `git-peek` (preferred for open source)**: When the tool, library, or framework has an open repository, shallow-clone it or its documentation at the matching tag using `git-peek` (`~/.cache/git-peek/...`). Run `rg` across its `docs/`, `examples/`, and test suites. Local Markdown and real working examples beat truncated web scrapes.
- **Installed definitions**: Inspect local `.d.ts`, `.pyi`, C/C++ headers, crate source, `--help`, or `man` pages in the active environment.
- **Online fetch & specifications**: For cloud APIs or proprietary portals without a public Git repository, fetch versioned docs or `llms.txt` via web search or browser.
- **Complete when:** every external symbol, flag, parameter, and native schema is confirmed from primary sources.

### 3. Apply & Isolate
- Route commands through canonical framework CLI entry points rather than raw compilers that bypass packaging or code generation.
- Trace configuration values from application parser to serialized request payload or command arguments.
- Keep generated metadata and scratch files separate from the active checkout.
- Preserve check exit codes (`$?`) before displaying excerpts or cleanup.
- **Complete when:** implementation matches verified contracts and checks exercise real application paths.

### 4. Cite Evidence
- Cite the authoritative doc URL, repository path, installed type path, man page, or header in the final response alongside verified version.
- **Complete when:** all external contracts trace to inspected sources.
