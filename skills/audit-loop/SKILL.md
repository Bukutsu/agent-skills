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

- **Run rule**: Continue through all rounds autonomously. Checkpoints record progress without pausing for approval; deliver the final summary only at the exit criterion or a genuine blocker. A user stop or finalize request overrides the loop: safely finish or revert the current owned change, report incomplete coverage, and never label an interrupted run CLEAN.
- **Context rule**: Keep context lean. Auditing is adversarial falsification, not passive reading. Search targeted patterns with grep or symbol queries, and read bounded slices (20-40 lines around boundaries and call sites). Dumping whole files into context exhausts attention and causes premature exit.
- **Evidence rule**: Time spent, files read, fixes landed, and passing tests are not evidence of a clean audit. Acceptance depends only on completed probes for the current round and revision. Preserve each check's exit status when filtering output (for example, use `pipefail` for shell pipelines); distinguish expected reproduction failures from checks that must pass.
- **Revision guard**: Before editing, staging, or accepting evidence, reconcile HEAD, the index, and working-tree state with the ledger. Any change to audited files, including concurrent work, invalidates all CLEAN verdicts. Preserve reports as history, mark perspectives pending, and finish the verified fix queue before a fresh round. Pause operations that overlap another actor's changes rather than overwriting or staging them.

### 1. Orientation & Perspective Selection
- List top-level entries, note domain and dependencies, and read the manifest-declared test command.
- **Toolchain readiness**: If any required toolchain or test runner is missing: ask the user to install it or request confirmation to let the agent set it up.
- Run tests to establish a green baseline.
- Select $N$ independent perspectives with non-overlapping boundaries from the reference list below. $N$ stays locked for the run.
- Start a compact round ledger: round number, commit and working-diff identity, locked perspectives, pending/completed reviews, planned versus executed probes, evidence, and unresolved findings. Keep audit artifacts (ledgers, reports, logs, and scratch reproducers) in permitted storage outside the repository by default. If using repository-local scratch storage, resolve the repository root, verify `git ls-files -- <path>` returns no tracked files, and verify `git check-ignore -- <path>` confirms every artifact path is ignored before writing. A tracked path or failed ignore check requires another location; never overwrite, untrack, or change shared ignore rules to make it usable. Repeat the tracked/ignored checks before staging and at exit, and stage fix files explicitly rather than using blanket adds. Intentional regression tests are deliverable source, not scratch artifacts. If no safe storage is available, emit checkpoint reports in the conversation.
- **Completion criterion:** Test suite green and $N$ named perspectives with defined audit scopes recorded in the ledger.

