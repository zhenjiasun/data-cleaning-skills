---
name: treat-missing-values
description: Apply the appropriate treatment — leave null, indicator column, impute, drop, or replace — based on a prior diagnosis.
---

# Treat Missing Values

Use this skill **after** `diagnose-missing-values`. Treatment depends on the cause.

## Process

1. **Confirm classification** from the diagnosis report.

2. **Pick treatment per column**
   - **MCAR, low rate** → median/mean (numeric) or mode (categorical) imputation
   - **MAR** → model-based imputation (KNN, MICE, gradient boosted) using the predictive columns
   - **MNAR / informative** → keep null and add a binary `<col>_was_missing` indicator
   - **Structural** → keep null; add a context column or use a sentinel category like `"N/A"` documented as such
   - **Pipeline** → fix the pipeline, do not impute. Backfill the affected partition.
   - **Time series** → forward-fill / backward-fill / linear interpolation depending on the variable; document edge handling

3. **Decide whether to drop rows**
   - Drop only if missingness is small, MCAR, and the cost of imputation > value of the row
   - Drop is irreversible — log the count and condition

4. **Fit imputation safely (ML)**
   - Fit imputer on **train only**; `.transform` on test/validation/production
   - Treat the imputer as a versioned model artifact
   - Persist the same defaults at inference time

5. **Validate after treatment**
   - Distribution change of the column
   - Effect on downstream metrics or model performance
   - No silent row loss

## Output Format

```
| Column | Class | Treatment | Imputed value / rule | Indicator added? | Rows affected |
```

## Rules

- **Never fit imputation on the test set.** It leaks distribution information backward.
- For ML, an indicator column on an MNAR feature is almost always worth it.
- Replace with `"Unknown"` only when missing genuinely means *unknown*, not just *not collected*.
- If the diagnosis says "pipeline", do not treat — fix.
- Apply the exact same treatment at training and inference; otherwise you have train-serve skew.
