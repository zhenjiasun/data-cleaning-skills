---
name: detect-table-grain
description: Determine what one row in a table represents, so aggregations and joins are correct.
---

# Detect Table Grain

Use this skill before any join, aggregation, or model. **If you do not know the grain, your numbers are guesses.**

## Process

1. **Ask: "what does one row mean?"**
   - One row per customer?
   - One row per customer per day?
   - One row per transaction?
   - One row per product per store per week?
   - One row per session per page view?

2. **Validate by uniqueness**
   - Find a candidate grain key (e.g. `customer_id, date`)
   - Check that `COUNT(*) = COUNT(DISTINCT key)` for that key
   - If not, the grain is finer than you thought

3. **Validate by sampling**
   - Pick 10 rows and read them
   - Does the data match the claimed grain?
   - Look for duplicates that suggest the grain is actually a composite

4. **Watch for grain mixing**
   - A table with both header and line-item rows (unioned)
   - A table where some rows are aggregates and others are detail
   - SCD2 tables where grain is `(entity, version)` not `(entity)`

5. **Document**
   - Write the grain in plain English at the top of the model
   - List the unique key
   - List counterexamples and edge cases

## Output Format

```
Table: <name>
Grain (English): "One row per <entity> per <period> per <dimension>"
Unique key: (<col>, <col>, …)
Sample row (interpreted): …
Caveats: …
```

## Rules

- Document grain before you compute on the table. Comments first, code second.
- A `SUM` is only correct if you understand the grain. Summing across mixed grains double-counts.
- If the grain is `(entity, version)`, all joins must include the version predicate.
- "I think it's one row per X" without a uniqueness check is not a grain — it is a hope.
