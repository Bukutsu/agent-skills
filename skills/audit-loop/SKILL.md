---
name: audit-loop
description: >-
  Audit and fix a repository through independent, project-specific perspectives.
  Use for repo-wide quality sweeps and repeated audits until evidence supports a
  clean round. Passing tests alone do not establish CLEAN.
---

# Audit Loop

Audit by trying to disprove the code's contracts. Continue autonomously through review, verified fixes, and fresh review until the exit gate passes or a genuine blocker or user interruption stops the run. Preserve existing intent and unrelated work. A read-only request limits the run to investigation and reports.

## 1. Orient and establish a baseline

- Resolve the repository root and full HEAD through Git. Record index and working-diff identity; preserve concurrent changes.
- Read manifests and declared check commands. Verify the active toolchain and documented prerequisites before running them. Missing prerequisites block dependent checks; request setup permission rather than installing automatically.
- Map major subsystems and lock independent perspectives with explicit boundaries. For each, decompose the requested scope into required production paths/contracts; mark each required, excluded with reason, or pending. Name the highest-risk paths and platform/runtime limits. A runtime limit may exclude that layer, not adjacent locally testable code. Record the complete requested perspective set at REVIEW START; never silently shrink it. A requested perspective without a report remains pending and makes the overall verdict INCOMPLETE.
- Create a run-owned evidence root outside the repository for the ledger, fixtures, source probes, build outputs, reports, raw logs, and exit statuses. Put temporary files under unique children; avoid fixed `/tmp` and pre-existing scratch paths. Record revision, scopes, hypotheses, probe owners, executed results, pending items, and findings. Track coverage by high-risk contract/path: unexamined, inspected only, probed (with receipt and limits), or blocked. Define required coverage from repository risks and the requested scope; any later exclusion needs a reason, not merely passing checks elsewhere. Conversation checkpoints are sufficient when safe writable storage is unavailable.
- Run baseline checks, retaining full logs and exit statuses. Baseline passes establish readiness, not audit coverage.

**Done:** baseline results and all locked scopes are recorded. A failed baseline is a finding to investigate or an explicit blocker, never CLEAN.

## 2. Review one perspective at a time

When a reviewer tool is available, delegate independent scopes and collect full reports. For delegation, interruptions, tool failures, malformed mocks, runtime symptoms, pre-fix comparisons, or persisted-state upgrades, read [recovery.md](references/recovery.md) before handling that branch. The parent owns edits and commits.

For direct review, finish this sequence before starting the next perspective:

1. Emit a **REVIEW START** checkpoint naming round, Git-resolved revision, scope, highest-risk path, and at least two concrete failure hypotheses.
2. Search targeted symbols and read bounded production spans. Trace callers and adapters where the contract crosses boundaries.
3. After the first targeted search and source read, attempt the smallest discriminating probe before expanding inspection. Prefer a standard-library probe when it exercises the real boundary. Run tools and tests from the project root unless a supported option changes it; test configuration may exclude external scratch paths. For third-party runners, inspect installed help/types and project configuration before choosing flags. After one corrected retry, use a documented alternative or record the probe blocked; do not guess further flags. Prefer malformed inputs, controlled task ordering, cancellation/error cleanup, and local service fixtures.
4. Choose probes from the failure hypotheses and required coverage gaps, not a novelty quota. Reuse an existing test when its inspected assertions discriminate the named failure through real production code; otherwise extend a run-owned probe with the smallest missing input, ordering, transition, or failure injection. Record expected versus observed behavior and bypassed layers. A filter matching zero intended tests supplies no evidence. Missing discriminating evidence makes the perspective INCOMPLETE; lack of a new test does not.
5. For custom probes, use throwing assertions or explicit failure statuses. Catch only expected application errors; keep assertions outside those catches and check the specific contract outcome. Assert the expected result and target execution unconditionally; do not hide checks behind conditionals or increment counts only when a condition happens to hold. If multiple outcomes are valid, name, assert, and record each. Add a valid control when malformed fixtures could explain the result. Read diagnostics even when exit is zero; a printed PASS label is not a result.
6. Reconcile every planned probe. A failure stays pending until the production contract establishes a reachable defect or the fixture is shown to be wrong. Correct fixtures with a recorded explanation; a different passing case leaves the original hypothesis unresolved.
7. Emit or save the perspective report below before proceeding. Direct reviews and probes are sequential: finish and report the current perspective before inspecting or probing the next. Parallelize only through independent delegated reviewers. Queue findings; keep this review read-only until all perspective reports are collected.

