---
name: detect-data-leakage
description: Audit an ML dataset for features, labels, joins, or preprocessing that leak future or target information.
---

# Detect Data Leakage

Use this skill **before** training or evaluating a model. Leakage produces unreasonably good training metrics and brutal production performance.

## Process

1. **Define the prediction moment precisely**
   - What is known *at the instant of prediction*?
   - What is not yet known?
   - What is the outcome window?

2. **Audit every feature**
   - Was the feature value computed using data only available at prediction time?
   - Watch for:
     - Post-outcome fields (`shipped_date` while predicting fraud at order placement)
     - Manually assigned decision fields (`approval_reason`)
     - Features computed using future windows (e.g., 30-day rolling means including the prediction day)
     - Updated-in-place fields that overwrite history (`current_status`)

3. **Audit labels**
   - Is the label observable at the prediction moment? Usually yes — it is the outcome — but the *features* must not be.
   - Are labels delayed? If a churn label takes 30 days to materialize, anything within that window is uncertain.
   - Proxy labels: are you actually predicting the proxy, not the real outcome? (`logged_in` ≠ `engaged`)

4. **Audit splits**
   - For temporal data, use a time-based split — random splits leak.
   - For grouped data (multiple rows per customer), use group-aware splitting so the same entity does not appear in train and test.
   - Stratification should not break temporal order.

5. **Audit preprocessing**
   - Imputation, scaling, encoding, target encoding — all must `.fit` on train only and `.transform` on test.
   - Anything fit globally before splitting is leakage.

6. **Audit joins**
   - Joined dimensions must reflect their state at prediction time, not "now".
   - SCD2 with proper `valid_from / valid_to` is the safe pattern.
   - Backfilled or restated rows in a dim leak history backward.

## Output Format

```
Prediction moment: …
What is known: …
What is unknown: …

Risks found:
| Source | Risk | Severity | Recommendation |

Recommended actions:
- remove feature X
- replace feature Y with as-of-prediction-time version
- switch to time-based split
- refit imputer on train only
```

## Rules

- "Too good to be true" cross-validation results are leakage until proven otherwise.
- Time-series data needs time-based splits — never random.
- Preprocessing steps are model artifacts. They must not see the test set.
- Dimensions used for ML features must support point-in-time lookup. If they don't, stop and fix that first.
- Especially important for: credit risk, fraud, churn, recommendations, forecasting.
