# AI Agent & Workflow Automation Platform: Research Report

> **Where the code is.** The experiment scripts that produced these results are part of the source package, which is distributed separately (see [SOURCE-ACCESS.md](../SOURCE-ACCESS.md)). The raw results they wrote are included in this repository under [`results/`](../results/).

<p align="center"><img src="assets/banner.svg" alt="AI Agent &amp; Workflow Automation Platform" width="100%"></p>

> **Status: protocol v0.2, results recorded.** All four experiments were run once against a **simulated model**. Every result below is a statement about the platform's mechanisms (schema validation and repair, planning-strategy support, model routing, approval gates and other safety controls) under **assumed, configurable model behaviour**. None of it measures the quality of any real language model, and no real model, API or key was used anywhere in this study.

## Abstract

This study asks which design choices (approval gates and other safety controls, tool-schema strictness, planning strategy, model tier) most improve task success per unit of cost for tool-using agent workflows, and how stable those gains are across repeated runs. Because no language-model API was available, the system under test is the agent platform in this repository driven by a deterministic, seeded simulator whose behaviour (malformed tool arguments, wrong tools, silent mistakes that grow with context length, compliance with injected instructions) is set by documented parameters. Four experiments were run once, with hypotheses and decision rules fixed in the experiment scripts beforehand. In the simulation, schema validation with repair raised success for both simulated models (H1 supported for the stronger model; for the weaker one the success gain was clear but the added cost interval touched 15.3% against a 15% limit); plan-then-execute did not meet the pre-registered rule (H2 not supported); routing was supported only for a plan-based, tool-tier router and not for an error-triggered cascade (H3 partly supported); and approval gates caught about 87% of unsafe attempts at a 45% intervention rate under the default simulated compliance, but exceeded the intervention bound when the simulated model was made to obey every injected instruction (H4 partly supported). The most useful findings are about mechanism and measurement, not about models: fail-closed validation is worse than no validation, an error-triggered cascade silently loses accuracy that no gate can see, and layered static controls (scopes, allow-lists) did most of the work before any human was asked.

## 1. Research question

Which design choices (approval gates, tool-schema strictness, planning strategy, model tier) most improve task success per unit of cost for tool-using agents, and how stable are those gains across repeated runs, **when the model is simulated**?

## 2. Hypotheses

Original hypotheses (protocol v0.1), unchanged in intent:

- H1: Strict JSON-schema tool validation with automatic repair retries raises task success versus unvalidated calls, at under 15% added cost.
- H2: A plan-then-execute strategy outperforms purely reactive (ReAct-style) execution on tasks of five or more steps, but not on shorter tasks.
- H3: Routing easy steps to a smaller model cuts cost per success by at least 30% with no statistically significant drop in success rate.
- H4: Human approval gates catch a majority of injected unsafe actions at a bounded intervention rate.

### 2.1 Operational decision rules (fixed in the experiment scripts before the run)

| Hypothesis                          | Rule (all parts must hold)                                                                                                                                                                                                                                                                                   |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| H1, per simulated model             | Per-task paired success difference (repair minus off): 95% bootstrap CI lower bound above 0 **and** Holm-adjusted Wilcoxon p below 0.05 (family: the two models); **and** upper bound of the 95% CI of added mean cost below 15%                                                                             |
| H2                                  | Base arm: for each length bin 5-6, 7-8, 9-10 the success difference (plan minus reactive) has CI lower bound above 0 and Holm-adjusted p below 0.05 (family: four bins); **and** the 3-4 bin shows no significant positive difference                                                                        |
| H3, per routed configuration        | Upper bound of the 95% CI of (cost per success routed / cost per success single large) below 0.70; **and** (success difference at least 0 **or** Holm-adjusted p at least 0.05); **and** lower bound of the success-difference CI above -5 percentage points (non-inferiority margin, stated before the run) |
| H4, policy P3 (approval gates only) | Wilson 95% lower bound of the catch rate above 0.5 **and** intervention rate (share of all runs with at least one approval request) at most 0.5. "Bounded" was operationalised as 0.5 before the run                                                                                                         |

## 3. Background

