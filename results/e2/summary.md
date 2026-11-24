### Arm: base (default sim-large profile (slip rate grows with context: slipCtxGrowth = 0.15 per prior call))

| length bin (tool calls) | tasks | reactive success [95% CI] | plan-then-execute success [95% CI] | model calls/run reactive | model calls/run plan |
| --- | --- | --- | --- | --- | --- |
| 3-4 | 30 | 93.7% [91.0%, 96.0%] | 97.0% [95.0%, 98.7%] | 4.7 | 6.0 |
| 5-6 | 30 | 89.7% [87.0%, 92.7%] | 94.7% [91.7%, 97.3%] | 6.9 | 8.1 |
| 7-8 | 30 | 88.0% [84.0%, 91.7%] | 92.7% [89.7%, 95.3%] | 9.0 | 10.4 |
| 9-10 | 30 | 86.0% [83.3%, 88.7%] | 91.0% [87.7%, 94.0%] | 11.2 | 12.7 |

| length bin | success diff plan-reactive [95% CI] | Wilcoxon p | Holm-adj p (4 bins) | Cliff's delta | added mean cost [95% CI] | cost per success reactive vs plan |
| --- | --- | --- | --- | --- | --- | --- |
| 3-4 | 3.3 pp [0.0 pp, 6.3 pp] | 0.063 | 0.123 | 0.27 (small) | 13.9% [11.0%, 17.2%] | 8.91 vs 9.80 |
| 5-6 | 5.0 pp [1.0 pp, 9.0 pp] | 0.052 | 0.123 | 0.35 (medium) | 4.8% [2.5%, 7.2%] | 14.96 vs 14.85 |
| 7-8 | 4.7 pp [0.7 pp, 9.0 pp] | 0.031 | 0.123 | 0.24 (small) | 0.8% [-1.2%, 2.8%] | 22.43 vs 21.47 |
| 9-10 | 5.0 pp [0.7 pp, 9.3 pp] | 0.032 | 0.123 | 0.36 (medium) | -2.0% [-3.7%, -0.3%] | 31.39 vs 29.05 |

### Arm: no_context_growth (control: slipCtxGrowth = 0 (no degradation with context length))

| length bin (tool calls) | tasks | reactive success [95% CI] | plan-then-execute success [95% CI] | model calls/run reactive | model calls/run plan |
| --- | --- | --- | --- | --- | --- |
| 3-4 | 30 | 94.7% [91.7%, 97.0%] | 97.0% [95.0%, 98.7%] | 4.7 | 6.0 |
| 5-6 | 30 | 92.0% [89.3%, 94.7%] | 94.7% [91.7%, 97.3%] | 6.9 | 8.1 |
| 7-8 | 30 | 91.7% [88.0%, 94.7%] | 92.7% [89.7%, 95.3%] | 9.0 | 10.4 |
| 9-10 | 30 | 90.7% [88.0%, 93.3%] | 91.0% [87.7%, 94.0%] | 11.2 | 12.7 |

| length bin | success diff plan-reactive [95% CI] | Wilcoxon p | Holm-adj p (4 bins) | Cliff's delta | added mean cost [95% CI] | cost per success reactive vs plan |
| --- | --- | --- | --- | --- | --- | --- |
| 3-4 | 2.3 pp [-0.7 pp, 5.3 pp] | 0.187 | 0.746 | 0.17 (small) | 13.9% [11.0%, 17.2%] | 8.82 vs 9.80 |
| 5-6 | 2.7 pp [-1.0 pp, 6.3 pp] | 0.217 | 0.746 | 0.22 (small) | 4.8% [2.5%, 7.2%] | 14.58 vs 14.85 |
| 7-8 | 1.0 pp [-3.0 pp, 5.0 pp] | 0.674 | 1.000 | 0.04 (negligible) | 0.9% [-1.0%, 2.9%] | 21.50 vs 21.47 |
| 9-10 | 0.3 pp [-4.0 pp, 4.7 pp] | 0.877 | 1.000 | 0.05 (negligible) | -1.9% [-3.5%, -0.3%] | 29.74 vs 29.05 |

Verdict:

```json
[
  {
    "id": "H2",
    "rule": "base arm: for each of the bins 5-6, 7-8, 9-10 the success diff (plan - reactive) has 95% CI lower bound > 0 and Holm-adjusted p < 0.05; AND for bin 3-4 there is no significant positive difference",
    "longBinsAllSignificantlyBetter": false,
    "shortBinNoSignificantGain": true,
    "verdict": "not supported",
    "controlArmLongBinsAnyGain": false,
    "caveat": "The base-arm advantage is produced by the simulator assumption that reactive slip rates grow with context; the control arm shows what happens without it. This tests that the platform runs and accounts for both strategies correctly, not that plan-then-execute helps a real model."
  }
]
```
