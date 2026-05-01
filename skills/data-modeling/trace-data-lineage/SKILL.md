---
name: trace-data-lineage
description: Trace a column or metric from its consumer back to its source(s), and identify everything downstream that depends on it.
---

# Trace Data Lineage

Use this skill when:
- A metric changed and you need to find the upstream cause
- A dashboard broke after a deploy
- You are about to change a column and need impact analysis
- You are debugging why a value looks wrong

## Process

1. **Identify the asset of interest** — a column, a model, a dashboard tile, a metric.

2. **Walk upstream**
   - For each input, identify the table and the transformation
   - Continue until you reach raw / source system
   - Note every join, filter, aggregation, and standardization on the way

3. **Walk downstream**
   - Find every model, dashboard, alert, and reverse-ETL sync that depends on the asset
   - Identify which ones the change will break

4. **Use whatever lineage source you have**
   - dbt docs / dbt graph
   - OpenLineage, Atlan, DataHub, Marquez, Monte Carlo, Collibra
   - Warehouse query history (last 90 days of `SELECT … FROM table`)
   - Manual `grep` over the repo as a fallback

5. **Annotate suspicious nodes**
   - Recently changed
   - Owned by a different team
   - No tests
   - Stale / rarely refreshed

## Output Format

```
Asset: <table.column>

Upstream chain:
  raw.source_x.field_a
    -> staging.stg_x.field_a (rename + cast)
    -> int.fact_y.computed_field (joined with stg_z)
    -> mart.fact_y.computed_field

Downstream consumers:
  - dashboard "Q1 Pipeline" tile "Conversion"
  - dbt model mart.exec_summary
  - reverse_etl: salesforce.opportunity.score
```

## Rules

- A change with unknown downstream impact is not safe to deploy.
- Lineage from the warehouse query log is authoritative when the modeling tool's lineage is incomplete.
- Document the lineage in the PR description for any column-shape change.
