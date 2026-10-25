# Official result: CoreRouter v2 (amendment 1, variant C)

The official result is from RouteWorks/RouterArena#240, run 40211493346, RouterArena's own `pr-evaluation.yml` on upstream
`main` @ `04c99fd`. The unofficial verification run (wmlba/routerarena-verification#2, with the #239 memory fix) gave
identical numbers.

| | Pre-registered prediction (`d87c282`) | Official | Error |
|---|---|---|---|
| RouterArena score | 77.65 (Gemini-calibrated) to 78.01 (v1-calibrated) | **76.43** | −1.22 to −1.58 |
| Accuracy | 79.12% to 79.50% | **77.71%** | −1.41 to −1.79 points |
| Cost per 1K queries | $0.1753 | **$0.1754** | +0.06% |
| Robustness | – | 83.81 | |

- **Against v1:** v1 scored 73.40 (74.12%, $0.1532 per 1K), so v2 is **+3.03**.
- **Against the decision rule:** the rule required a predicted lower bound of 77.15. The official score is below that.

**The prediction missed, outside the ±1.0 back-test tolerance.**

The most likely cause is that the label-free estimator scores answers partly by cross-model agreement. Variant C returns
Qwen's answer whenever Qwen and Gemma agree, which happens on 63% of rows. The estimator therefore overrates C when the two
models share an error.

The back-tests did not exercise agreement-based routing:
- v1 routes with a critic threshold;
- Gemini alone involves no routing.

Before any further prediction is used, the estimator will be back-tested on agreement-routed twin data, where labels exist.
