### sim-large

| validation | runs | success rate (95% CI over tasks) | mean cost/run | cost per success [CI] | model calls/run | schema violations/run | top failure causes (count) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| off | 600 | 81.3% [77.7%, 84.7%] | 16.46 | 20.24 [18.37, 22.31] | 7.77 | 0.000 | failed: too_many_errors (15); succeeded: final answer does not state the expected refund window (4) |
| on | 600 | 64.7% [60.3%, 69.0%] | 13.17 | 20.36 [18.33, 22.49] | 6.44 | 0.260 | failed: validation_failed (156); succeeded: final answer does not state the expected total (6) |
| repair | 600 | 88.3% [85.5%, 90.7%] | 16.92 | 19.16 [17.39, 20.94] | 7.91 | 0.322 | failed: validation_failed (5); succeeded: final answer does not state the expected refund window (4) |

| comparison (second minus first) | success diff [95% CI] | Wilcoxon p (unadjusted) | Cliff's delta | added mean cost [95% CI] |
| --- | --- | --- | --- | --- |
| repair_vs_off | 7.0 pp [4.3 pp, 9.7 pp] | <0.001 | 0.20 (small) | 2.8% [1.7%, 4.0%] |
| on_vs_off | -16.7 pp [-19.7 pp, -13.7 pp] | <0.001 | -0.41 (medium) | -20.0% [-23.3%, -16.6%] |
| repair_vs_on | 23.7 pp [20.2 pp, 27.2 pp] | <0.001 | 0.58 (large) | 28.5% [23.1%, 34.4%] |

### sim-small

| validation | runs | success rate (95% CI over tasks) | mean cost/run | cost per success [CI] | model calls/run | schema violations/run | top failure causes (count) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| off | 600 | 50.0% [45.2%, 54.8%] | 1.48 | 2.96 [2.54, 3.42] | 8.15 | 0.000 | failed: too_many_errors (81); succeeded: final answer does not state the expected total (11) |
| on | 600 | 21.0% [17.3%, 24.8%] | 0.65 | 3.11 [2.60, 3.78] | 4.15 | 0.727 | failed: validation_failed (436); succeeded: final answer does not state the expected refund window (4) |
| repair | 600 | 68.7% [64.5%, 72.3%] | 1.65 | 2.41 [2.15, 2.69] | 8.72 | 1.525 | failed: validation_failed (64); succeeded: final answer does not state the expected total (11) |

| comparison (second minus first) | success diff [95% CI] | Wilcoxon p (unadjusted) | Cliff's delta | added mean cost [95% CI] |
| --- | --- | --- | --- | --- |
| repair_vs_off | 18.7 pp [14.0 pp, 23.2 pp] | <0.001 | 0.37 (medium) | 11.5% [7.9%, 15.3%] |
| on_vs_off | -29.0 pp [-33.0 pp, -25.2 pp] | <0.001 | -0.57 (large) | -55.9% [-59.0%, -52.9%] |
| repair_vs_on | 47.7 pp [44.0 pp, 51.2 pp] | <0.001 | 0.85 (large) | 153.2% [134.2%, 174.7%] |

Hypotheses (Holm correction across the two primary comparisons):

| hypothesis | success diff | Holm-adj p | added cost | verdict |
| --- | --- | --- | --- | --- |
| H1 (sim-large) | 7.0 pp [4.3 pp, 9.7 pp] | <0.001 | 2.8% [1.7%, 4.0%] | supported |
| H1 (sim-small) | 18.7 pp [14.0 pp, 23.2 pp] | <0.001 | 11.5% [7.9%, 15.3%] | success gain confirmed but cost criterion not met |
