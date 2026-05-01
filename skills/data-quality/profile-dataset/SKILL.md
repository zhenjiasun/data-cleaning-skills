---
name: profile-dataset
description: Generate a structured data profiling report for a raw dataset before cleaning or modeling.
---

# Profile Dataset

Use this skill when the user provides a new dataset — a table, CSV, parquet file, SQL query result, or dataframe — and asks to understand its quality. Diagnose first; do not clean yet.

## Process

1. **Inspect structure**
   - Count rows and columns
   - List column names
   - Infer current data types
   - Identify likely primary keys (single or composite)

2. **Profile missingness**
   - Missing count and percentage per column
   - Columns with suspiciously high null rates
   - Rows with many missing fields (often a separate failure mode)

3. **Profile uniqueness**
   - Unique value count per column
   - Duplicate rows
   - Duplicate primary keys
   - High-cardinality categorical fields (often free text disguised as a category)

4. **Profile numeric fields**
   - Min, max, mean, median, stddev
   - Zero count, negative count
   - Outlier candidates
   - Suspicious sentinel values: `-1`, `-999`, `9999`, `99999999`, `unknown`

5. **Profile categorical fields**
   - Top values and their counts
   - Rare values (often typos)
   - Inconsistent casing (`USA` vs `usa`)
   - Inconsistent spelling (`United States` vs `U.S.`)
   - Empty strings and whitespace-only values (different from NULL)

6. **Profile date/time fields**
   - Min and max date
   - Invalid dates
   - Future dates (where unexpected)
   - Time zone presence/ambiguity
   - Mixed date formats

7. **Produce final report**
   - Executive summary
   - Highest-risk columns
   - Cleaning recommendations (do not execute yet)
   - Open questions for the data owner
   - Suggested automated quality tests

## Output Format

```
# Profile Report: <dataset name>

## Summary
- Rows: …
- Columns: …
- Likely primary key: …
- Top 3 quality risks: …

## Column-level table
| Column | Type | %Null | Unique | Min | Max | Top values | Issues |

## Issues
- …

## Recommendations
- …

## Questions for data owner
- …

## Suggested tests
- …
```

## Rules

- Do not modify data unless explicitly asked.
- Separate **observation** from **recommendation**.
- Do not assume missing values should be filled — diagnose first (`diagnose-missing-values`).
- If the dataset is for ML, flag any column that looks like it could be post-outcome (`detect-data-leakage`).
- Sentinel values like `-1` or `9999` count as missing for stats, but report them separately.
