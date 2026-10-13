# Audit recovery and special cases

Apply only the branch that occurs. Keep the ordinary review and exit gates in `../SKILL.md`.

## Interruption

- Stop scheduling work when the user asks to stop or finalize. Safely finish or revert only the run's owned in-progress change.
- Request cancellation or send stop-only instructions to outstanding reviewers through the supported interface. Check their status once where supported and record each reviewer ID as finished, stopped, or stop-unconfirmed, including interface limits.
- Record late reports as partial evidence. Do not restart the audit unless the user resumes it. An interrupted run is never CLEAN; report completed coverage and pending work.

## Delegation

- **Delegation authority:** Multi-perspective review is explicitly authorized to delegate. When a subagent or task tool (`subagent`, `task`) is declared in the environment, delegate independent reviews within its concurrency limit. Direct execution is reserved for unavailable delegation or recovery of a failed or incomplete reviewer.

- **Dispatching subagents:** Use the current tool schema to dispatch one reviewer per perspective, within the concurrency limit. Pass scope, round, revision, changed contracts and blast radius, remaining high-risk coverage, open questions, and the report schema in `../SKILL.md`. Reviewers follow Step 2's evidence and probe requirements. Prefer fresh reviewer contexts for verification rounds when supported. Resume only with a returned child ID; omit optional resume fields for new reviewers. Keep handoffs neutral: omit CLEAN streaks, expected verdicts, and claims that verification is trivial. Prior coverage guides sampling but never exempts changed paths from review. Reviewers inspect and run safe probes; the parent owns repository edits and commits. Before dispatch, assign every required probe. If a reviewer lacks execution tools, have it return the command or fixture, expected result, and missing evidence; the parent runs it or keeps the perspective pending.

- **Dispatch recovery:** After a failure, inspect the error before retrying. Retry once with corrected arguments or a supported provider configuration; if still unavailable, finish affected perspectives directly. Respect provider access restrictions. A single child process is one reviewer, not a parallel multi-perspective audit.

- **Inspection recovery:** For repeated read, search, or probe-tool failures, inspect the error and retry once with corrected arguments or a smaller supported operation. If it still fails, use an available alternative or mark the probe blocked. Provider truncation and transport errors are failed operations, not inspected code; repeating them is not progress.

- **Collect results:** Track run and task IDs by round. Await all reviewers using the tool's supported wait or result mechanism. Retrieve full reports before accepting verdicts; completion notifications alone are not proof. Ignore late notifications from prior rounds. A failed or incomplete reviewer must be rerun or completed directly before the round can pass.

- **Fixture recovery:** When a mock fails, trace its inputs, emitted requests, and consumed responses against the real contract before correcting it. Preserve the original result and explain the correction. A different passing case covers only that case; the original hypothesis still needs its own resolution.

- **Runtime evidence:** When authorized logs or telemetry are available, use redacted symptoms to prioritize hypotheses. Record their time window and deployed revision, or mark the revision unknown. Verify symptoms against the reviewed code; an older deployment's failure is not proof that current HEAD is defective. Keep private evidence out of repository artifacts.

- **Isolated comparison:** When the checkout may have concurrent work, run pre-fix comparisons in a disposable checkout or worktree matching the recorded pre-fix commit and diff, with only the probe added. Apply Step 1's artifact rules and Step 2's probe isolation to both source and generated outputs. Leave the active checkout and its stash untouched. If isolation is unavailable, report the reproduction gap and use code-path evidence; do not simulate the old behavior by changing the expected result.

- **Upgrade safety:** When touching persisted state, migrations, or versioned artifacts, check compatibility with existing installations, not only fresh test fixtures. Preserve applied migration bytes, including comments, when the migration system checksums them; use a new migration or separate documentation. Exercise the affected upgrade path in a disposable fixture where feasible; required upgrade checks that cannot run remain blockers.
