---
name: diagnose-missing-values
description: Classify why values are missing — random, pattern-driven, structural, or pipeline failure — before deciding how to treat them.
---

# Diagnose Missing Values

Use this skill **before** imputing or dropping anything. Treatment without diagnosis hides bugs.

## Process

1. **Quantify**
   - % null per column
   - Co-missingness: which columns are null together?
   - Time pattern: did null rate change after a date? (often a deploy/source change)

2. **Classify the missingness**
   - **MCAR** — missing completely at random; null rate independent of any column
   - **MAR** — missing related to other observed variables (e.g., income missing more often for users without verified email)
   - **MNAR** — missing because of the value itself (high earners decline to disclose income)
   - **Structural** — field is not applicable (`shipping_address` for a digital-only purchase)
   - **Pipeline** — broken ingestion, failed join, renamed source field, schema drift

3. **Test for MAR vs MCAR**
   - Group by other columns; does null rate vary significantly?
   - Significant variation suggests MAR — the missingness carries information.

4. **Test for pipeline causes**
   - Did the null rate jump on a specific date?
   - Is it 100% null in a recent partition?
   - Is there a corresponding source-side change?

5. **Test for sentinels masquerading as values**
   - `-1`, `-999`, `9999`, `"unknown"`, empty string
   - These count as "present" to most tools but are semantically null.

## Output Format

```
| Column | %Null | Class (MCAR/MAR/MNAR/Structural/Pipeline) | Evidence | Recommendation |
```

## Rules

- A jump in null rate on a specific date is a pipeline issue until proven otherwise — investigate before imputing.
- "Missing" can be informative. For ML, an indicator column often beats imputation when missingness is MNAR.
- Sentinels are missing values. Convert them before profiling, or stats will be wrong.
- Only after diagnosis should you call `treat-missing-values`.
