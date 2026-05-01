---
name: validate-business-rules
description: Check that data obeys domain logic (e.g. ship_date >= order_date, revenue = qty × price) and surface violations.
---

# Validate Business Rules

Use this skill when you need to confirm data respects domain logic. This is where domain knowledge matters most — schema and types are not enough.

## Process

1. **Elicit rules** from the data owner, docs, or system behavior. Common categories:
   - **Temporal**: `order_date <= ship_date`, `start_date <= end_date`
   - **Arithmetic**: `revenue = quantity × price`, `discount <= list_price`
   - **State machine**: `account_close_date IS NULL when status = 'active'`
   - **Range**: `customer_age >= 0`, `0 <= probability <= 1`
   - **Mutual exclusion**: a row cannot be both `refunded` and `pending`
   - **Existence**: a paid order must have a payment record

2. **Translate each rule into a runnable check** (SQL, dbt test, Great Expectations, Python assertion).

3. **Run the checks and quantify violations**
   - Count and percentage of rows violating each rule
   - Time distribution of violations (recent? historical? specific source?)
   - Sample of violating rows with all relevant columns

4. **Classify each violation**
   - True data error (fix upstream or in staging)
   - Stale rule (rule needs updating)
   - Edge case the rule did not anticipate
   - Pipeline bug

5. **Propose action per rule**
   - Add an automated test (`add-data-quality-tests`)
   - File an upstream bug
   - Quarantine violating rows
   - Update the rule and re-run

## Output Format

```
| Rule | Expression | Violations | % | First seen | Likely cause | Action |
```

Plus, for each rule, 3–5 sample violating rows.

## Rules

- Write rules in plain English first, then translate to code. Otherwise rules drift from intent.
- Never silently filter violators out of a mart — surface them.
- A rule that fails 100% of the time is a wrong rule, not a wrong dataset.
- Promote validated rules into automated tests so this audit becomes continuous.
