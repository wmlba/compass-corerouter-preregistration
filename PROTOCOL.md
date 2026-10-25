# Compass CoreRouter v2: pre-registered evaluation protocol

Version 0, committed 2026-10-25, before any CoreRouter v2 answer on RouterArena is scored.

This repository timestamps the decisions behind the next Compass CoreRouter submission to
[RouterArena](https://github.com/RouteWorks/RouterArena), so anyone can check they were fixed in advance.

## Status of v1 (disclosed)
- **Submission:** `compass-corerouter` (RouteWorks/RouterArena#236, resubmitted as #237).
- **Unofficial run.** Scored with RouterArena's own, unchanged workflow plus the LiveCodeBench memory fix proposed in RouteWorks/RouterArena#239, on GitHub's standard runner ([run](https://github.com/wmlba/routerarena-verification/actions/runs/40162661126)).
- **Result:** score 73.40, accuracy 74.12%, $0.153 per 1K queries, robustness 81.90.
- **Our forecast was too high.** Our own twin cross-validation over 67% of the Arena category mix gave 79.0. The postmortem attributes the gap to:
  - uncovered task categories;
  - the escalation rate under distribution shift;
  - proxy datasets;
  - selection optimism.
  Section 4 is designed so this cannot recur.

## 1. Data rules
- **RouterArena is evaluation-only.**
  - No RouterArena item, label, score or cached result is used for training, tuning or model selection.
  - No other router's per-prompt predictions or evaluations are used.
- **Training and validation data** are public train/dev splits, rendered with RouterArena's published prompt templates.
  - Every item is overlap-screened against RouterArena's Context, Question and Options text. Answer columns are never read.
- **Non-Arena items from benchmark families that RouterArena also samples** (MMLU val/dev, GSM8K train, GeoBench, AIME/AMC from NuminaMath, LiveCodeBench releases after release_v2, FLORES+/NTREX) are used only after that screen, and are disclosed.
- **Licences:** only licences compatible with commercial use. Sources under CC-BY-NC licences are excluded.

## 2. Models
- **Prices:** RouterArena list prices from `model_cost.json` only.
- **Self-hosted open models** are served from the original released checkpoints at native precision, with no quantized repositories and no runtime quantization.
- **Request format:** each call carries one user message and the provider's default sampling, as RouterArena's own inference does. Gemini calls use thinking budget 0, as RouterArena's Google path does.
- **Cost accounting:** every model call a request makes is billed in the submission's cost accounting, including cascade calls whose answers are discarded.

## 3. Router
- **Decision model:** a typed, answer-aware value-of-information cascade. It sees the prompt and every answer observed so far, then either returns an answer or calls the model with the highest expected improvement net of λ·cost.
- **Parameters:** λ, the escalation rules and the model pool are fixed on twin data before the freeze in section 5.

## 4. Validation gates
All of these must pass before submission:
1. **Coverage.**
   - Twin categories cover ≥95% of RouterArena prompts by category, and all prompt-template groups.
   - At least 100 twin prompts in each large MMLU-Pro subject.
2. **Shift check, label-free.** On RouterArena inputs, using no labels:
   - the router's escalation rate is within ±5 points of its twin rate;
   - no category differs by more than 15 points.
3. **Same router.** The router that is validated is byte-identical to the one that is submitted (hash-pinned).
4. **Locked test.**
   - A fresh twin test of at least 4,000 prompts, hash-locked in this repository before it is read.
   - It is read exactly once, after the freeze.
5. **Variant cap.** At most 3 router variants are compared on the locked test. The total number of variants tried during development is disclosed.
6. **Decision rule.**
   - v2 is submitted only if, on the locked test, it beats v1 by at least 3.7 RouterArena-score points.
   - Its prediction must also be at least 77.15 plus the bias measured by replaying the protocol on v1. The back-test must reproduce v1's 73.40 ± 1.0.
7. **Traceability.** Every reported number is backed by a saved artifact with code hashes.

## 5. Freezes
Freeze manifests (critic checkpoint hashes, policy file, λ, price-table version, locked-test hash) will be appended
here as `freeze-*.json` before the corresponding read.