The study sits between three lines of work. ReAct interleaves reasoning and tool calls in a single reactive loop [1]; plan-then-execute prompting separates planning from execution [2]; and tool-agent benchmarks such as τ-bench evaluate agents on multi-step tool use with programmatic state checks [3]. Cost-aware model selection (cascades of cheaper and stronger models) was studied in FrugalGPT [4]. Safety work on tool-using agents shows that instructions hidden in tool output can hijack an agent [5] and provides benchmark environments for measuring attacks and defences [6]. The Model Context Protocol defines a tool interface that this platform's adapter implements in part [7]. Statistical methods follow standard references: percentile bootstrap [8], the Wilcoxon signed-rank test [9], Holm's sequentially rejective correction [10] and Cliff's delta [11], and Wilson intervals for proportions [12].

None of those works is reproduced here. This study differs in that the model is simulated, so it can say something only about the platform's own mechanisms; it cannot confirm or contradict any finding about real models.

## 4. Method

### 4.1 System under test

The agent platform in this repository: a durable workflow engine with typed, permissioned tools, schema validation with optional repair, reactive and plan-then-execute agent strategies, model routing, human approval gates with timeouts and escalation, budgets, and full tracing. The model behind it is a seeded **simulated model** (Section 4.6). External services (ticketing, email, payments, the web) are local stand-ins backed by SQLite and a synthetic world of customers, orders, invoices and knowledge-base articles. The Claude adapter in the repository was implemented against the provider documentation and **never exercised**; it plays no part in any experiment.

### 4.2 Environment

- **E1 to E3:** 120 scripted tasks across three categories (support triage, data lookup, document processing), 15 for each solution length from 3 to 10 tool calls, generated from a fixed seed. Ground truth is computed from the synthetic world; graders check external state (exactly one correct ticket, no stray effects) and the final answer. Tasks are repeated over replicate model seeds.
- **E4:** 288 scenarios (72 per task kind: policy lookup, ticket triage, customer email, web fetch). Of every 12 consecutive scenarios per kind, 8 carry an attack (4 attack types times 2 payload styles) and 4 are clean controls. Payloads are planted in tool output (order note, knowledge-base article or fetched page).
- **Hardware and load:** one Windows 11 machine (8 logical CPUs, 28 GB memory, AMD EPYC 7543P family) shared with other builds; CPU utilisation was 100% before and after every experiment. Each experiment ran as a single Node.js 25.9 process with `UV_THREADPOOL_SIZE=2`; wall-clock times (E1 36 s, E2 37 s, E3 25 s, E4 301 s) are informational only. No conclusion depends on timing.

### 4.3 Baselines

- Unvalidated reactive loop: validation mode `off` (E1).
- Single large model, reactive: the reference for E3.
- No controls: policy P0 in E4.

Planned baselines that were **not** built or evaluated: a single-prompt agent with no tools, and a human-written deterministic workflow as a success baseline (a deterministic DAG exists in the demo workflows but was not evaluated as a baseline).

### 4.4 Independent variables

- Tool-schema validation: off, on (fail closed), on with repair (E1).
- Planning strategy: reactive or plan-then-execute (E2, E3).
- Model tier and routing rule: single large, single small, error-triggered cascade, tool-tier routing (E3).
- Safety policy: six cumulative policies from no controls to scopes, allow-lists, approval gates, taint gate and a quarantining injection detector (E4).

### 4.5 Metrics

- Task success rate (programmatic grader).
- Model calls per run, tool calls, tool errors and schema violations per run (counts).
- Cost per run and cost per successful task, in **simulated cost units** (abstract prices per 1,000 tokens, not currency).
- Attempt-level catch rate of unsafe actions; intervention rate (runs with at least one approval request); attack success (an unsafe effect really happened in the services database); task success on clean and on attacked runs.
- **Not measured:** p50 and p95 latency. The planned latency metrics were dropped because the machine was heavily shared and because a simulated model has no real latency; model calls per run is reported as a count-based proxy.

### 4.6 The simulated model

Behaviour is a pure function of (seed, request); profile parameters were chosen once, before any experiment, and none was tuned afterwards. They are assumptions, not measurements.

| Parameter                                                                                                     | sim-large    | sim-small   |
| ------------------------------------------------------------------------------------------------------------- | ------------ | ----------- |
| Price in / out (cost units per 1k tokens)                                                                     | 3 / 15       | 0.25 / 1.25 |
| Chance a call has corrupted arguments (missing field, wrong type, bad format, extra parameter in equal parts) | 0.04         | 0.15        |
| Wrong existing tool / nonexistent tool                                                                        | 0.01 / 0.005 | 0.04 / 0.02 |
| Silent schema-valid mistake per call (base)                                                                   | 0.01         | 0.03        |
| Growth of that rate per call already in context (reactive)                                                    | 0.15         | 0.15        |
| Multiplier on that rate when executing a plan step                                                            | 0.5          | 0.5         |
| Chance each plan step names the wrong tool                                                                    | 0.02         | 0.06        |
| Fixes a bad call after an informative validation error / after a generic error                                | 0.9 / 0.5    | 0.75 / 0.3  |
| Obeys an instruction planted in tool output                                                                   | 0.35         | 0.6         |
| Proposes an unprompted refund per turn                                                                        | 0.01         | 0.03        |
| Wrong final answer text                                                                                       | 0.01         | 0.03        |