```text
REVIEW RESULT: <perspective>
Round / HEAD / diff identity: <full HEAD plus staged and unstaged patch hashes>
Scope: <subsystems examined, highest-risk path, unexamined boundaries>
Coverage: <contracts checked, reused or extended probes with rationale, receipts, remaining required coverage>
Required path evidence: <each mapped high-risk path -> source receipt -> probe receipt, or PENDING>
Hypotheses:
- <failure> -> CONFIRMED | REFUTED | UNRESOLVED
  Source receipt: <source-read tool result, exact production file:lines, mechanism>
  Probe receipt: <command/fixture, owner, tool-result/log path, expected vs observed>
  Limits: <mocked or bypassed layers, missing evidence>
Planned probes: <executed, pending, or withdrawn with code proof>
Findings: <severity, violated contract, reachable failing path, proposed fix>
Optional improvements: <non-defects, separate from findings>

VERDICT: NOT CLEAN | INCOMPLETE | CLEAN
```

Classify each perspective: **NOT CLEAN** requires a confirmed application defect with a reachable failing path. **INCOMPLETE** means unresolved hypotheses, missing receipts, pending probes, or blocked required checks, with no confirmed defect. **CLEAN** requires every applicable hypothesis resolved against inspected production code, all planned probes evidenced, and no findings or required coverage gaps. Keep optional improvements outside findings and verdicts.

**Blocked:** if a required probe, tool, or runtime is unavailable after one documented correction, record its error, owner, and missing result as pending; stop that probe and continue only independent safe work. For a blocked scoped run, record the unresolved hypothesis, actual blocker, evidence gathered, and INCOMPLETE verdict; do not start unrelated broad discovery. If remaining required coverage is blocked, exit promptly rather than retrying or leaving a summary-less turn.

**Done:** all locked perspectives have reports at one revision, every hypothesis has production-path evidence, every applicable probe is executed, and scope gaps are explicit. Every mapped high-risk path must have its own source and probe receipts; a listed risk without a probe remains pending regardless of nearby passing tests. One probe may cover multiple mapped paths when its assertions independently check each contract; map those receipts explicitly rather than duplicating the probe.

## Probe and evidence safety

- Use permitted storage outside the repository by default. Repository-local scratch requires both no tracked files (`git ls-files`) and ignored paths (`git check-ignore`) before writing, staging, and exit. Choose another location if either check fails; preserve shared ignore rules.
- Resolve writable paths before execution: generated metadata, build outputs, caches, and application state. Shared or symlinked artifacts can damage user builds even when Git stays clean.
- Verify each runner once with an uncaught deliberate failure. Invoke it without wrappers that mask failure and record its actual nonzero tool status. Capture command status before filtering or cleanup, and propagate required failures through the enclosing command; a printed status marker under exit 0 is not a receipt.
- Preserve compact ledgers, reports, fixtures, raw logs, and exit statuses through handoff. Keep them outside disposable build directories. After the final report, remove run-owned evidence when no longer needed unless the user requests retention; preserve project-owned and unrelated data. Bulky disposable outputs may be removed after capturing results.
- Use local fixtures rather than production or hardware side effects. Live or destructive checks require authorization. Resource exhaustion or unavailable runtimes leave required probes pending; after one supported recovery attempt, record the blocker and continue only independent safe work.

## 3. Verify and fix confirmed defects

