---
name: add-data-quality-tests
description: Wire automated quality tests into an ETL/ELT pipeline so regressions are caught at build time, not by stakeholders.
---

# Add Data Quality Tests

Use this skill after profiling and rule validation, when you want findings to *stay* fixed.

## Process

1. **Pick the layer** to enforce at:
   - **Source**: producer guarantees (data contract)
   - **Staging**: type, format, allowed values
   - **Mart**: business invariants, metric stability
   - Tests should be as close to the source of the bug as possible.

2. **Pick test categories** to wire in:
   - **Schema**: column exists, types stable
   - **Not null** on required fields
   - **Unique** on primary keys
   - **Accepted values** on enums (`status IN ('active','closed','frozen')`)
   - **Range**: numeric/date bounds
   - **Referential**: every FK resolves
   - **Freshness**: max(updated_at) within SLA
   - **Volume**: row count within ±X% of rolling average
   - **Distribution**: null rate, mean, distinct count within tolerance (drift)

3. **Pick a tool** and stay in one ecosystem per project:
   - dbt tests (`unique`, `not_null`, `relationships`, `accepted_values`, custom)
   - Great Expectations
   - Soda Core / Cloud
   - Deequ (Spark)
   - Dagster asset checks
   - Airflow `SQLCheckOperator`
   - Plain SQL assertions in your warehouse

4. **Set severity** per test:
   - **Error** — block the pipeline (uniqueness on a PK, freshness on a critical mart)
   - **Warn** — alert but continue (volume drift, rare-but-not-impossible nulls)

5. **Make failures actionable**
   - On failure, surface: which test, which rows, when, runbook link, on-call
   - Failed tests must produce a sample of offending rows, not just a count

## Output Format

A pull request that adds, for each table:

```
- schema definition / contract
- one or more *.sql or *.yml tests
- documentation of severity and owner
- a runbook entry: "if this test fails, do X"
```

## Rules

- Every fix from `validate-business-rules`, `detect-duplicates`, or `check-referential-integrity` should land with a test, or it will regress.
- Do not test what producers already guarantee in a contract; do not assume contracts you have not written.
- Avoid flaky tests with hard cutoffs on natural variance. Use rolling baselines.
- A failing test that nobody acts on is worse than no test — page the right team.
