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

Run autonomously and preserve correct behavior. After every action, immediately take the next action defined below. The next user-facing response after setup begins is either a setup blocker or the final report. Baselines, retained improvements, failed experiments, and completed hotspots are intermediate states.

## 1. Setup

1. Create a dedicated branch from a clean base.
2. Check the required build, test, and measurement tools. If one is missing, ask the user to install it or approve installation.
3. Run the tests. If they fail before changes, record the failures and ask whether to repair them or treat them as the known test baseline.
4. Create a unique untracked `.perf/<run-id>/` directory:

```text
.perf/<run-id>/
├── map.md
├── parts.tsv
├── hypotheses.tsv
├── attempts.tsv
└── logs/
```

Use these headers:

```text
# parts.tsv
rank	part	workload_share	mechanism	control	baseline_id	baseline	status	reason

# hypotheses.tsv
id	parent	part	baseline_id	hypothesis	prediction	status	result	insight	commit

# attempts.tsv
attempt	part	baseline_id	hypothesis	approach	before	after	delta	result	evidence	commit
```

Part status is one of `queued`, `active`, `saturated`, or `excluded`. Hypothesis status is one of `pending`, `tested`, or `pruned`. Attempt result is one of `keep`, `keep-simple`, `discard`, or `crash`; `inapplicable` belongs only in `hypotheses.tsv` as a `pruned` hypothesis.

**Setup is complete when:** the test baseline is known and all three TSV files contain their headers.

## 2. Map the Workload

Choose one path:

- **Target specified:** Treat the named module, route, or flow as the scope boundary. Trace every stage through calls, data changes, and resource use. Follow costly calls into deeper layers until each cost belongs to a concrete function, query, loop, allocation, conversion, or I/O operation.
- **No target specified:** Map the whole codebase architecture. Find entry points from manifests and source files, identify major components, trace calls and data between them, and mark computation, memory, storage, and network boundaries.

Write the complete user or caller operation and its usable-completion boundary to `map.md`. Measure end-to-end cost, then decompose it into concrete hotspots. For each hotspot, record its measured share of the complete workload, limiting mechanism, and whether the repository controls that mechanism. CPU profile percentages are evidence about CPU time, not request wall time; keep denominators explicit.

A hotspot may be `excluded` only with evidence for one of these conditions:

- its measured maximum possible whole-workload benefit is below the run's recorded materiality threshold;
- the limiting mechanism is outside the repository's control;
- a hard environment constraint prevents representative measurement.

Record the evidence and constraint in `reason`. Rank all other hotspots by measured whole-workload cost, mark the highest `active`, and mark the rest `queued`. Estimated `frequency * cost` may prioritize initial probes but cannot admit, exclude, or complete a hotspot.

**Mapping is complete when:** every stage or major component is assigned to a concrete hotspot or measured exclusion, shares use an explicit complete-workload denominator, and every non-excluded part names a plausible limiting mechanism.

## 3. Bind Measurement

For the active hotspot:

1. Reuse an existing reliable benchmark where possible. Otherwise add the smallest benchmark or `[PERF-PROBE]` needed and commit that setup separately.
2. Record measurement files added by this run in `map.md`; existing measurement files are outside this list.
3. Bind the concrete symbol or query, editable files, correctness command, complete benchmark command, scalar objective and direction, representative inputs, and measured noise threshold in `map.md`.
4. Treat evaluator and correctness files as read-only during candidate experiments. Commit harness changes separately and establish a fresh baseline before resuming candidates.
5. Run the complete benchmark several times with warmup. Record raw samples, median metric, and median wall-clock duration.
6. Create a baseline ID and write its metric to `parts.tsv`.
7. Set the timeout to `max(2 × median wall time, median wall time + 5 seconds)`.

**Measurement is complete when:** the active hotspot has a repeatable end-to-end baseline, measured workload share, baseline ID, noise threshold, bound evaluator, and derived timeout.

## 4. Build the Hypothesis Frontier

Classify the limiting mechanism: unnecessary work, allocation or initialization, bandwidth, dependent memory access, branch recovery, arithmetic throughput or dependency, synchronization, queueing, or I/O. Add concrete hypotheses to `hypotheses.tsv` using the Optimization Order as search directions. Each hypothesis names one predicted metric effect and one distinguishable code change.