Independently verify each candidate's violated contract and reachable failing path. Verify external library/runtime claims against authoritative documentation or matching source. Reject faulty probes with source evidence rather than changing expectations to force green.

Order confirmed defects by severity. Make surgical root-cause fixes inside existing intent. Show the same reproducer fails for the intended reason before the fix and passes afterward. Before retaining a permanent test, name its protected behavior, credible regression, and gap in existing protection. Prefer extending the owning test; another layer needs a distinct transport, lifecycle, or other risk. Use independent expectations and real production behavior, with mocks outside the behavior under test. Keep probes in the evidence root unless they earn permanent retention; avoid production seams used only by tests. Preserve existing assertions unless the contract changed or source evidence establishes a faulty test, and explain that change. Run focused checks while iterating and required compile, type, lint, and test checks before handoff, stating limits. Commit each verified fix separately when authorized; explicitly stage owned files.

For each fix, record its blast radius: changed contracts, direct callers, and indirect dependencies through shared state, serialized formats, adapters, lifecycle timing, or platform-specific behavior. Trace beyond symbol matches until affected consumers are accounted for. State the invariant that keeps each affected contract safe and probe real production code across the relevant boundary; an isolated helper test covers only that helper. Record bypassed layers and blocked checks. Blast-radius validation informs the next review; it does not replace fresh coverage.

Reconcile HEAD, index, and worktree before edits, staging, or evidence acceptance. Changes to audited code invalidate all CLEAN verdicts. Preserve other actors' work and pause overlapping operations.

**Done:** verified fix queue drained, owned changes committed when authorized, required checks pass, and remaining gaps are explicit. In read-only mode, report confirmed findings and stop without fixes.

## 4. Fresh review and exit

After any fix, start a new round at the observed final revision and repeat Step 2 for every perspective, including a separate current-round report for each. Read production paths and relevant test assertions afresh. In affected scopes, test incomplete-fix/regression hypotheses and the recorded blast radius.

Each later round prioritizes changed contracts, blast-radius risks, and unresolved required coverage. Reuse discriminating fixtures and tests, reread their production paths and assertions, and rerun them at the final revision. Add or deepen a probe only for a named evidence gap; when required coverage is complete, further novelty is not an exit requirement. Prior reports guide navigation but do not supply current-round results.

Before accepting a no-finding round, check that each perspective's highest-risk path has a discriminating failure-case receipt, not merely a happy-path run. Supply missing evidence with the smallest targeted probe. Tests, fix verification, time spent, and report headings do not replace production-path review.

Recover the ledger after compaction or resuming; reconcile revision and diff. Missing results remain pending. Before exit, verify:
- All locked reports match the same full HEAD and staged/unstaged diff hashes.
- Source receipts point to inspected production mechanisms, not merely tests or unrelated lines.
- All planned probes are reconciled, custom runner controls detect failure, and required checks pass.
- After fixes, accepted reads and probes postdate the last change.
- Each perspective has a separate current-round report with discriminating coverage receipts and reasons for probe reuse or extension; a summary ledger label cannot substitute for it.
- Reconcile each mapped high-risk path against its report's source and probe receipts. A path named as highest risk but lacking a probe is pending; unexamined, inspected-only, or blocked required coverage prevents CLEAN.
- Findings, unresolved hypotheses, and blast-radius checks are reconciled. Round count or a round with zero findings is not an exit criterion.

Choose exactly one overall verdict: **NOT CLEAN** if any perspective has a confirmed defect; otherwise **INCOMPLETE** if any perspective, required boundary, probe, or check is incomplete; otherwise **CLEAN** if all perspectives pass their gates. An interrupted run cannot be CLEAN. CLEAN means no defects found in the exercised scope, not proof that the repository is defect-free.

If a gate fails, continue its pending work; if genuinely blocked, emit INCOMPLETE with the exact blocker. Every exit includes a nonempty report: verdict, revision, rounds, fixes, checks, evidence location, and limitations. A final tool call alone is not a handoff.