### 2. Review Codebase
Audit the repository across all $N$ selected perspectives:
- **Delegation authority:** Multi-perspective review is explicitly authorized to delegate. When a subagent or task tool (`subagent`, `task`) is declared in the environment, delegate independent reviews within its concurrency limit. Direct execution is reserved for unavailable delegation or recovery of a failed or incomplete reviewer.
- **Dispatching subagents:** Use the current tool schema to dispatch one reviewer per perspective, within the concurrency limit. Pass scope, round, revision, changed contracts, open questions, and the report schema below. Prefer fresh reviewer contexts for verification rounds when supported. Resume only with a returned child ID; omit optional resume fields for new reviewers. Keep handoffs neutral: omit CLEAN streaks, expected verdicts, and claims that verification is trivial. Prior coverage guides sampling but never exempts changed paths from review. Reviewers inspect and run safe probes; the parent owns repository edits and commits.
- **Dispatch recovery:** After a failure, inspect the error before retrying. Retry once with corrected arguments or a supported provider configuration; if still unavailable, finish affected perspectives directly. Respect provider access restrictions. A single child process is one reviewer, not a parallel multi-perspective audit.
- **Inspection recovery:** For repeated read, search, or probe-tool failures, inspect the error and retry once with corrected arguments or a smaller supported operation. If it still fails, use an available alternative or mark the probe blocked. Provider truncation and transport errors are failed operations, not inspected code; repeating them is not progress.
- **Collect results:** Track run and task IDs by round. Await all reviewers using the tool's supported wait or result mechanism. Retrieve full reports before accepting verdicts; completion notifications alone are not proof. Ignore late notifications from prior rounds. A failed or incomplete reviewer must be rerun or completed directly before the round can pass.
- **Direct sequential fallback:** When delegation is unavailable, review all perspectives sequentially. For reviewer recovery, complete only the failed or incomplete perspectives directly. Keep whole-repo scope within each perspective. Before inspecting a perspective, record its scope and failure hypotheses. Execute targeted queries and bounded reads, then record its full report before opening the next perspective. Keep review read-only until all reports are collected; queue findings rather than fixing them mid-review. Private reasoning or a report reconstructed at the end does not replace these checkpoints.
- **Same-context discipline:** A new heading is not an independent review. On re-audit, treat earlier verdicts and fix explanations as unverified leads. Probe how a fix could remain incomplete or break a caller, then inspect a fresh path outside the fixes. Judge code against its contract, not against the earlier rationale.
- For each perspective, formulate at least two concrete **failure hypotheses** (specific ways code could fail, drop errors, corrupt state, or leak resources) and actively probe them. Audit probes hunt for defects that the test suite misses; test passes confirm baseline only.
- **Reproduce hypotheses:** For each hypothesis, attempt the smallest safe probe that distinguishes correct behavior from failure. Use minimal malformed fixtures, controlled task ordering for races, and cleanup checks after cancellation or errors. Prefer local mocks or recorded fixtures for external services. Keep scratch probes outside tracked source and isolate side effects; live checks require authorization, and destructive reproduction stays off production.
- **Probe ownership:** Assign every required probe to a reviewer or the parent. Reviewers without execution tools return the command or fixture, expected result, and missing evidence for the parent to run. Keep that review pending until the result is collected or a blocker is reported.
- Record expected versus observed behavior and the command or fixture needed to repeat each probe. A failure to reproduce does not refute a hypothesis. When reproduction is impractical, record why and trace a reachable failing path with its violated contract, or an exact mechanism that prevents it. If neither is established, mark the hypothesis UNRESOLVED; it cannot support CLEAN.
- **Runtime evidence:** When authorized logs or telemetry are available, use redacted symptoms to prioritize hypotheses. Record their time window and deployed revision, or mark the revision unknown. Verify symptoms against the reviewed code; an older deployment's failure is not proof that current HEAD is defective. Keep private evidence out of repository artifacts.
- Each perspective produces its own report block:
```text
## <perspective>
Round / revision: <round number and commit; identify any uncommitted changes>
Scope: <boundaries covered>
Probes planned / executed: <each planned probe, owner, tool-result reference, expected vs observed outcome; pending items remain explicit>
Reproduction: <repeatable command or fixture per hypothesis, or why impractical and the code-path proof>
Hypotheses tested:
1. <failure hypothesis> -> [CONFIRMED finding | REFUTED by file:line mechanism | UNRESOLVED + missing evidence]
2. <failure hypothesis> -> [CONFIRMED finding | REFUTED by file:line mechanism | UNRESOLVED + missing evidence]
Findings:
- [CRITICAL | IMPORTANT | MINOR] file:line - violated contract and failing path -> fix
Optional improvements: <suggestions with no demonstrated defect, separate from unresolved hypotheses>
VERDICT: <NOT CLEAN + findings | INCOMPLETE + pending evidence | CLEAN + reason> (per Finding Schema below)
```
- Reconcile planned probes with executed results before accepting a report. An omitted probe remains pending; explicitly withdraw it only when code evidence shows it is inapplicable, not merely because other probes passed.
- A CLEAN verdict requires every applicable hypothesis to be refuted with an exact file:line citation and mechanism. A generic summary or listing passing test commands does not satisfy the step.
- **Completion criterion:** $N$ report blocks recorded for the same revision, each with executed probes, at least two hypotheses tested with code-level proof, and findings or CLEAN with reason. An unexecuted probe stays pending. A required probe blocked by the environment yields incomplete coverage, not CLEAN.

