# Skill Evaluation

This directory measures whether the skills in `../skills/` actually change a model's behavior — and whether the change is for the better — versus a plain "clean this data for me" prompt.

## Question

> Does giving Claude a `SKILL.md` produce meaningfully better data-cleaning answers than a plain request?

## Method

For each case we hold the dataset and the user's request constant, and vary only the **system prompt**:

| Variant | System prompt |
| --- | --- |
| `baseline` | Empty (or: "You are a helpful data assistant.") |
| `skilled`  | The relevant `SKILL.md` from `../skills/` |

Both variants receive the same user message: a realistic dirty-data ask. We then grade each response against a per-case rubric.

## Rubric (out of 12)

The same rubric applies to every case so the totals are comparable.

| Dimension | Points | What we look for |
| --- | --- | --- |
| **Diagnosis depth** | 0–3 | Does it identify *what is wrong* (and why) before fixing anything? |
| **Structured output** | 0–2 | Tables / sections that someone else can audit, vs. free prose |
| **Risk flagging** | 0–3 | Silent decisions called out: leakage, grain, time zone, sentinels, fanout, etc. |
| **Avoids destruction** | 0–2 | Doesn't silently drop rows, dedupe, or impute without consent |
| **Reproducibility** | 0–2 | Documents conventions and units; outputs something runnable or a clear next step |
| **Total** | 12 | |

## Cases

| # | Skill under test | Dirty pattern |
| --- | --- | --- |
| 01 | [profile-dataset](cases/01-profile-dataset/) | Mixed customer CSV: nulls, duplicates, sentinels, mixed dates, outliers |
| 02 | [clean-datetime-fields](cases/02-clean-datetime-fields/) | Intraday stock bars with ambiguous time zones |
| 03 | [validate-join](cases/03-validate-join/) | `orders ⨝ customers` with silent row explosion |
| 04 | [detect-data-leakage](cases/04-detect-data-leakage/) | Loan default training set with a post-outcome feature |
| 05 | [resolve-entities](cases/05-resolve-entities/) | Company list with near-duplicate spellings |

## Files per case

```
cases/<n>-<name>/
  input.{csv,md}            # the dirty data
  task.md                   # the user's request (identical across variants)
  baseline-prompt.md        # full prompt for the baseline variant
  skilled-prompt.md         # full prompt for the skilled variant
  rubric.md                 # what a good answer looks like, with the score key
  results/
    baseline.md             # actual model output (no skill)
    skilled.md              # actual model output (with skill)
    grade.md                # rubric scoring + qualitative notes
```

## How to reproduce

1. `pip install anthropic`
2. `export ANTHROPIC_API_KEY=…`
3. `python run.py` to regenerate all `results/*.md` files.
4. `python run.py --case 03` to re-run a single case.
5. Edit `grade.md` files manually — grading is human work; the runner only generates outputs.

## Honesty about the included results

The committed results in `results/` are **single-shot outputs from one author** (Claude Opus 4.7) responding to each prompt configuration once. Not a statistical study. They illustrate the kind of behavioral difference the skill produces; they do not prove statistical significance.

To make this rigorous you would want:

- Multiple samples per case (k ≥ 5) at temperature > 0
- Multiple model versions (Sonnet, Opus, Haiku, GPT-4-class for cross-vendor)
- Blind human grading (graders unaware of which response is which)
- Per-dimension inter-rater agreement before trusting totals

The runner script supports the first two. The last two are on you.

## Summary of included single-shot results

See [summary.md](summary.md).
