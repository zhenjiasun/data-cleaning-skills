---
name: build-clean-data-model
description: Convert raw data into trustworthy analytics-ready tables via raw → staging → intermediate → mart layers.
---

# Build Clean Data Model

Use this skill when designing a new pipeline or refactoring a tangled one. The layered structure matches dbt-style workflows.

## Layers

```
raw -> staging -> intermediate -> marts
```

| Layer | Purpose | What goes here |
| --- | --- | --- |
| **raw** | Faithful copy of source | Source columns, source types, source casing. No transformations. |
| **staging** | One model per source table | Renamed columns, casted types, standardized values, surrogate keys. No joins between sources. |
| **intermediate** | Business logic | Joins, derived columns, deduplication, SCD handling. |
| **marts** | Analytics-ready | Wide tables for BI, narrow tables per metric, reporting cubes. |

## Process

1. **Land raw faithfully** — preserve original column names and types. Never edit raw.

2. **Build one staging model per source table**
   - One job, one table, one owner
   - Rename columns to project conventions (snake_case, no abbreviations)
   - Cast types
   - Apply `clean-data-types`, `clean-datetime-fields`, `standardize-values`
   - Define and document grain

3. **Build intermediate models for business logic**
   - Joins between staged sources
   - Sessionization, attribution, SCD handling
   - Pre-aggregations to a cleaner grain
   - Each intermediate has a documented grain and join history

4. **Build marts for consumption**
   - Wide fact tables suited to BI
   - Conformed dimensions
   - Stable column names — downstream depends on them

5. **At every layer, attach tests** (`add-data-quality-tests`)
   - Staging: type, not null, accepted values
   - Intermediate: grain uniqueness, referential integrity
   - Marts: business rules, freshness, volume

6. **Document at every layer**
   - One sentence: "what does this table represent?"
   - The grain
   - The owner
   - Known caveats

## Rules

- Never edit raw. Raw is your audit trail.
- Staging models are 1:1 with source tables; do not join sources in staging.
- Business logic belongs in intermediate, not in marts. Marts are for shape, not logic.
- Every model must have a grain documented at the top.
- Downstream tools read marts; everything else is internal.
