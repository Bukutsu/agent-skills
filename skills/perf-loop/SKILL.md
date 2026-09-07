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

## Workflow

### 1. Scope & Target Selection
- **Targeted mode** (argument provided): restrict all measurement and edits strictly to the specified module, route, or flow.
- **Whole-project mode** (argument omitted):
  - Survey repository data flows, hot loops, I/O boundaries, and serialization paths.
  - Rank candidates by leverage: `frequency of invocation * resource cost`.
  - Select candidate #1 as the active target; preserve the ranked backlog for subsequent rounds.

### 2. Baseline & Harness
- Run the project test suite. **All tests must pass before proceeding.**
- Identify or construct a deterministic, automated benchmark harness exercising the active target.
- Run the benchmark across multiple iterations with warmup. Record the baseline median and variance.
- **Completion criterion**: A single runnable command producing deterministic timing/resource numbers against a green test suite.

### 3. Locate the Critical Path
- Instrument or profile the target workload using platform-native tools to measure where time, memory, or I/O is spent.
- Isolate the primary bottleneck (the single site accounting for the majority of resource consumption). Focus exclusively on this critical path.
- **Completion criterion**: A named function, query, loop, or allocation site with its measured budget share.

### 4. Hypothesize & Mutate
- Formulate one falsifiable hypothesis before editing:
  - Target: `<file:line or symbol>`
  - Action: `<specific change, following the Optimization Hierarchy>`
  - Prediction: `<expected metric delta and causal explanation>`
- Apply surgical edits: change only what is required to test the hypothesis. Preserve code readability; reject changes that add disproportionate complexity.

### 5. Verify & Measure
- **Gate 1 (Correctness)**: Run the full test suite. If any test fails or observable behavior changes, rollback immediately (`git restore .`) and record the failure.
- **Gate 2 (Benchmark)**: Run the benchmark harness using the baseline configuration and warmup.
- Compare against baseline median:
  - **Hurdle not met** (`delta < 5%` or within noise): rollback immediately (`git restore .`).
  - **Hurdle met** (`delta >= 5%` win): keep the change.

### 6. Commit & Update Baseline
- Commit the win as an atomic commit naming the change and the verified delta (e.g. `perf(parser): eliminate redundant AST clones (-18% latency)`).
- Update the baseline measurement with the new post-optimization median.

### 7. Repeat Until Stopped
- Report cycle status: round number, active target, bottleneck addressed, measured delta, and outcome (committed or rolled back).
- **Next cycle selection**:
  - In **targeted mode**: continue profiling the active target for the next bottleneck until diminishing returns (`< 5%` remaining potential), then proceed to Step 8.
  - In **whole-project mode**: when the active target yields diminishing returns, promote the next candidate from the backlog and return to Step 2. If all candidates are exhausted, proceed to Step 8.
- Continue cycling until the user interrupts or no bottlenecks remain above the hurdle rate.

### 8. Final Summary (Before/After Table & Bro Rules)
When the loop exits, deliver the final summary:

1. **Before / After Table**:
   | Subsystem / Target | Bottleneck & Fix | Metric | Before | After | Net Delta |
   | :--- | :--- | :--- | :--- | :--- | :--- |

2. **Bro Skill Explanation**:
   - Restate the outcome in plain human language with zero jargon.
   - Speak simply and concisely, like one human talking to another: explain what was slow, what was changed, and what that means in actual practice.
