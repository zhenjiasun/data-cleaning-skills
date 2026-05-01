# Grade — Case 04

## Scoring

| Dimension | Baseline | Skilled | Notes |
| --- | --- | --- | --- |
| Diagnosis depth | 3 | 3 | Both flagged all five leaks (`loan_status`, `total_payments_made`, `latest_fico`, `collections_calls`, `decision_date`) with mechanism explained. |
| Structured output | 2 | 2 | Both used tables and code blocks. |
| Risk flagging | 3 | 3 | Both called out random-vs-time-based split. Both addressed class imbalance (baseline used `scale_pos_weight`; skilled noted "expect AUC 0.65–0.78"). |
| Avoids destruction | 2 | 2 | Neither modified data; both gave a code skeleton. |
| Reproducibility | 2 | 2 | Both included runnable code. |
| **Total** | **12 / 12** | **12 / 12** | |

## Qualitative notes

**The skill made no measurable difference here.** Real Claude Opus 4.7 already treats leakage detection as central to ML data prep — it opened the baseline response with "🚨 Critical Issue: Target Leakage" before being asked.

This is honest evidence about where skills do and do not help. For a frontier model with strong general ML hygiene, a leakage-audit skill is largely redundant. The skill might still matter for:
- weaker / older / smaller models
- domains where the leak is subtle (medical, finance with derived features)
- multi-step pipelines where the leak hides behind a join several layers up

Both responses are excellent. Tie.

## Verdict

Zero lift on this case with this model. Don't over-claim — the skill saved nothing here.
