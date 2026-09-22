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

- **Run rule**: Continue through all rounds autonomously and report only at the exit criterion or a genuine blocker. A user stop or finalize request overrides the loop: safely finish or revert the current owned change, report incomplete coverage, and never label an interrupted run CLEAN.
- **Context rule**: Keep context lean. Auditing is adversarial falsification, not passive reading. Search targeted patterns with grep or symbol queries, and read bounded slices (20-40 lines around boundaries and call sites). Dumping whole files into context exhausts attention and causes premature exit.

### 1. Orientation & Perspective Selection
- List top-level entries, note domain and dependencies, and read the manifest-declared test command.
- **Toolchain readiness**: If any required toolchain or test runner is missing: ask the user to install it or request confirmation to let the agent set it up.
- Run tests to establish a green baseline.
- Select $N$ independent perspectives with non-overlapping boundaries from the reference list below. $N$ stays locked for the run.
- **Completion criterion:** Test suite green and $N$ named perspectives with defined audit scopes selected.

### 2. Review Codebase
Audit the repository across all $N$ selected perspectives:
- **Delegation authority:** Multi-perspective review is explicitly authorized to delegate. When a subagent or task tool (`subagent`, `task`) is declared in the environment, delegate independent reviews within its concurrency limit. Direct execution is reserved for unavailable delegation or recovery of a failed or incomplete reviewer.
- **Dispatching subagents:** Use the current tool schema to dispatch one read-only reviewer per perspective, within the concurrency limit. Pass scope, round, revision, and the report schema below. Resume only with a child ID returned by that tool; omit optional resume fields for new reviewers. Reviewers inspect; the parent owns fixes and commits.
- **Dispatch recovery:** After a failure, inspect the error before retrying. Retry once with corrected arguments or a supported provider configuration; if still unavailable, finish affected perspectives directly. Respect provider access restrictions. A single child process is one reviewer, not a parallel multi-perspective audit.
- **Collect results:** Track run and task IDs by round. Await all reviewers using the tool's supported wait or result mechanism. Retrieve full reports before accepting verdicts; completion notifications alone are not proof. Ignore late notifications from prior rounds. A failed or incomplete reviewer must be rerun or completed directly before the round can pass.
- **Direct sequential fallback:** When delegation is unavailable, review all perspectives sequentially. For reviewer recovery, complete only the failed or incomplete perspectives directly. Keep whole-repo scope within each perspective. Use targeted pattern queries and bounded slice reads. Complete each perspective block before moving to the next.
- For each perspective, formulate at least two concrete **failure hypotheses** (specific ways code could fail, drop errors, corrupt state, or leak resources) and actively probe them. Audit probes hunt for defects that the test suite misses; test passes confirm baseline only.
- Each perspective produces its own report block:
```text
## <perspective>
Round / revision: <round number and commit; identify any uncommitted changes>
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
- **Verify findings:** Before editing, independently check each candidate's failing path and violated contract in the reviewed revision. Use a focused reproducer where feasible; verify library/runtime claims against official documentation or matching source. Record rejected candidates with code-level refutations. Reviewer confidence alone does not confirm a defect.
- Order fix queue by severity (`CRITICAL` first, then `IMPORTANT`, then `MINOR`). Resolve every finding autonomously; when trade-offs arise, pick the fix that best matches the surrounding code style and idioms while preserving caller contracts.
- Fix root causes in the shared path. Keep diffs surgical: every line traces to a finding. Preserve observable behavior.
- Validate each fix: compile, type-check, lint, and test suite.
- Commit each verified fix individually with an explanatory message (`fix: <cause and remedy>`).
- **Completion criterion:** Fix queue drained, all owned changes committed, and all tests passing. Preserve pre-existing or concurrent work; a clean-tree goal never authorizes discarding or committing someone else's changes.

### 4. Re-Audit & Exit
- Re-audit all $N$ perspectives against the updated codebase using the same mechanism as Step 2.
  - If Round 1 produced zero findings across all $N$ perspectives: execute a **depth probe** on the highest-complexity module in each perspective (audit error paths, unwraps/panics, cancellation, or concurrency limits under stress). Round 1 exits only when depth probes refute failure hypotheses with concrete code citations.
  - In subsequent rounds: re-probe changed files plus a fresh sample per perspective. Execute at least two probes per perspective again on the final revision; copied citations and previous CLEAN verdicts are navigation aids, not current-round evidence.
  - After compaction or resuming, recover the round, revision, pending reviewers, and unresolved findings. Reconcile them with the current diff before continuing.
- **Exit criterion:** The same round holds $N$ verified CLEAN-with-reason blocks (each with fresh probes run and code-cited hypothesis refutations) with passing tests.
- **Summary**: State rounds run, fixes landed, test evidence, and any remaining gaps in plain language. Suggest a next command only when useful; completion does not require deployment or another user action.

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
