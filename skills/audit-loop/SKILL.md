---
name: audit-loop
description: >-
  Autonomous iterative codebase audit and fix loop. Tailors N perspectives, resolves
  root causes with atomic commits, and loops until all perspectives report clean in
  one round. Trigger for repo-wide quality sweeps.
---

# Audit Loop

Audit an entire repository through $N$ project-tailored perspectives, resolve root causes, land each fix as its own ordinary commit, and repeat until all perspectives report clean in the same round. Runs autonomously without pausing for confirmation between rounds. Every fix stays inside existing intent: callers see the same behavior as before.

## Workflow

- **Run rule**: Continue through all rounds autonomously and report only at the exit criterion.
- **Context rule**: Keep context lean. Auditing is adversarial falsification, not passive reading. Search targeted patterns with grep or symbol queries, and read bounded slices (20-40 lines around boundaries and call sites). Dumping whole files into context exhausts attention and causes premature exit.

### 1. Orientation & Perspective Selection
- List top-level entries, note domain and dependencies, and read the manifest-declared test command.
- **Toolchain readiness**: If any required toolchain or test runner is missing: ask the user to install it or request confirmation to let the agent set it up.
- Run tests to establish a green baseline.
- Select $N$ independent perspectives with non-overlapping boundaries from the reference list below. $N$ stays locked for the run.
- **Completion criterion:** Test suite green and $N$ named perspectives with defined audit scopes selected.

### 2. Review Codebase
Audit the repository across all $N$ selected perspectives:
- **Delegation authority:** Multi-perspective review is explicitly authorized to delegate. When a subagent or task tool (`subagent`, `task`) is declared in the environment, parallel delegation is mandatory, not optional. Direct sequential execution is strictly a fallback when no subagent tool exists.
- **Dispatching subagents:** Spawn $N$ parallel subagent reviewers concurrently, one per perspective. Each reviewer receives whole-repo scope strictly within its assigned perspective, using targeted pattern queries and bounded slice reads. In Pi, compose parallel child runs in a single `subagent` call (`workflowScript` with `await runs.all([...])`). In other harnesses, call the environment's `subagent` or `task` tool concurrently.
- **Direct sequential fallback:** When no subagent tool exists in the environment, review one perspective at a time in sequence, whole-repo scope each. Use targeted pattern queries and bounded slice reads. Complete each perspective block before moving to the next.
- For each perspective, formulate at least two concrete **failure hypotheses** (specific ways code could fail, drop errors, corrupt state, or leak resources) and actively probe them. Audit probes hunt for defects that the test suite misses; test passes confirm baseline only.
- Each perspective produces its own report block:
```text
## <perspective>
Scope: <boundaries covered>
Probes run: <pattern searches and bounded checks executed>
Hypotheses tested:
1. <failure hypothesis> -> [CONFIRMED finding | REFUTED by file:line mechanism]
2. <failure hypothesis> -> [CONFIRMED finding | REFUTED by file:line mechanism]
Findings:
- [CRITICAL | IMPORTANT | MINOR] file:line - description -> fix
- VERDICT: CLEAN + reason (per Finding Schema below)
```
- A CLEAN verdict requires every tested hypothesis to be refuted with an exact file:line citation and mechanism. A generic summary or listing passing test commands does not satisfy the step.
- **Completion criterion:** $N$ report blocks present, each with probes run, at least two hypotheses tested with code-level proof, and findings or CLEAN with reason.

### 3. Fix & Commit
- Order fix queue by severity (`CRITICAL` first, then `IMPORTANT`, then `MINOR`). Resolve every finding autonomously; when trade-offs arise, pick the fix that best matches the surrounding code style and idioms while preserving caller contracts.
- Fix root causes in the shared path. Keep diffs surgical: every line traces to a finding. Preserve observable behavior.
- Validate each fix: compile, type-check, lint, and test suite.
- Commit each verified fix individually with an explanatory message (`fix: <cause and remedy>`).
- **Completion criterion:** Fix queue drained, working tree clean, and all tests passing.

### 4. Re-Audit & Exit
- Re-audit all $N$ perspectives against the updated codebase using the same mechanism as Step 2.
  - If Round 1 produced zero findings across all $N$ perspectives: execute a **depth probe** on the highest-complexity module in each perspective (audit error paths, unwraps/panics, cancellation, or concurrency limits under stress). Round 1 exits only when depth probes refute failure hypotheses with concrete code citations.
  - In subsequent rounds: re-probe changed files plus a fresh sample per perspective; prior-round probe lists do not count as fresh evidence.
- **Exit criterion:** The same round holds $N$ verified CLEAN-with-reason blocks (each with fresh probes run and code-cited hypothesis refutations) with passing tests.
- **Summary**: Plain conversational English, zero jargon. State rounds run, fixes landed, and test evidence. End with the single highest-leverage command the user can run next.

---

## Reference

### Finding Schema
- `[CRITICAL | IMPORTANT | MINOR] file:line - description -> recommended fix`
  - `CRITICAL`: breaks a contract or produces wrong output.
  - `IMPORTANT`: latent bug, concurrency risk, or resource leak.
  - `MINOR`: cleanliness, debt, or clarity.
- `VERDICT: CLEAN + reason`: zero defects found; reason names the refuted hypotheses and citing lines.

### Baseline Perspectives
- **Contract breaks**: callers, APIs, return types, error paths, unhandled status codes.
- **Data shape**: invariants, state ownership, schema validation, backward compatibility, atomic updates.
- **Explicit control**: swallowed errors (`.ok()`, `let _ =`, `unwrap_or_default`), hidden side effects, missing cancellation guards, silent fallbacks.
- **Resource lifecycle**: acquisition, teardown, handles, connection limits, unbounded channels/buffers, process kill groups.