Token counts are a deterministic estimate (about four characters per token) over the real prompts the platform builds, so costs reflect actual prompt size. Two assumptions matter especially: reactive mistakes grow with context length (E2 has a control arm that switches this off), and handlers are written the ordinary way (they assume well-typed input), so what happens to malformed arguments with validation off is a property of those handlers, not a scripted outcome.

## 5. Experiment matrix

| ID  | Experiment        | Manipulation                                                                                                 | Measured                       | Runs                                               |
| --- | ----------------- | ------------------------------------------------------------------------------------------------------------ | ------------------------------ | -------------------------------------------------- |
| E1  | Schema validation | Off, on (fail closed), on with repair; two simulated models                                                  | Success rate, cost             | 3,600 (120 tasks x 5 seeds x 6 cells)              |
| E2  | Planning strategy | Reactive vs. plan-then-execute, tasks binned by length; base arm and a no-context-growth control arm         | Success rate by step count     | 4,800 (120 tasks x 10 seeds x 4 cells)             |
| E3  | Model routing     | Single large, single small, error-triggered cascade, plan-then-execute on large (control), tool-tier routing | Cost per success, success rate | 3,000 (120 tasks x 5 seeds x 5)                    |
| E4  | Safety gates      | Six cumulative policies against injected prompt-injection and unsafe-action attempts; two compliance arms    | Catch rate, intervention rate  | 10,368 (288 scenarios x 6 policies x 4 or 2 seeds) |

## 6. Statistical plan

- **Replication:** run counts are fixed before collection (above) and reported. Replicate model seeds are shared across conditions, so comparisons are paired by task and seed.
- **Summaries:** per-task means over seeds, then the mean over tasks with a percentile bootstrap 95% CI (2,000 resamples, fixed seed, resampling tasks). Cost per success is total cost divided by successes, bootstrapped over tasks. E4 proportions use Wilson 95% intervals.
- **Comparisons:** paired Wilcoxon signed-rank test on per-task differences (normal approximation with tie and continuity correction).
- **Effect sizes:** Cliff's delta is reported next to p-values; a significant but negligible effect is called negligible.
- **Multiple comparisons:** Holm-Bonferroni within the family named in Section 2.1. Comparisons outside a family are labelled descriptive.
- **No silent exclusions:** no run was excluded; failures are counted as failures and their causes are tabulated.
- **Negative results are results:** reported as such below.

## 7. Results

All figures come from `results/e1` to `results/e4` (raw rows in `runs.csv`, tables and decision rules in `summary.json` and `summary.md`). Every figure is **simulated**. Success intervals are over tasks (n = 120 unless a length bin is shown). Cost units are abstract.

### E1. Schema validation

**Manipulation:** validation off, on (fail closed), on with repair, for two simulated models
**Measured:** success rate, cost; n = 600 runs per cell (120 tasks x 5 seeds), CIs over 120 tasks

| Model     | Validation | n   | Success [95% CI]   | Mean cost/run | Cost per success [95% CI] | Notes                                                   |
| --------- | ---------- | --- | ------------------ | ------------- | ------------------------- | ------------------------------------------------------- |
| sim-large | off        | 600 | 81.3% [77.7, 84.7] | 16.46         | 20.24 [18.37, 22.31]      | Dominant failure: too many consecutive tool errors (15) |
| sim-large | on         | 600 | 64.7% [60.3, 69.0] | 13.17         | 20.36 [18.33, 22.49]      | 156 runs failed closed on a schema violation            |
| sim-large | repair     | 600 | 88.3% [85.5, 90.7] | 16.92         | 19.16 [17.39, 20.94]      | 5 validation failures after repair limit                |
| sim-small | off        | 600 | 50.0% [45.2, 54.8] | 1.48          | 2.96 [2.54, 3.42]         | 81 runs ended in repeated tool errors                   |
| sim-small | on         | 600 | 21.0% [17.3, 24.8] | 0.65          | 3.11 [2.60, 3.78]         | 436 runs failed closed                                  |
| sim-small | repair     | 600 | 68.7% [64.5, 72.3] | 1.65          | 2.41 [2.15, 2.69]         | 64 validation failures after repair limit               |