### 3. Fix & Commit
- **Verify findings:** Before editing, independently check each candidate's failing path and violated contract in the reviewed revision. Rerun the reviewer's reproducer where feasible; otherwise verify its code-path proof and state the reproduction gap. Verify library/runtime claims against official documentation or matching source. Record rejected candidates with code-level refutations. Reviewer confidence alone does not confirm a defect.
- Order fix queue by severity (`CRITICAL` first, then `IMPORTANT`, then `MINOR`). Resolve every finding autonomously; when trade-offs arise, pick the fix that best matches the surrounding code style and idioms while preserving caller contracts.
- Fix root causes in the shared path. Keep diffs surgical: every line traces to a finding. Preserve observable behavior.
- Validate each fix: show the same reproducer fails for the intended reason before the fix and passes afterward. Compare against actual pre-fix source; retain a regression test when practical, then compile, type-check, lint, and run the test suite. Record any remaining validation gap.
- **Isolated comparison:** When the checkout may have concurrent work, run pre-fix comparisons in a disposable checkout or worktree matching the recorded pre-fix commit and diff, with only the probe added. Keep it in permitted scratch storage under Step 1's artifact rules. Leave the active checkout and its stash untouched. If isolation is unavailable, report the reproduction gap and use code-path evidence; do not simulate the old behavior by changing the expected result.
- **Upgrade safety:** When touching persisted state, migrations, or versioned artifacts, check compatibility with existing installations, not only fresh test fixtures. Preserve applied migration bytes, including comments, when the migration system checksums them; use a new migration or separate documentation. Exercise the affected upgrade path in a disposable fixture where feasible; required upgrade checks that cannot run remain blockers.
- Commit each verified fix individually with an explanatory message (`fix: <cause and remedy>`).
- **Completion criterion:** Fix queue drained, all owned changes committed, and all tests passing. Preserve pre-existing or concurrent work; a clean-tree goal never authorizes discarding or committing someone else's changes.

### 4. Re-Audit & Exit
- After any fixes, increment the round and record the final commit and working-diff identity. Re-audit all $N$ perspectives using Step 2 before considering exit. Fix verification and rerunning the suite do not count as this review.
- If Round 1 produced zero findings across all $N$ perspectives: execute a **depth probe** on the highest-complexity module in each perspective (audit error paths, unwraps/panics, cancellation, or concurrency limits under stress). Round 1 exits only when depth probes refute failure hypotheses with concrete code citations.
- In subsequent rounds: execute at least two probes per perspective after the last fix. Include a regression or incomplete-fix hypothesis where changes affect that perspective, plus a fresh sample outside the fixes. For unaffected scopes, choose fresh failure hypotheses. Verify citations against the current source; previous citations and CLEAN verdicts are navigation aids, not evidence.
- If re-audit finds another defect, complete the review, return to Step 3, and restart all perspectives in a new round after fixing it.
- After compaction or resuming, recover the ledger and reconcile its revision with the current diff. Missing probe results remain pending. A revision mismatch invalidates prior verdicts.
- **Exit gate:** Check the ledger before composing the summary: all $N$ reports belong to the same current revision, every required probe has a tool result, planned probes are reconciled, citations match that revision, no findings, unresolved hypotheses, or pending reviews remain, and tests pass. After fixes, all accepted probes must postdate the last change. A failed gate returns to the pending work; effort already spent never waives it.
- **Exit criterion:** One complete round meets the exit gate with $N$ CLEAN-with-reason reports. This means no defects found in the exercised scope, not proof that the repository has no defects.
- **Blocked exit:** If required verification cannot run, report the exact blocker, completed coverage, and pending probes. Label the audit incomplete rather than CLEAN.
- **Summary**: State rounds run, fixes landed, test evidence, and any remaining gaps in plain language. Suggest a next command only when useful; completion does not require deployment or another user action.

---

## Reference

### Finding Schema
- **Confirmed defect:** evidence establishes a violated contract and reachable failing path. Add `[CRITICAL | IMPORTANT | MINOR] file:line - description -> recommended fix` to the fix queue.
  - `CRITICAL`: breaks a contract or produces wrong output.
  - `IMPORTANT`: latent bug, concurrency risk, or resource leak.
  - `MINOR`: lower-impact defect, such as misleading documentation or diagnostics.
- **Unresolved hypothesis:** missing evidence about a possible defect. Keep it pending with an owner; investigate or report INCOMPLETE. Calling it optional does not resolve it.
- **Optional improvement:** a preference or enhancement with no demonstrated defect. Record separately, leave outside the fix queue, and implement only if requested. It neither blocks CLEAN nor counts as a refuted hypothesis.
- **Verdict:** NOT CLEAN for confirmed defects; INCOMPLETE for pending evidence; CLEAN only when neither confirmed defects nor unresolved hypotheses remain, naming the refuted hypotheses and citing lines.

### Baseline Perspectives
- **Contract breaks**: callers, APIs, return types, error paths, unhandled status codes.
- **Data shape**: invariants, state ownership, schema validation, backward compatibility, atomic updates.
- **Explicit control**: swallowed errors (`.ok()`, `let _ =`, `unwrap_or_default`), hidden side effects, missing cancellation guards, silent fallbacks.
- **Resource lifecycle**: acquisition, teardown, handles, connection limits, unbounded channels/buffers, process kill groups.
