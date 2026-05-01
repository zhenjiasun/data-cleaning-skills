---
name: clean-features
description: Prepare numeric, categorical, and temporal features for ML — handle missingness, outliers, scaling, and stability.
---

# Clean Features

Use this skill when assembling a feature set for modeling. Most performance gains come from cleaner features, not fancier models.

## Process

1. **Per-feature audit**
   - Missing rate (`diagnose-missing-values`)
   - Outliers (`detect-outliers`)
   - Cardinality (categorical)
   - Distribution shape and tails (numeric)
   - Stability over time (drift)

2. **Decide treatment**
   - Missing → impute / indicator / leave / drop (`treat-missing-values`)
   - Outliers → keep / cap / log-transform / leave; never silently drop
   - High cardinality → bucket rare values, or use target/hash/embedding encoding (`encode-categorical-features`)
   - Skewed numeric → log / Box-Cox / quantile transform if model is sensitive
   - Scale → Standard / MinMax for distance- or gradient-based models; not needed for trees

3. **Check leakage** (`detect-data-leakage`) on every feature
   - Especially aggregations and joins to dimensions

4. **Check feature stability over time**
   - Distribution per period
   - New categories appearing
   - Mean/variance shifts

5. **Document expected ranges**
   - Min, max, plausible mean
   - These become the production validation rules

6. **Persist preprocessing as a reusable artifact**
   - The same imputer / scaler / encoder used in train must be used in production
   - Fit on train only

## Output Format

```
| Feature | Type | %Null | Cardinality | Outliers | Treatment | Encoding | Scaling | Stability | Risk |
```

Plus a saved preprocessing pipeline (sklearn `Pipeline`, equivalent in your framework).

## Rules

- Trees do not need scaling; gradient/distance models do. Don't scale unnecessarily.
- Outlier capping and missing imputation belong inside the pipeline so they reproduce at inference.
- High-cardinality categoricals should be bucketed *before* one-hot, or use target/hash/embedding encoding.
- Every feature must answer: "is this available at prediction time?"
