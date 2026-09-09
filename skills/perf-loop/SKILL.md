---
name: perf-loop
description: >-
  Autonomous performance and resource optimization loop. Use when asked to
  optimize speed, profile bottlenecks, or reduce memory usage across a target or
  the entire project.
argument-hint: "[target area / module / flow] (optional: omit to scan entire project)"
---

# Perf Loop

## Execution rule

Run the cycle autonomously and preserve correct behavior. After every attempt, immediately take the next action shown by the cycle. The next user-facing response after setup begins is either a setup blocker or the final report. Treat a saved improvement, failed attempt, baseline, and completed part as intermediate states.

## Setup

1. Create a dedicated branch from a clean base.
2. Check that the required build, test, and measurement tools are available. If one is missing, ask the user to install it or approve installation.
3. Run the tests. If they fail before any changes, record the failing tests and ask the user whether to repair them or use those failures as the known baseline.
4. Choose the discovery path:
   - **Target specified:** Treat the named module, route, or flow as the scope boundary. Trace every stage of its runtime path through calls, data changes, and resource use. Follow costly calls into deeper layers until each cost belongs to a concrete operation, then divide the target into measurable parts.
   - **No target specified:** Map every discovered entry point and major component, the calls and data flows between them, and their computation, memory, storage, and network use. Derive measurable parts from the map. Account for every discovered entry point and major component before ranking.
5. Rank the parts by measured cost when measurements exist; otherwise rank by estimated `frequency * cost`. Select the highest-cost part.
6. Create a unique untracked run directory at `.perf/<run-id>/`. Create its `ledger.tsv` with this header:
   `attempt\tpart\tidea\tcommit\tmetric_before\tmetric_after\tdelta\tstatus\tevidence`
   Keep every part from this run in that ledger. Write benchmark output to a separate `<part>.log` file in the same directory.
7. Add the smallest reliable benchmark or `[PERF-PROBE]` measurements needed. Commit this measurement setup separately.
8. Run the benchmark several times with warmup and record the median as the selected part's baseline.

**Setup is complete when:** the test baseline is known; every stage in the specified target or every discovered entry point and major component in the codebase is accounted for; each candidate maps to a measurable part; the parts are ranked; and a repeatable benchmark has recorded the highest-cost part's baseline median in this run's ledger.

## Cycle

For the selected part:

1. **Find** — Measure the selected part, follow its costly call or data path into deeper layers, and stop at the concrete function, query, loop, allocation, conversion, or I/O operation responsible for the largest measured share.
2. **Try** — Make one small change based on one untried item from the Optimization Order.
3. **Check** — Run the tests and benchmark with a timeout of twice the baseline runtime. Write output to this part's log; inspect only extracted metrics or its last 40 lines.
4. **Decide**:
   - Tests match the test baseline and performance improves by at least 5%: record `keep`, commit the change, update the performance baseline, and reset failures to zero.
   - Tests match the test baseline, performance stays within 1%, and the measured path has fewer branches, allocations, queries, calls, or lines: record `keep-simple`, include that evidence in the ledger, commit the change, update the baseline, and reset failures to zero.
   - Otherwise: record `discard` or `crash`, restore the attempted change, and increase this part's consecutive failure count.
5. **Loop immediately**:
   - After a kept change: find the new most expensive operation and try the Optimization Order again.
   - After fewer than three consecutive failures: try the next untried item from the Optimization Order.
   - After all applicable items have been tried against the current baseline, or after three consecutive failures: mark this part finished and select the next part.
   - After finishing a part: use the collected measurements to re-rank the remaining parts, then select the highest-cost part.
   - After selecting a part: run its benchmark several times, record its baseline median, and return to **Find**.
   - When no parts remain: proceed to Finish.

**The cycle is complete only when every listed part is marked finished in the ledger.**

## Finish

Run the tests once more and report:

| Part | What changed | Metric | Before | After | Difference |
| :--- | :--- | :--- | :--- | :--- | :--- |

Explain plainly what improved and which parts ran out of useful ideas. Ask whether to keep or remove the benchmark and `[PERF-PROBE]` code.

**Finish is complete when:** the final test result and every part appear in the report, the working tree contains only intentional retained files, and the user has been asked about measurement-code retention.

## Optimization Order

Try each applicable item once against the current baseline, skipping attempts already recorded for the selected part and baseline:

1. Remove unnecessary work.
2. Reuse repeated work.
3. Combine many small operations.
4. Delay work until needed.
5. Use simpler data.
