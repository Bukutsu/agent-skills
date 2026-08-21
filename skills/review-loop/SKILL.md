---
name: review-loop
description: >-
  Iterative multi-perspective codebase audit and fix loop. Analyzes the project,
  spawns N tailored parallel subagent reviewers across the entire repository, fixes
  findings at the root cause, commits each verified round, and loops until all
  reviewers report clean in a single round. Use for whole-codebase audits, deep
  quality passes, or repo-wide cleanups.
---

# Review Loop

Audit an entire repository through $N$ project-tailored perspectives, resolve root causes, commit verified rounds, and repeat until all reviewers report clean in the same round.

## Workflow

### 1. Orientation & Perspective Selection
- Inspect the codebase layout, domain, dependencies, and test suite to establish a green baseline.
- Choose $N$ independent review perspectives tailored to this project's stack and architecture (e.g. API contracts, data invariants, concurrency safety, simplicity/bloat, performance, security).
- Ensure chosen perspectives have distinct, non-overlapping audit boundaries.

### 2. Dispatch Reviewers
Spawn $N$ parallel subagents (or run sequentially if subagents are unavailable). Each reviewer receives whole-repo scope and audits strictly within their assigned perspective.

Each subagent prompt must enforce a structured return format:
- **If issues found:** `[CRITICAL | IMPORTANT | MINOR] file:line - description -> recommended fix`
- **If clean:** exactly `VERDICT: CLEAN`

### 3. Fix & Commit Round
- Collate all findings across reviewers and sort by severity (`CRITICAL` first).
- Fix root causes directly; avoid adding wrapper layers or cosmetic workarounds.
- Verify fixes: compile, type-check, lint, and run the test suite.
- **Commit verified progress:** Commit the round's fixes with a message stating the problems solved (e.g. `fix(audit): resolve null session teardown and remove single-use config factory (round 1)`).

### 4. Re-Audit & Exit
- Re-dispatch all $N$ reviewers against the updated codebase.
- **Exit criterion:** Complete the loop only when **all $N$ reviewers return `VERDICT: CLEAN` in the same round** with all tests passing.
- If goal tracking is active, mark complete only after the clean round is verified with evidence.
