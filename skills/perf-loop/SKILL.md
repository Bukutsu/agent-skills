---
name: perf-loop
description: >-
  Optimize performance, latency, throughput, memory, or resource efficiency
  across a target subsystem or the whole project in an autonomous loop. Use
  when asked to "optimize", "speed up", "profile", "reduce memory", or tune bottlenecks.
argument-hint: "[target area / module / flow] (optional: omit to scan entire project)"
---

# Perf Loop

Autonomous performance, efficiency, and resource optimization loop. Operates on any stack without stack-specific assumptions. Optimizes measured bottlenecks, guarantees correctness, lands verified wins, and loops until the user stops it.

## Optimization Hierarchy

When addressing an identified hotspot, evaluate interventions in order of leverage:
1. **Eliminate work**: remove redundant computations, dead iterations, duplicate queries, and unnecessary allocations.
2. **Reuse work**: cache, memoize, or index expensive repeated calculations and lookups.
3. **Batch work**: combine frequent small operations, allocations, or I/O into bulk operations.
4. **Defer work**: lazy-load, evaluate on demand, or move non-blocking work out of the critical path.
5. **Simplify representation**: use simpler, contiguous, or more compact data structures. Algorithms follow data.

## Persistent Ledger (`.perf-ledger.tsv`)

Keep an untracked TSV file `.perf-ledger.tsv` in the repository root throughout the run. Because it is untracked by git, **it survives `git restore` and `git reset`**, preserving persistent experiment memory across rollbacks.

Format (tab-separated):
```
run_id	commit	metric_before	metric_after	delta	status	description
```
Statuses:
- `keep`: Verified improvement meeting the hurdle rate.
- `keep-simple`: Neutral performance delta (`-1% <= delta < 5%`) that simplifies code or eliminates dead abstractions.
- `discard`: Did not clear the hurdle rate or regressed performance.
- `crash`: Execution failed, timed out, or broke test invariants.

Before forming hypotheses, inspect `.perf-ledger.tsv` to prevent repeating previously discarded ideas or dead-ends.

## Workflow

### 1. Scope & Branch Setup
- **Branch isolation**: Cut a dedicated working branch from the clean base (e.g. `git checkout -b perf/<target-or-date>`).
- **Target selection**:
  - **Targeted mode** (argument provided): restrict all profiling, harnesses, and edits strictly to the specified module, route, or flow.
  - **Whole-project mode** (argument omitted): survey data flows, hot loops, serialization paths, and I/O boundaries. Rank candidates by leverage: `frequency of invocation * resource cost`. Select candidate #1 as the active target; preserve the ranked backlog.
- Initialize `.perf-ledger.tsv` with the header row if not present.

### 2. Baseline & Harness
- Run the project test suite. **All tests must pass before proceeding.**
- **Instrumentation permission**: If existing tools or benchmarks lack fine-grained resolution, the agent is explicitly authorized to inject minimal, low-overhead profiling probes (e.g., monotonic timers, memory delta checkpoints, counters) directly into the code or create a dedicated benchmark script.
  - **Probe rules**: Keep probes zero-cost or ultra-light (e.g., monotonic clock diffs); avoid I/O, string formatting, or heavy allocations inside the measured path. Tag all in-tree probes with `[PERF-PROBE]`.
  - **Commit probes before mutating**: Commit the harness and probes to the branch first (`perf(harness): add minimal profiling probes`) so subsequent `git restore .` calls roll back only optimization mutations without wiping the measurement infrastructure.
- **Protect context window**: Never let raw benchmark output flood stdout. Redirect output to `.perf-run.log`:
  ```bash
  <benchmark-command> > .perf-run.log 2>&1
  ```
  Extract target metrics via `grep`. On failures, inspect only `tail -n 40 .perf-run.log`.
- Run multiple iterations with warmup. Record the baseline median and execution time budget.
- **Completion criterion**: A runnable command producing deterministic metrics against a green test suite, logged as run `0` (`baseline`) in `.perf-ledger.tsv`.

### 3. Locate the Critical Path
- Profile the active workload using platform-native tools to identify where time, memory, or I/O is spent.
- Isolate the primary bottleneck (the single site accounting for the majority of resource consumption).
- **Completion criterion**: A named function, query, loop, or allocation site with its measured budget share.

### 4. Hypothesize & Mutate
- Check `.perf-ledger.tsv` to ensure the planned idea has not already been attempted.
- Formulate one falsifiable hypothesis before editing:
  - Target: `<file:line or symbol>`
  - Action: `<specific change, following the Optimization Hierarchy>`
  - Prediction: `<expected metric delta and causal explanation>`
- Apply surgical edits: change only what is required to test the hypothesis.

### 5. Verify & Measure
- **Gate 1 (Correctness)**: Run the test suite. If any test fails, log `crash` in `.perf-ledger.tsv`, rollback immediately (`git restore .`), and proceed to the next hypothesis.
- **Gate 2 (Benchmark with Timeout)**: Run the benchmark harness redirecting to `.perf-run.log`. Enforce a hard timeout at $2\times$ baseline runtime. If the run hangs or exceeds timeout: terminate, log `crash (timeout)` in `.perf-ledger.tsv`, and rollback (`git restore .`).
- Compare against baseline median:
  - **Hurdle met** (`delta >= 5%` win): Mark `keep`.
  - **Simplification win** (neutral delta `-1% <= delta < 5%` with measurably reduced lines/complexity): Mark `keep-simple`.
  - **Hurdle not met** (`delta < 5%` without simplification): Log `discard` in `.perf-ledger.tsv`, rollback (`git restore .`).

### 6. Commit & Advance Baseline
- For `keep` and `keep-simple` runs: commit the atomic change naming the win (e.g. `perf(parser): eliminate redundant AST clones (-18% latency)`).
- Update the active baseline median with the new post-optimization metric.
- Log the completed run in `.perf-ledger.tsv`.

### 7. Plateau Pivot & Loop
- **Plateau pivot (3-strike rule)**: If 3 consecutive attempts on the active target result in `discard` or `crash`, declare the target saturated.
  - In **targeted mode**: proceed to Step 8.
  - In **whole-project mode**: promote the next candidate from the backlog and return to Step 2. If all backlog candidates are exhausted, proceed to Step 8.
- **Autonomous continuation**: Once running, do not pause to ask if you should continue. Run autonomously until interrupted by the user or all targets are exhausted.

### 8. Final Summary & Cleanup
When the loop exits:

1. **Clean temporary probes**: Remove all temporary `[PERF-PROBE]` lines from production code (preserving permanent benchmark harnesses if desirable). Run the test suite to verify clean production code.
2. **Before / After Table**:
   | Subsystem / Target | Bottleneck & Fix | Metric | Before | After | Net Delta |
   | :--- | :--- | :--- | :--- | :--- | :--- |

3. **Bro Skill Explanation**:
   - Restate the outcome in plain human language with zero jargon.
   - Speak simply and concisely, like one human talking to another: explain what was slow, what was changed, and what that means in actual practice.