Static analysis, historical results, and pruned hypotheses guide this frontier but do not count as experiments. When a direction has no applicable mutation, prune that hypothesis with concrete evidence and generate another. When the frontier becomes empty before saturation, inspect failed and near-miss results, follow the next-largest cost deeper, and add refined or sibling hypotheses.

**The frontier is ready when:** it contains at least one pending, controllable code-change hypothesis tied to the active hotspot and current baseline.

## 5. Experiment

Run one pending hypothesis at a time:

1. **Predict:** Record the expected metric effect and likely failure mode.
2. **Change:** Make one small candidate diff within the bound editable files. Distinct approaches change different work, representation, ownership, batching, or execution mechanisms; parameter tweaks and cosmetic variants of one mechanism are one approach.
3. **Check:** Run the test baseline and complete benchmark. Save output under `logs/` and inspect the metrics or last 40 lines.
4. **Record and route:**
   - Tests match and improvement is at least 5%: record numeric before/after evidence as `keep`, commit, and continue at **Re-profile**.
   - Tests match, performance is within measured noise, and the diff observably removes a branch, allocation, query, call, conversion, or repeated operation from the measured path: record `keep-simple`, commit, and continue at **Re-profile**.
   - Tests fail, time out, or miss both keep gates: record `crash` or `discard`, restore the candidate diff, capture the result as an insight, and select or generate the next hypothesis.

Every candidate diff gets one numeric row in `attempts.tsv`; preserve timeout rows when changing the timeout. A pruned or `inapplicable` hypothesis gets no attempt row and does not consume experiment count, budget, or patience.

Against one current baseline, use a default budget of eight distinct measured approaches and patience of three consecutive measured non-wins. A material hotspot becomes `saturated` only after at least three distinct approaches have numeric measurements against that current baseline and either patience is exhausted or all eight budget slots are consumed. Continue generating hypotheses while this condition is false.

**Experimentation is complete when:** the candidate is retained and routed to Re-profile, or the active hotspot satisfies the mechanical saturation rule.

## 6. Re-profile and Advance

After every retained candidate:

1. Run the complete workload again and create a new baseline ID.
2. Re-profile it, update hotspot shares and mechanisms, and add newly exposed concrete hotspots to `parts.tsv`.
3. Re-rank queued hotspots using current measurements.
4. Return the retained hotspot to `active` under its new baseline and rebuild its hypothesis frontier. Its experiment count and patience restart at zero because earlier attempts targeted an old baseline.

After saturation, promote the highest-cost queued hotspot and return to Bind Measurement. When only `saturated` and `excluded` hotspots remain, proceed to Final Audit.

**Advance is complete when:** a measured hotspot is active or every hotspot has the final state `saturated` or `excluded`.

## 7. Final Audit

Check all of the following mechanically:

- `map.md` accounts for the selected target or whole codebase against a complete-workload measurement.
- No hotspot remains `queued` or `active`.
- Every `excluded` hotspot has measured insignificance, external control, or a hard measurement constraint in `reason`.
- Every `saturated` hotspot points to at least three numeric attempt rows against its final baseline using distinct approaches.
- Each saturated hotspot's final-baseline rows show either three consecutive non-wins or eight measured approaches.
- Pruned and `inapplicable` hypotheses contribute to neither attempt count nor saturation.
- Every retained gain points to a commit from this run and was followed by re-profiling.
- Every timeout appears in `attempts.tsv`.
- Tests match the test baseline and the working tree contains only intentional retained files.

Route a failed check to its owning step. Proceed only when every check passes.

## 8. Finish

Report separately:

1. **Changes made in this run**

   | Hotspot | What changed | Metric | Before | After | Difference | Commit |
   | :--- | :--- | :--- | :--- | :--- | :--- | :--- |

2. **Saturated hotspots** — list final-baseline experiments, numeric evidence, and resulting insights.
3. **Excluded hotspots** — list measured share and exclusion constraint.
4. **Existing optimizations observed** — label historical evidence as existing work rather than gains from this run.

Label claims as measured, derived, hypothesized, or unvalidated. Ask whether to retain or remove only benchmarks and `[PERF-PROBE]` code introduced by this run; leave existing measurement code untouched.

**Finish is complete when:** every hotspot appears in exactly one report section, every current-run gain names its commit, the final test result is reported, and the measurement-code question names only files introduced by this run.

## Optimization Order

Use these as directions for generating concrete hypotheses, not as a completion checklist:

1. Remove unnecessary work.
2. Reuse repeated work.
3. Combine many small operations.
4. Delay work until needed.
5. Use simpler data.