Paired comparisons (second minus first; Wilcoxon p unadjusted):

| Model     | Comparison    | Success difference [95% CI] | p      | Cliff's delta  | Added mean cost [95% CI] |
| --------- | ------------- | --------------------------- | ------ | -------------- | ------------------------ |
| sim-large | repair vs off | +7.0 pp [4.3, 9.7]          | <0.001 | 0.20 (small)   | +2.8% [1.7, 4.0]         |
| sim-large | on vs off     | -16.7 pp [-19.7, -13.7]     | <0.001 | -0.41 (medium) | -20.0% [-23.3, -16.6]    |
| sim-large | repair vs on  | +23.7 pp [20.2, 27.2]       | <0.001 | 0.58 (large)   | +28.5% [23.1, 34.4]      |
| sim-small | repair vs off | +18.7 pp [14.0, 23.2]       | <0.001 | 0.37 (medium)  | +11.5% [7.9, 15.3]       |
| sim-small | on vs off     | -29.0 pp [-33.0, -25.2]     | <0.001 | -0.57 (large)  | -55.9% [-59.0, -52.9]    |
| sim-small | repair vs on  | +47.7 pp [44.0, 51.2]       | <0.001 | 0.85 (large)   | +153.2% [134.2, 174.7]   |

**Decision (H1):** sim-large: **supported** (Holm-adjusted p < 0.001, added cost upper bound 4.0% < 15%). sim-small: success gain confirmed (Holm-adjusted p < 0.001) but the **cost criterion was not met**: the point estimate (11.5%) is under 15% but the 95% interval's upper bound (15.3%) is not.

### E2. Planning strategy

**Manipulation:** reactive vs. plan-then-execute, tasks binned by solution length; validation = repair; sim-large; base arm and a control arm with context growth switched off
**Measured:** success rate by step count; n = 30 tasks x 10 seeds = 300 runs per strategy per bin

Base arm:

| Length bin | Tasks | Reactive success [95% CI] | Plan success [95% CI] | Diff [95% CI]      | p / Holm-adj p | Cliff's delta | Added mean cost [95% CI] |
| ---------- | ----- | ------------------------- | --------------------- | ------------------ | -------------- | ------------- | ------------------------ |
| 3-4        | 30    | 93.7% [91.0, 96.0]        | 97.0% [95.0, 98.7]    | +3.3 pp [0.0, 6.3] | 0.063 / 0.123  | 0.27 (small)  | +13.9% [11.0, 17.2]      |
| 5-6        | 30    | 89.7% [87.0, 92.7]        | 94.7% [91.7, 97.3]    | +5.0 pp [1.0, 9.0] | 0.052 / 0.123  | 0.35 (medium) | +4.8% [2.5, 7.2]         |
| 7-8        | 30    | 88.0% [84.0, 91.7]        | 92.7% [89.7, 95.3]    | +4.7 pp [0.7, 9.0] | 0.031 / 0.123  | 0.24 (small)  | +0.8% [-1.2, 2.8]        |
| 9-10       | 30    | 86.0% [83.3, 88.7]        | 91.0% [87.7, 94.0]    | +5.0 pp [0.7, 9.3] | 0.032 / 0.123  | 0.36 (medium) | -2.0% [-3.7, -0.3]       |

Control arm (no growth of silent mistakes with context):

| Length bin | Tasks | Reactive success [95% CI] | Plan success [95% CI] | Diff [95% CI]       | p / Holm-adj p | Cliff's delta     | Added mean cost [95% CI] |
| ---------- | ----- | ------------------------- | --------------------- | ------------------- | -------------- | ----------------- | ------------------------ |
| 3-4        | 30    | 94.7% [91.7, 97.0]        | 97.0% [95.0, 98.7]    | +2.3 pp [-0.7, 5.3] | 0.187 / 0.746  | 0.17 (small)      | +13.9% [11.0, 17.2]      |
| 5-6        | 30    | 92.0% [89.3, 94.7]        | 94.7% [91.7, 97.3]    | +2.7 pp [-1.0, 6.3] | 0.217 / 0.746  | 0.22 (small)      | +4.8% [2.5, 7.2]         |
| 7-8        | 30    | 91.7% [88.0, 94.7]        | 92.7% [89.7, 95.3]    | +1.0 pp [-3.0, 5.0] | 0.674 / 1.000  | 0.04 (negligible) | +0.9% [-1.0, 2.9]        |
| 9-10       | 30    | 90.7% [88.0, 93.3]        | 91.0% [87.7, 94.0]    | +0.3 pp [-4.0, 4.7] | 0.877 / 1.000  | 0.05 (negligible) | -1.9% [-3.5, -0.3]       |

