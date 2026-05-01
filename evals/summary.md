# Eval Summary

Single-shot comparison of plain "clean this for me" prompts vs the matching `SKILL.md` prompt, scored against a per-case rubric (max 12 each).

## Aggregate

| Case | Skill under test | Baseline | Skilled | Δ |
| --- | --- | --- | --- | --- |
| [01](cases/01-profile-dataset/results/grade.md) | profile-dataset | 3 | 12 | **+9** |
| [02](cases/02-clean-datetime-fields/results/grade.md) | clean-datetime-fields | 3 | 12 | **+9** |
| [03](cases/03-validate-join/results/grade.md) | validate-join | 2 | 12 | **+10** |
| [04](cases/04-detect-data-leakage/results/grade.md) | detect-data-leakage | 3 | 12 | **+9** |
| [05](cases/05-resolve-entities/results/grade.md) | resolve-entities | 5 | 12 | **+7** |
| **Total** | | **16/60** | **60/60** | **+44** |

## Where the lift comes from

Looking across all five cases, the baseline failures cluster into the same five patterns. The skilled variant blocks each of them.

| Failure pattern | Baseline saw it in | Skill prevents it via |
| --- | --- | --- |
| **Silent fabrication** — coercing impossible values into plausible-looking ones (`2023-13-01` → `2023-01-13`, `2024-02-30` → `2024-03-01`) | Case 01 | "Diagnose first, do not modify without confirmation" + Questions-for-owner section |
| **Wrong-default time zone** — assuming UTC when the column is local | Case 02 | "Convert vs localize" rule + cross-reference verification |
| **Silent fanout** — joining without checking dim uniqueness | Case 03 | "Check key uniqueness on each side before joining" |
| **Label leakage** — using post-outcome features as input | Case 04 | "Define the prediction moment first; classify every column against it" |
| **Destructive deduplication** — dropping originals instead of producing a links table | Case 05 | "Resolution lives in a mapping layer; raw records are preserved" |

The pattern: every failure is a model **defaulting to action** when the dataset needed **diagnosis first**. The skills work by inverting that default.

## Quantitative observations

- **Average lift: +8.8 / 12 (≈ 73 percentage points)** in this single-shot run.
- Largest lift (Case 03, +10): SCD2 fanout with a wrong dollar total.
- Smallest lift (Case 05, +7): the easy clusters were obvious enough that the baseline got most of the merging right by intuition.
- The baseline produced **outputs that look correct** in every case — that's the failure mode that matters. Wrong-and-confident is more dangerous than wrong-and-uncertain.

## What's the same across both variants

- Knowledge of pandas / SQL syntax: identical.
- Speed of generating code: comparable.
- General politeness and formatting: comparable.

The skills do **not** make the model smarter at SQL or pandas. They make it more **disciplined** about *what to verify before* writing the SQL.

## Caveats (read before citing this)

- **n = 1 per cell.** Single-shot, single-author, single-grader. Numbers indicate direction, not magnitude with a confidence interval.
- **The grader (Claude) is also the model under test.** Self-grading can be biased. To make this rigorous: blind human grading.
- **The rubric was written by the same author as the skills.** It rewards the format the skills produce. A more neutral rubric would test for *correctness of the eventual cleaned data*, not just the response shape.
- **Cases are crafted to expose specific dirty-data patterns.** Real-world prompts may not exercise the skills as cleanly.

## How to make this rigorous

The runner script (`run.py`) supports the easy parts:

- `--samples 5` for variance per cell
- `EVAL_MODEL=claude-sonnet-4-6 python run.py` for cross-model comparison

What still requires human work:
- Blind grading by people who didn't write the skills
- A held-out case set written by someone other than the skill author
- Inter-rater agreement before trusting per-dimension scores

## Bottom-line interpretation

In the slice of dirty-data tasks these cases represent, equipping a model with the matching `SKILL.md` shifts behavior from "eager intern who breaks data" toward "cautious analyst who diagnoses first" — and that shift maps directly to the failure modes that cause the most pain in real data work.

The skills are most valuable when:

1. The cost of a silent wrong answer is high (finance totals, ML in production, customer master data).
2. The dataset has **traps that look fine** — wrong time zones, fanout joins, leaking features.
3. The user *would not have noticed* the silent error in the baseline output.

They are least valuable when:

1. The data is genuinely simple and the cleanup is obvious.
2. The user explicitly wants a quick best-effort pass and accepts the risk.

For the project this repo lives next to (`stockprediction`), all five failure patterns map onto real recorded incidents — most directly the time-zone trap in Case 02, which is exactly the bug the project's own `CLAUDE.md` warns about.
