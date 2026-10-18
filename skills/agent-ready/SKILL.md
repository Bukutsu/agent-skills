---
name: agent-ready
description: Cold-start the current repo as a new agent would, then fix what blocks it. Use for "make this repo agent-friendly/agent-ready" and agent-friendliness audits.
---

# Agent ready

Work the repo as a cold-start agent. Run each path for real, then fix the friction in the repo. Output is repo edits and a short summary.

## Cold start

Each step is done when its command has been run and the real result observed, not inferred from docs, mtimes, or "it compiles". When a check fails, suspect your observation method before the repo.

1. **Setup**: find the install and run commands, then run them from a clean state. Ready means usable, not just a process answering. Record undeclared tools, versions, env vars, secrets. Commands run headless (flags instead of prompts), errors name the fix as a working invocation, and a re-run is safe.
2. **Verify**: find the *tight* loop, one fast command that checks a small change. It goes *red* on a real bug. Test names and dry-run flags prove nothing; observe what they actually do.
3. **Docs**: run each documented command literally. It works and does what the docs say.
4. **Context**: knowledge lives in the repo, and nothing burns context needlessly (giant files, dead code, unexplained generated dirs, layers with one caller).
5. **History**: read recent commits, reverts, and review comments. A mistake that happened twice is a class.

## Fix

Smallest change first, proceeding without asking on reversible edits; confirm irreversible ones. Re-run each fix and see it pass.

- Wrong or missing command: correct it where documented.
- Each mistake class, at the highest level that works: architecture (one owner per state, one way per task, delete the old way) > types > a lint or check whose error names the fix > a behavior test > docs last. Prove each new check fails on a real past mistake.
- Entry doc (`AGENTS.md`) is a *cache*: only what no command or file reveals (unwritten conventions, reasons, gotchas). Create it only when absent.
- No tight loop: add one script to the existing task runner.
- Product code, refactors, new dependencies, extra tooling: list for the user with the history evidence behind each. Settle open questions by trying them, not by asking.

## Finish

Kill only processes you started. Leave no scratch files or reports.

Reply with three short lists: fixed, left for the user, unverified. For each claim say how far proof went: read it, reasoned it, or ran it. Name blockers you cannot reach (secrets, accounts, infrastructure) and what a stand-in run did not cover.
