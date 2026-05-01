---
name: detect-outliers
description: Identify suspicious numeric, categorical, or temporal values without auto-deleting them, and propose a triage plan.
---

# Detect Outliers

Use this skill when numeric values, dates, or categories look implausible. The goal is to **flag and explain**, not to silently drop.

## Process

1. **Statistical outliers**
   - IQR rule (1.5×IQR beyond Q1/Q3)
   - Z-score > 3 or robust z-score (MAD)
   - Percentile cutoffs (top/bottom 0.1%) for heavy-tailed distributions

2. **Domain outliers**
   - `age = 250`, `price = -1`, `quantity = 0`, `signup_date = 1900-01-01`
   - Values outside the physically/logically possible range

3. **Sentinel outliers**
   - `-1`, `-999`, `9999`, `99999999`
   - These are *encoded missingness*, not real values. Treat with `treat-missing-values`.

4. **Pattern outliers**
   - Transaction 10,000× the user's normal
   - Sudden change in event frequency for a single user
   - Categorical value that appears once in millions of rows

5. **Likely-cause classification**
   - Bug in upstream system
   - Test/staging data leaked into prod
   - Legitimate but rare event (do not drop)
   - Timezone/encoding artifact
   - Sentinel for missing

## Output Format

```
| Column | Outlier rule | Count | Sample values | Likely cause | Recommended action |
```

`Recommended action` is one of: `keep`, `cap/winsorize`, `treat as missing`, `route to review`, `drop with audit`.

## Rules

- Do not auto-drop. Always produce the report first.
- A "rare but real" event (fraud, viral spike) is exactly what the model should learn from — do not winsorize it away.
- Sentinels go through `treat-missing-values`, not outlier handling.
- For ML, document the outlier rule and apply it consistently in train and prediction time.
