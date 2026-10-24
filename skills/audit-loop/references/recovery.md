# Audit recovery

Read the section that matches the situation. The main loop is in `../SKILL.md`.

## Reviewer failure

- Inspect a dispatch or resume error and retry once with corrected arguments.
- If a reviewer is lost, cannot be resumed, or the harness requires a new task per round, record the blocker, keep its reports and open coverage, and ask the user whether to continue directly or allow a replacement reviewer. Stay within the harness's lifecycle rules; another thread interface is not a substitute.
- If a reviewer cannot run probes, take its proposed command, fixture, and expected result, then run it yourself or leave the evidence pending. A missing report leaves the round incomplete.

## Tool failure

- Inspect the error and retry once with a corrected or smaller supported operation. For third-party runners, read installed help, types, and project configuration before choosing flags. After that, use a documented alternative or mark the probe blocked.
- Truncated output and transport errors are failed operations, not inspected code. Record the error and the missing result.

## Runtime symptoms

When the user offers logs or telemetry, use redacted symptoms to prioritize hypotheses. Record the time window and deployed revision, or mark it unknown, and confirm the mechanism in current code before recording a finding.

## Interruption

- Stop scheduling rounds and request cancellation of outstanding reviewers through the supported interface. Check each once and record it as finished, stopped, or stop-unconfirmed.
- Keep late reports as partial evidence. Report with the pending work listed; an interrupted run is INCOMPLETE.

## Code changes during the audit

Step 3 of `../SKILL.md` covers revalidation. Treat the change as someone else's work: pause overlapping operations, and leave reverting, stashing, and repair to its owner.