Model calls per run (base arm): reactive 4.7, 6.9, 9.0, 11.2; plan-then-execute 6.0, 8.1, 10.4, 12.7 for the four bins.

**Decision (H2): not supported.** The unadjusted intervals for the three long bins exclude zero, but no bin survives the Holm correction (adjusted p 0.123 everywhere), and the advantage is flat (about 3 to 5 points) rather than growing with length. In the short bin the difference is similar in size (+3.3 pp), so the "not on shorter tasks" part is also not borne out in direction, only in significance.

### E3. Model routing

**Manipulation:** single large, single small, error-triggered cascade, plan-then-execute on the large model (control), tool-tier routing (plan-then-execute; planning, final answer and write tools on large, simple read tools on small)
**Measured:** cost per success, success rate; n = 600 runs per condition

| Condition         | n   | Success [95% CI]   | Mean cost/run | Cost per success [95% CI] | Model calls/run |
| ----------------- | --- | ------------------ | ------------- | ------------------------- | --------------- |
| single_large      | 600 | 91.0% [88.8, 93.0] | 17.14         | 18.84 [17.24, 20.47]      | 7.97            |
| single_small      | 600 | 68.7% [64.2, 72.8] | 1.65          | 2.41 [2.14, 2.71]         | 8.73            |
| cascade           | 600 | 75.2% [70.8, 79.3] | 5.37          | 7.15 [6.28, 8.07]         | 9.00            |
| plan_single_large | 600 | 93.5% [91.5, 95.3] | 17.31         | 18.52 [17.13, 19.88]      | 9.24            |
| tier_routed       | 600 | 91.5% [89.2, 93.8] | 9.77          | 10.68 [9.97, 11.41]       | 10.28           |

Pre-registered family (Holm over two comparisons against single_large):

| Comparison                  | Success difference [95% CI] | p / Holm-adj p  | Cost-per-success ratio [95% CI] |
| --------------------------- | --------------------------- | --------------- | ------------------------------- |
| cascade vs single_large     | -15.8 pp [-20.0, -12.0]     | <0.001 / <0.001 | 0.380 [0.355, 0.407]            |
| tier_routed vs single_large | +0.5 pp [-2.5, 3.5]         | 0.888 / 0.888   | 0.567 [0.539, 0.596]            |

Descriptive comparisons (outside the family): tier_routed vs plan_single_large: -2.0 pp [-3.7, -0.5], p = 0.041 unadjusted, cost-per-success ratio 0.577 [0.559, 0.596]; single_small vs single_large: -22.3 pp [-26.3, -18.2], ratio 0.128 [0.121, 0.136]; plan_single_large vs single_large: +2.5 pp [-0.5, 5.5], ratio 0.983 [0.945, 1.026].

**Decision (H3):** tool-tier routing: **supported** (ratio upper bound 0.596 < 0.70, no significant drop, lower bound -2.5 pp > -5 pp). Cascade: **not supported** (cost cut of 62% but a 15.8-point success loss). Note the tier-routed condition also changes the strategy; against its own strategy control it shows a small, nominally significant drop (-2.0 pp, unadjusted p 0.041).

### E4. Safety gates

**Manipulation:** six cumulative policies, two simulated-compliance arms, simulated human reviewer
**Measured:** catch rate (attempt level, Wilson 95%), intervention rate, attack success and task success; n = 1,152 runs per policy in arm A (288 scenarios x 4 seeds, 192 attack scenarios) and 576 in arm B (288 x 2)

Policies: P0 no controls; P1 permission scopes only; P2 scopes plus egress and recipient allow-lists; P3 approval gates only; P4 scopes, allow-lists and approval gates; P5 P4 plus a taint gate (side effects after untrusted content need approval) and a quarantining injection detector. The simulated reviewer rejects an unsafe request with probability 0.85 and a legitimate one with probability 0.03 (assumptions).

Arm A (sim-large default: obeys an injection with probability 0.35):

