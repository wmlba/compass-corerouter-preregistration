# Amendment 1: label-free RouterArena calibration, new pool, three frozen variants

Committed before the locked test is read and before any RouterArena evaluation of v2. Hashes are in
`freeze-amendment1.json`; the frozen policy and the predictions are `policy-amendment1.json` and
`arena-prediction-amendment1.json`.

## Why the protocol changes
The twin (practice data) misjudges models by different amounts on RouterArena.

| | Twin accuracy | RouterArena accuracy |
|---|---|---|
| v1 (Qwen3-235B → DeepSeek V4 Pro) | 78.50% | 74.12% (measured) |
| Gemini-3-Flash alone | 80.31% | 78.44% (RouterArena README) |

That 2.5-point gap between models is larger than every difference between router variants measured on twin data. Two
consequences:

- The original decision rule (≥ +3.7 over v1 on a twin-built locked test) inherits the same distortion.
- The back-test in section 4.6 was the intended guard. It had not yet been run.

## What changes

### 1. Label-free RouterArena estimate (replaces the twin-only prediction)
- `experiments/replay/arena_label_free.py`, per model, is a gradient-boosted regressor of the grader score.
  - **Features:** which peer models gave the same final answer, peer presence, number of distinct answers, and answer format.
  - **Training:** twin labels only, prompt-fold CV on twin. Overall error is ≤ 0.007 per model; mean per-category error is ≈ 0.05.
- It is applied to our own models' cached RouterArena answers. RouterArena labels are never read.

### 2. Back-test (section 4.6), run now
Raw estimates are about 1 point low:

| Anchor | Raw estimate | Reference |
|---|---|---|
| v1's actual RouterArena routing | 72.86% | 74.12% measured |
| Gemini alone | 77.59% | 78.44% README |

- **Calibrated on v1:** Gemini alone is predicted at 76.04 vs 75.69 from the README figures (+0.35).
- **Calibrated on Gemini:** v1 is predicted at 73.09 vs the measured 73.40 (−0.31).

Both are within ±1.0. Predictions are reported under both calibrations.

### 3. Pool
Qwen3-235B, Gemma-4-31B, Gemini-3-Flash and DeepSeek V4 Flash.

- DeepSeek V4 Pro is dropped. Its label-free RouterArena estimate is 70.7%, below Qwen, even though it scores 78.3% on
  twin.
- Gemini RouterArena and locked-test answers were generated through the Gemini Batch API. The request is identical to
  RouterArena's `_call_google`.

### 4. Variants (at most 3, section 4.5)
All three are implemented by one function, `route()` in `experiments/replay/corerouter_v2_policy.py`, with the policy
fitted on twin labels only.

- **A. Agree.** Call Qwen and Gemma. If they give the same final answer, return Qwen's. Otherwise call Gemini. Categories
  without comparable answers use the category fallback model.
- **B. Category.** The category's twin pick among the four models, as a single call. λ = 0.02.
- **C. Hybrid.** Same as B, except that where the pick is Gemini, A's Qwen+Gemma agreement check runs first.

### 5. Pre-registered RouterArena predictions

| Variant | Accuracy (raw) | Cost per 1K | Predicted score (calibrated on v1 / on Gemini) |
|---|---|---|---|
| A | 78.53% | $0.2417 | 77.86 / 77.50 |
| B | 78.89% | $0.3387 | 77.70 / 77.35 |
| C | 78.24% | $0.1753 | 78.01 / 77.65 |

### 6. Decision rule (replaces section 4.6)
1. Read the locked test once, scored with the RouterArena score formula on the RouterArena category mix. Report A, B, C
   and each single pool model.
2. Select the variant with the highest locked-test score.
3. Submit it only if both conditions hold:
   - it beats every single pool model on the locked test;
   - its lower predicted RouterArena score is ≥ 77.15, which is also ≥ v1 + 3.7 = 77.10.
4. A v1 comparison on the locked test is not possible: there are no Pro locked answers, and the v1 critic is retired. v1 is
   compared through the back-tested RouterArena prediction instead.
5. After RouterArena evaluates the submission, the official score is reported next to the prediction, whatever it is.

### 7. Shift check (section 4.2)
Section 4.2 was defined for critic-threshold escalation. A, B and C escalate on cross-model disagreement or category,
which carries no calibration that could drift. Gemini-call rates on RouterArena versus twin are therefore reported as a
diagnostic, not as a gate.

### 8. Same router (section 4.3)
- The RouterArena predictions file is produced by `route()` with the frozen policy, over the hash-listed RouterArena
  caches.
- Cascade costs follow v1's disclosed convention: every call is charged, with the extra calls' dollar cost added as
  input-price tokens of the returned model.

### 9. Disclosure of development variants
During development the following were tried, all on twin data except the label-free family:

- 7 VOI-cascade variants (critics v2, v3, and v2+v3; Qwen or Gemma first; with or without critic selection);
- 2,024 agreement-cascade configurations (thresholds and models chosen by CV);
- the label-free RouterArena policy family:
  - 6 single models;
  - 5 category routers;
  - 4 agreement cascades;
  - 2 hybrids.

The three variants above were fixed after that exploration. Only they are read on the locked test.
