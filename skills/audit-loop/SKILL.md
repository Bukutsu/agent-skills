---
name: audit-loop
description: >-
  Autonomous iterative codebase audit and fix loop. Tailors N perspectives, resolves
  root causes with atomic commits, and loops until all perspectives report clean in
  one round. Trigger for repo-wide quality sweeps or deep multi-angle audits.
---

# Audit Loop

Audit an entire repository through $N$ project-tailored perspectives, resolve root causes, land each fix as its own ordinary commit, and repeat until all perspectives report clean in the same round. Runs autonomously without pausing for confirmation between rounds. Every fix stays inside existing intent: callers see the same behavior as before.

## Workflow

### 1. Orientation & Perspective Selection
- Inspect layout, domain, dependencies, and test suite. Run tests to establish a green baseline.
- Select $N$ independent perspectives with non-overlapping boundaries from the reference list below.
- **Completion criterion:** Test suite green and $N$ named perspectives with defined audit scopes selected.

### 2. Review Codebase
Audit the repository across all $N$ selected perspectives:
- **Subagent tools available:** Spawn $N$ parallel subagents concurrently (`subagent`, `task`), each reviewing whole-repo scope strictly within its perspective.
- **No subagent tools:** Review the repository directly in-session across each perspective in sequence. Keep whole-repo scope for each.
- Enforce the structured finding format from the reference section below.
- **Completion criterion:** All $N$ perspectives evaluated and findings collated.

### 3. Fix & Commit
- Order fix queue by severity (`CRITICAL` first). Park `DECISION` items for user review.
- Fix root causes in the shared path. Keep diffs surgical: every line traces to a finding. Preserve observable behavior.
- Validate each fix: compile, type-check, lint, and test suite.
- Commit each verified fix individually with an explanatory message (`fix: <cause and remedy>`).
- **Completion criterion:** Fix queue drained, working tree clean, and all tests passing.

### 4. Re-Audit & Exit
- Re-audit all $N$ perspectives against the updated codebase using the same mechanism as Step 2.
- **Exit criterion:** All $N$ perspectives return `VERDICT: CLEAN` in the same round with passing tests.
- **Summary**: Plain conversational English, zero jargon. State rounds run, fixes landed, test evidence, and open `DECISION` items. End with the single highest-leverage command or fix the user can run next.

---

## Reference

### Finding Schema
- `[CRITICAL | IMPORTANT | MINOR] file:line - description -> recommended fix`
  - `CRITICAL`: breaks a contract or produces wrong output.
  - `IMPORTANT`: latent bug, concurrency risk, or resource leak.
  - `MINOR`: cleanliness, debt, or clarity.
- `DECISION: description`: behavior or API changes beyond existing intent (reserved for human choice).
- `VERDICT: CLEAN`: perspective contains zero defects.

### Baseline Perspectives
- **Contract breaks**: callers, APIs, return types, error paths.
- **Data shape**: invariants, state ownership, schema validation.
- **Explicit control**: swallowed errors, hidden side effects, unhandled edge cases.
- **Resource lifecycle**: acquisition, teardown, handles, connection limits.