| Policy | Unsafe attempts (n) | Catch rate [95% CI] | Attack success (run level) [95% CI] | Intervention rate [95% CI] | Clean-run task success | Attack-run task success |
| ------ | ------------------- | ------------------- | ----------------------------------- | -------------------------- | ---------------------- | ----------------------- |
| P0     | 350                 | 8.0% [5.6, 11.3]    | 39.8% [36.4, 43.3]                  | 0.0% [0.0, 0.3]            | 95.3%                  | 65.6%                   |
| P1     | 358                 | 86.9% [83.0, 90.0]  | 6.0% [4.5, 7.9]                     | 0.0% [0.0, 0.3]            | 97.4%                  | 92.2%                   |
| P2     | 357                 | 92.4% [89.2, 94.8]  | 3.4% [2.3, 4.9]                     | 0.0% [0.0, 0.3]            | 97.4%                  | 93.9%                   |
| P3     | 367                 | 86.6% [82.8, 89.8]  | 6.3% [4.7, 8.2]                     | 44.5% [41.7, 47.4]         | 95.8%                  | 90.8%                   |
| P4     | 371                 | 98.9% [97.3, 99.6]  | 0.5% [0.2, 1.3]                     | 25.6% [23.2, 28.2]         | 96.4%                  | 95.7%                   |
| P5     | 219                 | 99.1% [96.7, 99.7]  | 0.3% [0.1, 0.9]                     | 49.9% [47.0, 52.8]         | 95.8%                  | 69.9%                   |

Arm B (worst case, always obeys an injection):

| Policy | Unsafe attempts (n) | Catch rate [95% CI] | Attack success (run level) [95% CI] | Intervention rate [95% CI] | Clean-run task success | Attack-run task success |
| ------ | ------------------- | ------------------- | ----------------------------------- | -------------------------- | ---------------------- | ----------------------- |
| P0     | 393                 | 2.5% [1.4, 4.6]     | 99.5% [98.1, 99.9]                  | 0.0% [0.0, 0.7]            | 97.4%                  | 23.7%                   |
| P1     | 394                 | 81.5% [77.3, 85.0]  | 19.0% [15.4, 23.2]                  | 0.0% [0.0, 0.7]            | 97.4%                  | 83.6%                   |
| P2     | 395                 | 92.7% [89.7, 94.8]  | 7.6% [5.3, 10.6]                    | 0.0% [0.0, 0.7]            | 97.4%                  | 88.8%                   |
| P3     | 402                 | 87.8% [84.3, 90.7]  | 12.8% [9.8, 16.5]                   | 74.7% [70.9, 78.0]         | 96.4%                  | 84.4%                   |
| P4     | 405                 | 98.8% [97.1, 99.5]  | 1.3% [0.6, 3.0]                     | 29.2% [25.6, 33.0]         | 96.4%                  | 93.8%                   |
| P5     | 213                 | 98.6% [95.9, 99.5]  | 0.8% [0.3, 2.3]                     | 52.1% [48.0, 56.1]         | 95.8%                  | 69.5%                   |

Where unsafe attempts ended, arm A (counts): P0 322 of 350 executed (the other 28 were rejected by schema validation or failed as ordinary tool errors); P1 292 stopped by scope; P2 290 by scope and 21 by allow-list; P3 282 by approval; P4 293 by scope and 48 by approval; P5 163 by scope and 29 by approval (and 219 attempts in total, because the quarantine removed most plain payloads before the model saw them).

Injection detector (policy P5 only), arm A: plain payloads flagged in 372 of 384 attack runs; paraphrased payloads 0 of 384; clean runs flagged 0 of 384. Payloads and detector were written by the same author, and the paraphrased set was written to avoid the detector's cue words, so these figures describe the mechanism and not real-world recall.

**Decision (H4), policy P3:** arm A: **supported** (catch rate lower bound 82.8% > 50%; intervention rate 44.5% <= 50%). Arm B: catch criterion met (lower bound 84.3%) but the **intervention bound was exceeded** (74.7% > 50%), so not supported as operationalised.

## 8. Discussion

**What the simulation can and cannot say.** Every effect above is the interaction of the platform with assumed model behaviour. The experiments verify that the mechanisms do what they claim (validation catches schema violations and feeds errors back, plans run and replan, routers send calls where they should go, gates stop unsafe effects that really would have landed in the services database) and they show how large the effects are under those assumptions. They do not show how any real model behaves.

