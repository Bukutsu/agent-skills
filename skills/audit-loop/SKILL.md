---
name: audit-loop
description: >-
  Iterative multi-perspective codebase audit and fix loop. Analyzes the project,
  spawns N tailored parallel subagent reviewers across the entire repository, fixes
  findings at the root cause, commits each fix on its own, and loops until all
  reviewers report clean in a single round. Use for whole-codebase audits, deep
  quality passes, or repo-wide cleanups.
---

# Audit Loop

Audit an entire repository through $N$ project-tailored perspectives, resolve root causes, land each fix as its own ordinary commit, and repeat until all reviewers report clean in the same round. Every fix stays inside the code's existing intent: callers see the same behavior they saw before.

## Workflow

### 1. Orientation & Perspective Selection
- Inspect the codebase layout, domain, dependencies, and test suite to establish a green baseline.
- Choose $N$ independent perspectives with distinct, non-overlapping boundaries, tailored to this stack and the project's stated rules. Strong defaults drawn from universal engineering practice: contract breaks (callers, APIs, error paths), data shape (invariants, state ownership), explicit control (hidden magic, swallowed errors, deep nesting), and resource lifecycle (who owns and releases what).

### 2. Dispatch Reviewers
Spawn $N$ parallel subagents (or run sequentially if subagents are unavailable). Each reviewer receives whole-repo scope and audits strictly within their assigned perspective.

Each subagent prompt must enforce a structured return format:
- **Defects against existing intent:** `[CRITICAL | IMPORTANT | MINOR] file:line - description -> recommended fix`. Severity: `CRITICAL` breaks a contract or produces wrong output; `IMPORTANT` is a latent bug or risk; `MINOR` is cleanup.
- **Ideas that would change how the code works** (public interfaces, observable behavior, product scope): `DECISION: description`. These belong to the user; reviewers report them instead of recommending fixes.
- **No defects:** exactly `VERDICT: CLEAN`

### 3. Fix & Commit
- Collate all findings across reviewers and sort by severity (`CRITICAL` first). Park `DECISION` items for the user; they leave the fix queue.
- Fix root causes in the shared path rather than patching the reported symptom. Keep diffs surgical: every changed line traces to a finding.
- Each fix preserves observable behavior: same inputs, same outputs, same caller expectations. When priorities conflict, order them correctness, then performance, then simplicity, then style.
- Verify before landing: compile, type-check, lint, test suite. A failing fix gets reworked before it lands.
- **One commit per fix:** split unrelated fixes into separate commits so history reads like normal development. The message names the change and why it was needed, as a person would when refactoring (e.g. `fix: tear down sessions before closing the pool`).

### 4. Re-Audit & Exit
- Re-dispatch all $N$ reviewers against the updated codebase.
- **Exit criterion:** Complete the loop only when **all $N$ reviewers return `VERDICT: CLEAN` in the same round** with all tests passing.
- Final summary (bro rules): restate the outcome in plain human language. Stop using jargon and speak coherently. State it more simply and concisely, like one human talking to another. Cover: rounds run, fixes landed, validation evidence, open `DECISION` items.
- Biggest fix you can try right now: end with one highest-leverage next step the user can do immediately, with the exact file or command to try.
