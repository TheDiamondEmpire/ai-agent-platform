| condition | runs | success [95% CI over tasks] | mean cost/run (units) | cost per success [95% CI] | model calls/run | configuration |
| --- | --- | --- | --- | --- | --- | --- |
| single_large | 600 | 91.0% [88.8%, 93.0%] | 17.14 | 18.84 [17.24, 20.47] | 7.97 | reactive, every call on sim-large (the reference) |
| single_small | 600 | 68.7% [64.2%, 72.8%] | 1.65 | 2.41 [2.14, 2.71] | 8.73 | reactive, every call on sim-small |
| cascade | 600 | 75.2% [70.8%, 79.3%] | 5.37 | 7.15 [6.28, 8.07] | 9.00 | reactive; sim-small first, escalate to sim-large for the turn after an error |
| plan_single_large | 600 | 93.5% [91.5%, 95.3%] | 17.31 | 18.52 [17.13, 19.88] | 9.24 | control: plan-then-execute, all calls on sim-large |
| tier_routed | 600 | 91.5% [89.2%, 93.8%] | 9.77 | 10.68 [9.97, 11.41] | 10.28 | plan-then-execute; planning, final answer and write tools on sim-large, simple read tools on sim-small |

| comparison | success diff [95% CI] | Wilcoxon p | Holm-adj p (2 comparisons) | cost-per-success ratio routed/large [95% CI] |
| --- | --- | --- | --- | --- |
| cascade vs single_large | -15.8 pp [-20.0 pp, -12.0 pp] | <0.001 | <0.001 | 0.380 [0.355, 0.407] |
| tier_routed vs single_large | 0.5 pp [-2.5 pp, 3.5 pp] | 0.888 | 0.888 | 0.567 [0.539, 0.596] |

Descriptive comparisons (not part of the pre-registered family):

| comparison | success diff [95% CI] | Wilcoxon p (unadjusted) | cost-per-success ratio [95% CI] |
| --- | --- | --- | --- |
| tier_routed vs plan_single_large | -2.0 pp [-3.7 pp, -0.5 pp] | 0.041 | 0.577 [0.559, 0.596] |
| single_small vs single_large | -22.3 pp [-26.3 pp, -18.2 pp] | <0.001 | 0.128 [0.121, 0.136] |
| plan_single_large vs single_large | 2.5 pp [-0.5 pp, 5.5 pp] | 0.229 | 0.983 [0.945, 1.026] |

Verdicts:

```json
[
  {
    "id": "H3 (cascade)",
    "rule": "upper bound of the 95% CI of (cost per success routed / large) < 0.70 AND (success diff >= 0 OR Holm-adjusted p >= 0.05) AND lower bound of success-diff 95% CI > -5 pp",
    "costPerSuccessRatio": {
      "estimate": 0.37951249233421036,
      "lo": 0.35483039297249275,
      "hi": 0.40674286613572447,
      "n": 120
    },
    "successDiff": {
      "estimate": -0.15833333333333324,
      "lo": -0.19999999999999982,
      "hi": -0.11999999999999994,
      "n": 120
    },
    "holmAdjustedP": 3.3305358471125146e-11,
    "costReductionCriterionMet": true,
    "noSignificantDropCriterionMet": false,
    "nonInferiorityCriterionMet": false,
    "verdict": "not supported"
  },
  {
    "id": "H3 (tier_routed)",
    "rule": "upper bound of the 95% CI of (cost per success routed / large) < 0.70 AND (success diff >= 0 OR Holm-adjusted p >= 0.05) AND lower bound of success-diff 95% CI > -5 pp",
    "costPerSuccessRatio": {
      "estimate": 0.5670490322105349,
      "lo": 0.5390576687637816,
      "hi": 0.5959923791291248,
      "n": 120
    },
    "successDiff": {
      "estimate": 0.004999999999999997,
      "lo": -0.025,
      "hi": 0.034999999999999996,
      "n": 120
    },
    "holmAdjustedP": 0.8883357967185912,
    "costReductionCriterionMet": true,
    "noSignificantDropCriterionMet": true,
    "nonInferiorityCriterionMet": true,
    "verdict": "supported"
  }
]
```