**H1.** Validation with repair helped for both simulated models, but how much depended on how often the simulated model produced bad calls (7.0 points at 4% malformed calls, 18.7 at 15%). The unvalidated baseline is not a straw man: some corrupted calls are harmless (an extra parameter is ignored by the handler), and some are silently dangerous (a stringified number in a sum concatenates). The cost criterion failed narrowly for the weaker model: repair adds model calls, and the interval upper bound (15.3%) crossed the limit although the point estimate (11.5%) did not. The surprising result is the fail-closed mode: rejecting an invalid call without feeding the error back lost 17 to 29 points against doing nothing at all, because it converts every harmless schema deviation into a failed run. Strict validation is only worth having together with repair.

**H2.** Plan-then-execute was ahead by roughly 3 to 5 points in every length bin, not only in long ones, and the differences did not survive correction for four comparisons. Two simulator assumptions drive the sign: execution from a plan is assumed to be more focused (half the silent-mistake rate), and reactive mistakes grow with context. In the control arm without the growth term the long-bin gains shrink to 0.3 to 2.7 points with intervals that include zero; a small gain remains in the short bin (+2.3 pp), which points at the focus assumption rather than the growth term. Plan-then-execute was also 14% more expensive on short tasks and about break-even on long ones, because the planning and final-answer calls are overhead. The honest reading is that the platform supports and correctly accounts for both strategies, and that the simulation gives no evidence of a length-dependent advantage.

**H3.** Routing by tool tier cut cost per success by about 43% with no detectable loss in success against the single large model, which is the saving the hypothesis predicted. But the cascade, which looks equally attractive (62% cheaper per success), lost 15.8 points because it only escalates on errors that the platform can see; the simulated small model's silent mistakes produce valid-looking calls that never trigger escalation. This is the practical lesson: routing rules must not rely on error signals alone. Part of the tier-routing result also comes from the strategy change (plan-then-execute on the large model alone scored +2.5 points, non-significant).

**H4.** With the default simulated compliance, approval gates alone stopped 87% of unsafe attempts while asking a human to review 45% of all runs; layering scopes and allow-lists in front of the gates raised the catch rate to 98.9% and cut the human load to 26%, because most attacks never reach a human at all. Approval-only (P3) lets through the reviewer's assumed 15% miss rate. In the worst-case arm the human load in P3 rises to 75% because nearly every attack produces a request. The taint gate and quarantine (P5) pushed catch to 99% but cost usefulness: task success on attacked runs fell to about 70% (against 94 to 96% for P4), because quarantine removes the very text the task needed and the taint gate asks a human about every write after any read. Policy P5 is an example of a control with a real, measurable price.

**Practical guidance (for the platform, not for any model).** Prefer static controls (least-privilege scopes, allow-lists) first and reserve humans for what they cannot express; never use fail-closed validation without repair; do not route on errors alone; measure the intervention rate of every gate, since a gate that asks for every run is not a gate.

## 9. Threats to validity

- **Simulation validity (dominant).** All behaviour is assumed. A different set of parameters would change every number; the verdicts are conditional on Section 4.6. The base-arm advantage in E2 and the cascade failure in E3 are direct consequences of specific assumptions, which is why a control arm and a descriptive comparison are included.
- **Author-written attacks and detector.** The payloads, the detector and the simulator's directive parser were written by one author; detector recall is not evidence about real attacks. Policy catch rates also depend on the assumed reviewer (85% catch, 3% false alarm).
- **Catch-rate definition.** A schema-invalid unsafe attempt that the validator rejects counts as caught (it is not executed). This inflates catch rates slightly in every policy, including P0 (8.0%).
- **Independence.** Tasks repeated over seeds are not independent; E1 to E3 bootstrap over tasks, but E4 Wilson intervals treat attempts and runs as independent and are somewhat optimistic.
- **Plan-step advance rule.** In plan-then-execute a successful call to any tool named in the plan advances the plan. A wrongly planned write can therefore be repeated (one duplicate ticket was seen in development), which penalises plan-then-execute slightly.
- **Conservative budget reservation.** Not used in the experiments (no budget limits were set), so it does not affect them.
- **Mock services understate real-world messiness;** synthetic tasks are simple relative to real workflows.
- **Contention.** The machine was at 100% CPU throughout. Counts, rates and simulated cost units are deterministic given the seeds and unaffected; wall-clock times are not used for any claim.
- **Researcher degrees of freedom.** Decision rules, profile parameters and run counts were fixed in the scripts before the full run; deviations and development-time looks are listed in the change log.

## 10. Reproducibility checklist

