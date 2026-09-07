---
name: perf-loop
description: >-
  Autonomous performance and resource optimization loop. Use when asked to
  "optimize", "speed up", "profile", "reduce memory", or tune bottlenecks across
  a target or the whole project.
argument-hint: "[target area / module / flow] (optional: omit to scan entire project)"
---

# Perf Loop

Autonomous performance, efficiency, and resource optimization loop. Operates on any stack without stack-specific assumptions. Optimizes empirical bottlenecks, guarantees correctness, lands verified wins, and loops until stopped.

## Workflow

### 1. Scope & Branch Setup
- **Branch isolation**: Cut a dedicated working branch from clean base (e.g. `git checkout -b perf/<target-or-date>`).
- **Target selection**:
  - **Targeted mode** (argument provided): restrict all profiling, harnesses, and edits strictly to the specified module, route, or flow.
  - **Whole-project mode** (argument omitted): survey data flows, hot loops, serialization paths, and I/O boundaries. Rank candidates by leverage: `frequency of invocation * resource cost`. Select candidate #1 as active target; preserve the backlog.
- **Initialize ledger**: Create untracked `.perf-ledger.tsv` with header:
  `run_id\tcommit\tmetric_before\tmetric_after\tdelta\tstatus\tdescription`
- **Completion criterion**: Dedicated git branch active, active target selected, and `.perf-ledger.tsv` initialized.

### 2. Baseline & Harness
- Run the test suite. **All tests must pass before proceeding.**
- **Inject minimal probes**: If existing tools lack resolution, add low-overhead probes (monotonic timers, memory deltas, counters) directly into the code or create a dedicated benchmark script.
  - Keep probes zero-cost: raw clock/counter diffs only; avoid I/O or formatting on the measured path. Tag all in-tree probes with `[PERF-PROBE]`.
  - **Commit probes before mutating**: Commit the harness and probes first (`perf(harness): add minimal profiling probes`) so subsequent rollbacks keep the measurement infrastructure intact.
- **Protect context window**: Never print raw benchmark runs to stdout. Redirect to `.perf-run.log`:
  `<benchmark-command> > .perf-run.log 2>&1`
  Extract target metrics via `grep`. On failures, inspect only `tail -n 40 .perf-run.log`.
- Run multiple iterations with warmup. Record baseline median and budget.
- **Completion criterion**: A runnable command producing deterministic metrics against passing tests, recorded as run `0` (`baseline`) in `.perf-ledger.tsv`.

### 3. Locate Critical Path
- Profile the active workload using platform-native tools to identify where time, memory, or I/O is spent.
- Isolate the primary bottleneck (the single site accounting for majority consumption).
- **Completion criterion**: A named function, query, loop, or allocation site with its measured budget share.

### 4. Hypothesize & Mutate
- Consult `.perf-ledger.tsv` to avoid repeating discarded attempts or dead-ends.
- Formulate one falsifiable hypothesis before editing:
  - Target: `<file:line or symbol>`
  - Action: `<specific change, following the Optimization Hierarchy>`
  - Prediction: `<expected metric delta and causal mechanism>`
- Apply surgical edits: touch only what tests the hypothesis.
- **Completion criterion**: A stated hypothesis and a single atomic uncommitted diff.

### 5. Verify & Measure
- **Gate 1 (Correctness)**: Run the test suite. If any test fails, log `crash` in `.perf-ledger.tsv`, rollback immediately (`git restore .`), and proceed to next hypothesis.
- **Gate 2 (Benchmark with Timeout)**: Run benchmark redirected to `.perf-run.log`. Enforce hard timeout at $2\times$ baseline runtime. If run hangs: terminate, log `crash (timeout)` in `.perf-ledger.tsv`, and rollback (`git restore .`).
- Compare against baseline median:
  - **Hurdle met** (`delta >= 5%` win): Mark `keep`.
  - **Simplification win** (neutral delta `-1% <= delta < 5%` with measurably reduced complexity/lines): Mark `keep-simple`.
  - **Hurdle not met** (`delta < 5%` without simplification): Log `discard` in `.perf-ledger.tsv`, rollback (`git restore .`).

### 6. Commit & Advance Baseline
- For `keep` and `keep-simple`: commit the atomic win naming the change and delta (e.g. `perf(parser): eliminate redundant AST clones (-18% latency)`).
- Update active baseline median with the new post-optimization metric.
- Log the run in `.perf-ledger.tsv`.

### 7. Plateau Pivot & Loop
- **Plateau pivot (3-strike rule)**: If 3 consecutive attempts on the active target result in `discard` or `crash`, declare target saturated.
  - In **targeted mode**: proceed to Step 8.
  - In **whole-project mode**: promote the next candidate from the backlog and return to Step 2. If backlog is exhausted, proceed to Step 8.
- **Autonomous continuation**: Run continuously without pausing to ask permission until interrupted by user or all targets are exhausted.

### 8. Final Summary & Cleanup
When the loop exits:
1. **Clean probes**: Remove temporary `[PERF-PROBE]` lines from production code (preserve standalone benchmark harnesses). Verify tests pass.
2. **Before / After Table**:
   | Subsystem / Target | Bottleneck & Fix | Metric | Before | After | Net Delta |
   | :--- | :--- | :--- | :--- | :--- | :--- |
3. **Bro Skill Explanation**:
   - Restate the outcome in plain human language with zero jargon.
   - Explain what was slow, what was changed, and what that means in actual practice.

---

## Reference: Optimization Hierarchy

When addressing an identified hotspot, evaluate interventions in order of leverage:
1. **Eliminate work**: remove redundant computations, dead iterations, duplicate queries, and unnecessary allocations.
2. **Reuse work**: cache, memoize, or index expensive repeated calculations and lookups.
3. **Batch work**: combine frequent small operations, allocations, or I/O into bulk operations.
4. **Defer work**: lazy-load, evaluate on demand, or move non-blocking work out of the critical path.
5. **Simplify representation**: use simpler, contiguous, or more compact data structures. Algorithms follow data.
