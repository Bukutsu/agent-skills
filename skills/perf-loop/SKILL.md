---
name: perf-loop
description: >-
  Autonomous performance, efficiency, and resource optimization loop. Accepts an
  optional target subsystem or audits the whole project to rank hotspots.
  Establishes an empirical baseline, profiles the critical path, tests
  falsifiable hypotheses, preserves correctness, commits verified wins, and
  repeats until stopped.
argument-hint: "[target area / module / flow] (optional: omit to scan entire project)"
---

# Perf Loop

Autonomous performance, efficiency, and resource optimization loop. Operates on any tech stack without stack-specific assumptions. Optimizes empirical bottlenecks, guarantees behavioral correctness, commits measurable gains, and loops until the user stops it.

## Core Rules

1. **Correctness is the Hard Gate**: An optimization that breaks a test, contract, or invariant is not an optimization—it is broken code. The test suite must pass before profiling and immediately after every mutation. Zero semantic regression.
2. **Measurement Before Mutation**: Never touch code based on static assumptions or intuition. Only optimize what is measured on the critical path (Amdahl's Law).
3. **Falsifiable Hypotheses**: Every change must state a predicted metric delta and the mechanism causing it before editing.
4. **Statistical Rigor**: Run benchmarks with warmups. Use medians over $N$ runs (minimum 3–5 runs) to reject environment noise. Clear a minimum hurdle rate (e.g., $\ge 5\%$).
5. **Atomic Commit & Instant Rollback**: One change per cycle. If tests fail or the benchmark delta is below the hurdle rate, rollback immediately via git. If the win is verified, commit it cleanly.
6. **Complexity Budget**: Readability and simplicity are resources. Never trade clarity for marginal gains. Prefer clean data structures and eliminating redundant work over convoluted caching or premature concurrency.

## Optimization Hierarchy (Order of Leverage)

When addressing an identified bottleneck, evaluate interventions in this strict order:
1. **Eliminate work**: Remove redundant computations, dead iterations, duplicate queries, unnecessary allocations.
2. **Reuse work**: Cache, memoize, or index expensive repeated calculations or lookups.
3. **Batch work**: Combine frequent small I/O operations, allocations, or round-trips into bulk operations.
4. **Defer work**: Lazy-load, evaluate on-demand, or push non-blocking work out of the critical path.
5. **Simplify representation**: Use tighter, simpler data structures. Algorithms follow data.

## Workflow

### 1. Scope & Target Selection
- **User-specified target**: If an argument or focus area was provided (e.g., a specific module, endpoint, pipeline, or file), restrict the optimization scope strictly to that subsystem.
- **Whole-project audit (default when unspecified)**:
  - Survey the repository's main data flows, heavy loops, I/O boundaries, serialization paths, and existing benchmarks/test fixtures.
  - Rank candidates by estimated leverage: $\text{frequency of invocation} \times \text{resource cost}$.
  - Select the #1 highest-leverage candidate as the active target for the first round. Maintain the ranked backlog for subsequent rounds.

### 2. Baseline & Harness
- Identify the project's existing test suite and verify that all tests pass cleanly. A green test suite is required before touching any code.
- Locate an existing benchmark or construct a minimal, deterministic benchmark harness exercising the active target.
- Run the benchmark across multiple iterations with warmup to establish a stable baseline (measure latency, throughput, memory, or allocations). Record the median and variance.

### 3. Locate the Critical Path
- Instrument or profile the target workload using the platform's native or standard tools to identify where resources are actually spent.
- Target only the single largest bottleneck (>50–80% of execution time, memory allocations, or I/O). Ignore non-critical paths.

### 4. Hypothesize & Mutate
- State a single, falsifiable hypothesis:
  - Bottleneck identified: `<file:function or data flow>`
  - Planned intervention: `<exact change, following the Optimization Hierarchy>`
  - Expected effect: `<predicted metric delta and why>`
- Apply the surgical change. Touch only what is required for this hypothesis.

### 5. Verify & Measure
- **Gate 1 (Correctness)**: Run the full test suite. If any test fails or behavior changes, rollback immediately (`git restore .`) and record the failure.
- **Gate 2 (Benchmark)**: Run the benchmark harness using the exact same parameters and runs as the baseline.
- Calculate the delta against baseline median:
  - If improvement is below the hurdle rate ($\Delta < 5\%$ or within noise): rollback immediately (`git restore .`).
  - If improvement meets or exceeds the hurdle rate ($\Delta \ge 5\%$): keep the change.

### 6. Commit & Log
- Commit the win as an ordinary commit with the measured improvement in the commit message (e.g., `perf(core): eliminate redundant allocations in parser (-14% latency)`).
- Update the baseline with the new measurement.

### 7. Repeat Until Stopped
- Report a concise cycle summary:
  - Round number & active target
  - Bottleneck addressed
  - Measured delta (before vs after)
  - Status (Committed / Rolled back)
- **Next cycle selection**:
  - If working on a **user-specified target**: continue profiling that target for the next critical path until diminishing returns (<5% potential remaining), then report completion.
  - If working on **whole-project mode**: once the active target yields diminishing returns, promote the next highest-leverage target from the audit backlog and repeat from **Step 2**.
- Continue cycling until the user explicitly stops the agent, or all identified areas are exhausted.