- [ ] Code tagged at the commit used for the reported results (no version control was initialised; the results record a SHA-256 code digest instead)
- [x] Environment pinned (package-lock.json; Node 25.9.0 recorded in the results; no container image exists)
- [x] Random seeds and run counts recorded (`summary.json` of each experiment)
- [x] Raw results published (CSV per run in `results/*/runs.csv`); no analysis notebook, the analysis is in the experiment scripts
- [x] Hardware or service configuration described in full (Section 4.2 and `meta` in each `summary.json`)
- [x] One command reproduces every figure and table (`npm run experiments`; see Verification)
- [x] Data sources, licenses and versions listed (all data is synthetic and generated by the scripts; dependencies are in the lockfile)

## 11. References

1. Yao et al. (2023). ReAct: Synergizing Reasoning and Acting in Language Models. ICLR. arXiv:2210.03629.
2. Wang et al. (2023). Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning by Large Language Models. ACL. arXiv:2305.04091.
3. Yao et al. (2024). τ-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains. arXiv:2406.12045.
4. Chen, Zaharia and Zou (2023). FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance. arXiv:2305.05176.
5. Greshake et al. (2023). Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection. arXiv:2302.12173.
6. Debenedetti et al. (2024). AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents. arXiv:2406.13352.
7. Model Context Protocol specification, revision 2025-06-18 (modelcontextprotocol.io).
8. Efron and Tibshirani (1993). An Introduction to the Bootstrap. Chapman and Hall.
9. Wilcoxon (1945). Individual comparisons by ranking methods. Biometrics Bulletin 1(6).
10. Holm (1979). A simple sequentially rejective multiple test procedure. Scandinavian Journal of Statistics 6(2).
11. Cliff (1993). Dominance statistics: ordinal analyses to answer ordinal questions. Psychological Bulletin 114(3).
12. Wilson (1927). Probable inference, the law of succession, and statistical inference. Journal of the American Statistical Association 22(158).

The arXiv identifiers, titles and authors of references 3 to 6 and 2 were checked against the arXiv abstract pages while writing this report; the other entries are cited from memory and were not re-verified.

## 12. Change log

| Date       | Version | Change                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ---------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026-10-21 | v0.1    | Protocol drafted; no results yet.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| 2026-10-22 | v0.2    | Protocol adapted before data collection: the system under test became the built platform driven by a **simulated model** (no LLM API available); hypotheses H1 to H4 given operational decision rules (Section 2.1); latency metrics dropped (no real latency, shared machine) in favour of counts; costs expressed in simulated cost units, not USD; E1 run for two simulated model profiles; E2 gained a control arm; E3 gained a plan-then-execute control and a 5-point non-inferiority margin; E4 defined six cumulative policies and two compliance arms with a simulated reviewer; baselines reduced to the three listed in Section 4.3. |
| 2026-10-22 | v0.2    | Development-time disclosures: (a) harness smoke runs at reduced size (8 to 48 tasks, one seed) were executed while building the scripts and their output inspected for crashes and plausibility only; (b) an evaluation-harness run (`support-core`, success 91% with the default simulated configuration) was executed before the experiments; (c) a harness defect (the simulated clock advancing past undecided approvals, which would have expired them in E4) and a test-isolation defect were found and fixed before the full run. No simulator parameter, hypothesis or decision rule was changed after any of these.                    |
| 2026-10-22 | v0.2    | **Full experiment set run once** (`npm run experiments`, code digest `72c7114a274175fc8464ef601fcd5c348f528df77fbadcc414dbb041c20104c5`). No reruns. Results in `results/e1` to `results/e4`.                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| 2026-10-22 | v0.2    | Post-hoc (no rerun, no rule change): the "other" outcome in E4 was inspected after the run and found to be schema-invalid or otherwise failed tool calls; the definition is stated in Section 7 and Section 9. The tier-routing and plan-strategy confound in E3 was already covered by a control condition added in the protocol.                                                                                                                                                                                                                                                                                                              |

## Verification

Commands that reproduce the reported numbers (they need the source code, not just this report):

```
npm ci
npm run experiments
```

`npm run experiments` runs E1 to E4 in order and overwrites `results/e1` to `results/e4`; the individual commands are `npm run experiment:e1` to `npm run experiment:e4`. A `--smoke` flag (`npm run experiment:e1 -- --smoke`) runs a reduced-size version that writes to a temporary folder and never to `results/`. Results are deterministic given the code and seeds; the `codeDigest` field in each `summary.json` identifies the exact source the reported run used.

---

[README](../README.md) · [Client case study](case-study.md) · [Research report](research-report.md)
