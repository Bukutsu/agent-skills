# Comparable autonomous optimization skills

Research question: Which existing agent skills or first-party agent workflows are closest to `perf-loop`, and which mechanisms could prevent its premature “all ideas are inapplicable” exit?

## Sources reviewed

Primary repository sources, pinned to the revisions inspected:

1. [Karpathy, `autoresearch/program.md`](https://github.com/karpathy/autoresearch/blob/228791fb499afffb54b46200aca536f79142f117/program.md)
2. [K-Dense Scientific Agent Skills, Arbor](https://github.com/K-Dense-AI/scientific-agent-skills/blob/9cf7d9aea7d84754db4c167ab04b299d33c444bc/skills/arbor/SKILL.md)
3. [Agent Loop Skills, `optimize-loop`](https://github.com/gaasher/Agent-Loop-Skills/blob/f1169e6db0b0f8a83ced3a18562b7c57e14a748a/loops/optimize-loop/SKILL.md)
4. [Agent Loop Skills, Karpathy adaptation](https://github.com/gaasher/Agent-Loop-Skills/blob/f1169e6db0b0f8a83ced3a18562b7c57e14a748a/loops/karpathy/SKILL.md)
5. [Mnwa, Performance Engineering skill](https://github.com/Mnwa/performance-engineering/blob/1f8c23164f35c4c5868e765753904202ac4fe834/skills/performance-engineering/SKILL.md)

## Closest match

No single source matches `perf-loop` exactly. The strongest design is a combination of three:

- **Karpathy Autoresearch** supplies the uncompromising experiment loop: make one real change, run it, keep or revert, and immediately continue. Its loop does not permit static analysis to stand in for an experiment.
- **Arbor** supplies persistent search memory: hypotheses form a frontier, failed experiments become constraints, and near-misses create refined hypotheses instead of ending a direction.
- **Performance Engineering** supplies sound systems diagnosis: identify a limiting mechanism rather than merely a hot location, retain an end-to-end workload, estimate potential benefit, and distinguish measured, derived, hypothesized, and unknown claims.

`optimize-loop` adds a practical bounded stop: a hard experiment budget plus patience measured in consecutive non-improving **iterations**.

## Reusable mechanisms

### 1. Require a real experiment per iteration

Karpathy's loop always edits the artifact before running the evaluator. A failed or crashing experiment still produces useful evidence and is logged before rollback. “Run out of ideas” causes more ideation, not a successful conclusion.

**Use in `perf-loop`:** A material hotspot cannot become saturated from `inapplicable` rows. Saturation requires measured candidate diffs. Static analysis may reject a specific idea before mutation, but does not count as an experiment or a failure.

### 2. Bind the evaluator before searching

Karpathy fixes the editable file, evaluator, metric, and run budget before experimentation. `optimize-loop` similarly binds a correctness command, metric command, editable files, iteration budget, and patience.

**Use in `perf-loop`:** For each hotspot, bind:

- concrete symbol or query;
- editable files;
- end-to-end correctness command;
- benchmark command;
- scalar objective and direction;
- noise threshold;
- experiment budget and patience.

This prevents broad areas such as “background/startup” from being closed through prose.

### 3. Maintain a hypothesis frontier, not a checklist

Arbor stores pending, running, executed, and pruned hypotheses in a durable tree. Each result carries a factual outcome and a reusable insight. A half-right result becomes a sharper child hypothesis. A failed branch is pruned with a reason, while siblings remain available.

**Use in `perf-loop`:** Replace the five one-shot optimization-order rows with a hypothesis frontier. The Optimization Order generates initial directions, but each direction may contain multiple concrete hypotheses. After every result:

- keep and re-profile;
- refine a near-miss;
- prune only the tested hypothesis;
- select another pending hypothesis;
- generate new hypotheses when the frontier becomes empty before the experiment budget is met.

A TSV can represent this without subagents: `id`, `parent`, `hotspot`, `hypothesis`, `prediction`, `status`, `result`, `insight`, `commit`.

### 4. Diagnose a mechanism before changing code

The Performance Engineering skill says a hot function is a location, not a diagnosis. It requires classification of the limiting mechanism, competing hypotheses, a discriminating experiment, and an end-to-end check. It also recommends estimating the maximum application benefit from the hotspot's share.

**Use in `perf-loop`:** Admit only concrete hotspots with measured workload share. Before mutation, name the mechanism—for example unnecessary work, allocation, dependent memory access, branch recovery, synchronization, or I/O wait—and estimate the largest possible whole-workload gain. Exclude a hotspot only when its measured maximum benefit is below the run's materiality threshold or outside the repository's control.

### 5. Stop on measured patience or budget

`optimize-loop` increments patience only after an actual iteration fails to establish a new best. It also uses a hard iteration budget. This is more checkable than “all applicable ideas exhausted.”

**Use in `perf-loop`:** Give each hotspot:

- a minimum experiment count, such as 3;
- a default maximum budget, such as 8;
- patience, such as 3 consecutive measured non-wins.

A hotspot saturates when minimum experiments have run and patience is exhausted, or when the hard budget is consumed. `inapplicable` judgments affect neither counter.

### 6. Keep evaluator and artifact separate

Karpathy and `optimize-loop` make the evaluator read-only. This prevents optimizing the benchmark rather than the application.

**Use in `perf-loop`:** Record benchmark ownership. Existing tests and evaluator logic are read-only during candidate experiments. Measurement setup changes belong to a separate harness commit and require a fresh baseline before application changes resume.

### 7. Report provenance and uncertainty

The Performance Engineering skill requires every result to be labeled measured, derived, hypothesized, or unvalidated. Arbor separately reports explored candidates and candidates admitted by the final gate.

**Use in `perf-loop`:** Keep current-run changes, current-run rejected experiments, historical evidence, and unmeasured exclusions in separate report sections. Historical speedups cannot satisfy current-run experiment counts.

## What not to adopt

- **Infinite execution:** Karpathy's “loop forever” is useful for unattended ML search with a fixed evaluator, but unsafe and impractical for arbitrary repositories. Use a measurable budget and patience instead.
- **Mandatory subagents:** Arbor's isolated executor model is valuable for parallel experiments, but `perf-loop` must work sequentially in-session when subagent tools are unavailable.
- **One generic metric:** `optimize-loop` minimizes code complexity or SQL latency. Repo-wide performance needs different metrics per concrete hotspot plus an end-to-end application check.
- **Forced meaningless mutations:** Requiring three edits to a negligible or externally dominated area wastes time. First require measured hotspot share and a plausible controllable mechanism; only admitted hotspots receive the experiment minimum.

## Recommended design for `perf-loop`

Use a two-level loop:

1. **Architecture loop**
   - map the workload;
   - measure end-to-end cost;
   - identify concrete hotspots and their shares;
   - exclude only insignificant or uncontrollable hotspots with measured evidence;
   - rank admitted hotspots.

2. **Hypothesis loop per hotspot**
   - bind evaluator, metric, editable files, noise, budget, and patience;
   - maintain a persistent hypothesis frontier;
   - make one real diff per experiment;
   - test, measure, keep or revert;
   - convert results into insights and refined hypotheses;
   - re-profile after every keep;
   - saturate only after the minimum measured experiments and patience/budget condition.

The key completion rule should be:

> A material, controllable hotspot is saturated only after at least three distinct candidate changes have been measured against its current baseline and three consecutive measured candidates fail to establish a new best, or its experiment budget is exhausted. Static analysis and historical results guide the hypothesis frontier but do not count toward saturation.

## Expected effect on the reviewed failure

The session that wrote 30 `inapplicable` rows would no longer pass final audit:

- its six broad areas would need decomposition into concrete measured hotspots;
- only low-share or external costs could be excluded without mutation;
- `inapplicable` rows would not consume experiment budget or patience;
- each admitted hotspot would require real candidate diffs and measurements;
- prior repository wins would guide new hypotheses but could not prove current saturation.
