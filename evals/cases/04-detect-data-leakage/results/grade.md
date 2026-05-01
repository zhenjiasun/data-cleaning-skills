# Grade — Case 04

## Scoring

| Dimension | Baseline | Skilled | Notes |
| --- | --- | --- | --- |
| Diagnosis depth | 0 | 3 | Baseline kept 4 of the 5 leaking features. Skilled identified all 5 with mechanism explained. |
| Structured output | 1 | 2 | Both sectioned; skilled added a per-feature audit table. |
| Risk flagging | 0 | 3 | Baseline missed leakage entirely *and* recommended random split. Skilled flagged leakage + right-censoring + group-aware split + class imbalance + preprocessing leakage. |
| Avoids destruction | 1 | 2 | Baseline didn't damage data, but recommended a model that would silently overfit. Skilled is non-destructive *and* prevents downstream damage. |
| Reproducibility | 1 | 2 | Baseline gave generic steps. Skilled gave a concrete feature list + sanity-check threshold. |
| **Total** | **3 / 12** | **12 / 12** | |

## Qualitative notes

This is the failure mode that keeps ML engineers up at night. The baseline response would lead the user to:

1. Train a model with `total_payments_made`, `latest_fico`, and `collections_calls` as features.
2. See cross-validation AUC of ~0.99 (because those features encode the label).
3. Ship to production.
4. Watch live AUC collapse to ~0.6, because none of those features exist at application time.
5. Spend two weeks debugging.

The baseline's reasoning ("`total_payments_made` is a strong signal of repayment behavior") sounds plausible but is exactly backward — it's not a signal *of* repayment, it's a *consequence* of repayment, only knowable after the fact.

The skilled response did the one thing that prevents this entire class of bug: it explicitly defined the prediction moment first, then audited every column against it. That's a procedure, not a fact, and it generalizes to every supervised problem.

## Verdict

For ML pipelines, this is the highest-stakes skill in the repo. The base rate of model authors who skip a leakage audit is high; the cost is months of wasted work. Very high value.
