---
name: check-referential-integrity
description: Confirm that foreign keys resolve across tables and surface orphan records, late-arriving dimensions, and broken joins.
---

# Check Referential Integrity

Use this skill when data is split across tables and you need every reference to actually resolve.

## Process

1. **List the references** to check:
   - `orders.customer_id → customers.customer_id`
   - `transactions.account_id → accounts.account_id`
   - `sales.product_id → product_dim.product_id`

2. **Run anti-joins** for each pair: rows in the fact table whose key does not exist in the dimension table.

3. **Quantify and slice the orphans**
   - Orphan count and % of fact rows
   - Time distribution: are orphans recent (late-arriving dim) or historical (hard error)?
   - Source distribution: is one ingestion path bad?

4. **Diagnose the cause**
   - **Late-arriving dimension**: dim built on a slower schedule than fact
   - **Slowly changing dimension mismatch**: SCD2 join missing a `valid_from/valid_to` predicate
   - **Environment mismatch**: prod fact joined to staging dim or vice versa
   - **Hard delete upstream**: dim row deleted but fact still references it
   - **Type mismatch**: `INT` vs `VARCHAR` keys silently coerced

5. **Recommend resolution**
   - Wait/backfill for late-arriving dims
   - Add the missing SCD predicate
   - Soft-delete instead of hard-delete upstream
   - Add a referential integrity test (`add-data-quality-tests`)

## Output Format

```
| FK column | References | Orphans | % of fact | Recent share | Likely cause | Action |
```

## Rules

- An orphan does not always mean bad data — it often means timing. Slice by ingestion time before blaming the source.
- Never inner-join a fact to a dim without first knowing the orphan rate.
- For SCD2 joins, the predicate must include `valid_from <= fact_time < valid_to`.
- Surface orphans via a `dq_orphans` view; do not delete fact rows.
