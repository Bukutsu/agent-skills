---
name: audit-loop
description: >-
  Repeated read-only code audits of a repository or scope, looped until no new
  findings. Use for audit-until-clean requests and repo-wide quality sweeps.
---

# Audit loop

Audit the requested scope in rounds until the stop rule passes, then report the findings and ask whether to address them. The rounds are read-only: reviewers inspect code and run isolated probes. Code changes start only in Step 5, after the user says yes.

The parent runs the whole loop in one active turn. A finished reviewer report is input for the next step: collect it, reconcile it, and start the next round straight away. End the turn only when the stop rule passes, the user interrupts, or a blocker leaves no safe work.

## Inputs

- `scope` (optional): paths, subsystem, behavior, or concern. Default: the whole repository. Trace callers and dependencies only as far as an in-scope contract needs, and list unrelated discoveries separately.
- `stop` (optional): the user's stop rule, such as a round limit, a time budget, or one quiet round. Default: stop when the coverage map has no unprobed entry and the last two rounds were quiet, or when the map saturates. Any round that is not quiet resets the count.

Accept both in plain language or as `scope=...` and `stop=...`. Ask before starting when the scope is ambiguous or the stop rule is unmeasurable. The stop rule and round count stay with the parent.

## Terms

- **Round**: one report from every perspective at the same revision, then reconciliation.
- **Finding**: a reported problem with status confirmed, unresolved, or rejected.
- **Coverage map**: the required high-risk paths, contracts, and failure classes for each perspective, each marked unprobed, probed, excluded with a reason, or blocked. It grows during the audit.
- **Fresh ground**: map entries a round probes for failure that no earlier round probed.
- **Quiet round**: a complete round that covers fresh ground, adds no new Medium-or-higher finding, and leaves no finding unresolved. Low findings are recorded and reported without breaking quiet.
- **Saturated**: every entry is probed or excluded, and in one round no perspective could add and probe a new entry.
- **Ledger**: the parent's record of scope, stop rule, revision, coverage map, reviewer IDs, reports, and findings. Keep it in conversation checkpoints, with bulky logs in a unique run-owned directory outside the repository.

Read [recovery.md](references/recovery.md) when a reviewer or tool fails, the user interrupts, code changes during the audit, or the user offers runtime logs.

## 1. Baseline

1. Read [reviewer.md](references/reviewer.md). It holds the review standard, probe rules, and report format that every round uses, whether you review directly or brief reviewers.
2. Record the repository root, full HEAD, staged and unstaged diff identities, and relevant untracked files.
3. Read manifests, declared checks, and documented prerequisites. Ask before installing missing setup. Run applicable baseline checks and keep commands, exit statuses, and logs. Passing checks show readiness, not coverage.
4. Split the scope into project-specific perspectives and build the coverage map: each perspective's high-risk paths, contracts, and failure classes, all marked unprobed. A runtime limit excludes only the unavailable layer.
5. Start the ledger.

**Done:** scope, stop rule, baseline results, perspectives, coverage map, and ledger are recorded. A failing baseline is a finding or a blocker.

## 2. Run a round

Review every perspective at the same recorded revision.

- **Delegated**, only when the user explicitly asked for reviewers: create one reviewer per perspective once, using the initial prompt in `reviewer.md`, and keep its ID. Every later round resumes that same reviewer with the continuation prompt. The first round is **blind**; later rounds are **steered**, and the ledger records which. Wait for each full report; a completion notice is not a report.
- **Direct**: review perspectives one at a time in the current thread, following the initial prompt yourself and finishing each report before starting the next.

**Done:** every perspective has a report at the recorded revision, with source and failure-case probe evidence for each map entry it claims, or an explicit gap.

## 3. Reconcile

Verify each reported finding yourself: the violated contract and reachable failing path for a defect, or the demonstrated maintenance hazard for structural debt. Use the probe rules in `reviewer.md` for your own checks. Preferences that remove no case, path, or duplication are optional improvements.

Record one ledger entry per root cause: stable ID, kind (defect or structural debt), severity, locations, impact, trigger or maintenance task, evidence, counterevidence, durable remedy, and status. A finding stays unresolved until evidence settles it; passing checks elsewhere settle nothing.

Match duplicates by mechanism and affected contract, not wording or line number, and attach new evidence to the existing ID. A different failure path, wider impact, or higher severity counts as new discovery once verified. Judge remedies by the complexity of the resulting design rather than patch size, and keep a justified larger root-cause fix.

Before accepting reports, confirm HEAD, index, worktree, and relevant untracked files still match the recorded revision. If audited code changed, keep earlier findings, revalidate the affected ones, reset the stop count, and rerun the round at the new revision.

Update the coverage map: mark entries probed only where the report shows failure-case probe evidence, and add the new paths and failure classes reviewers reported. Re-reading or a happy-path run leaves an entry unprobed. Record which entries were fresh ground this round.

**Done:** every finding has a stable ID and status, the coverage map reflects this round's evidence, and all evidence matches the recorded revision.

## 4. Continue or report

Check the stop rule. If it has not passed, start the next round now: steer each reviewer with the continuation prompt, naming its noted findings and its unprobed map entries. When a perspective has no unprobed entry left, ask it to extend the map with new entries such as interactions between paths, combined failure classes, and boundaries shared with other perspectives, then probe them. Short progress notes between rounds are fine; the turn continues.

A complete round also needs a current report for every perspective, all planned probes resolved, and required checks accounted for.

A blocker that survives one supported correction ends the run as INCOMPLETE once independent safe work is done. A user interruption, round limit, or time budget stops new rounds, and the report lists what is pending. Keep the evidence needed to reproduce findings and delete other run-owned scratch.

The final report contains:
- Scope, revision, stop rule, stop reason, and rounds completed.
- Confirmed findings, deduplicated and ordered by severity, each with ID, locations, impact, evidence, and durable remedy.
- Unresolved findings, blockers, coverage limits, check results, and evidence locations.
- One verdict: **NOT CLEAN** if any confirmed finding remains; otherwise **INCOMPLETE** if required evidence is missing or the run was interrupted; otherwise **CLEAN**. Meeting a user stop rule leaves these conditions unchanged.

With confirmed findings, end with **"Do you want me to address them all?"** and wait for the answer. With only unresolved findings, ask whether to investigate the blockers. With none, say the scope is clean.

## 5. Address findings on approval

When the user says yes, address the confirmed findings they approved, in severity order, using each ledger entry's durable remedy.

- Recheck the revision first. If the code changed since the report, revalidate the affected findings before editing.
- Reproduce each defect with its recorded probe, make the change, and confirm the same probe now passes. For structural debt, show the duplicated path, bypassable invariant, or special case is gone.
- Delete the obsolete code, flags, and paths the remedy names, and keep a permanent test only where it guards a named regression the existing tests miss.
- Run the project's required checks, then report what changed per finding ID, check results, and anything left undone. Commit only when the user asks.
