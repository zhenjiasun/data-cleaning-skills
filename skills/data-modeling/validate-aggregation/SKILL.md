---
name: validate-aggregation
description: Confirm that sums, averages, ratios, and distinct counts are computed correctly given the data's grain.
---

# Validate Aggregation

Use this skill when computing metrics, especially after joins.

## Process

1. **Confirm grain** of the input table (use `detect-table-grain`).

2. **Pick the correct aggregation function**
   - Counts of users → `COUNT(DISTINCT user_id)`
   - Counts of events → `COUNT(*)`
   - Money → `SUM` on lowest-grain rows; never `SUM` of pre-aggregated rows without checking

3. **Catch the classic mistakes**
   - **Averaging averages**: `AVG(daily_avg)` ≠ overall average
   - **Summing ratios**: `SUM(rate)` is meaningless; recompute the ratio at the right grain
   - **Double-counting after a fanout join**: pre-aggregate one side first
   - **Mixing grains**: user-level and event-level data unioned, then summed
   - **Counting rows for distinct entities**: use `COUNT(DISTINCT)`

4. **Reconstruct ratios from numerator and denominator**
   ```
   conversion_rate = SUM(converted_users) / SUM(eligible_users)
   ```
   not
   ```
   AVG(row_level_conversion_rate)
   ```

5. **Cross-check**
   - Do the totals reconcile against a known source of truth (finance system, source database)?
   - Are slice sums equal to the total? `SUM over all slices == grand total`?

## Output Format

```
Metric: …
Source grain: …
Aggregation: …
Numerator: …
Denominator: …
Reconciliation check: …
Known caveats: …
```

## Rules

- Build ratios from raw numerator and denominator at the correct grain. Do not average ratios.
- After any join, re-confirm the grain before aggregating.
- Reconcile to an external source of truth at least once per metric.
- For analyses, write out the numerator and denominator explicitly — even if the SQL is shorter without them.
