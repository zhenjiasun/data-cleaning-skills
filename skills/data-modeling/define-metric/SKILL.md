---
name: define-metric
description: Produce an unambiguous metric definition with formula, inclusion/exclusion rules, grain, owner, and known caveats.
---

# Define Metric

Use this skill when stakeholders disagree on a number, a metric is being added to a dashboard, or a definition is implicit and needs to be made explicit.

## Process

1. **Capture the business intent in one sentence**
   - "What decision does this metric inform?"

2. **Specify the formula precisely**
   - Numerator: what counts?
   - Denominator: what is the population?
   - Aggregation function and grain
   - Time window: trailing 7d? calendar month? since-account-start?

3. **Write inclusion and exclusion rules**
   - Internal users / test accounts → exclude
   - Refunded transactions → include or exclude?
   - Free trials → include or exclude?
   - Specific countries / regions filtered out?
   - Bots / non-human traffic?

4. **Pin the data source**
   - Which table(s)?
   - Which columns?
   - Which join keys?
   - At what grain?

5. **List known caveats**
   - Late-arriving data window
   - Time zone of the time bucket
   - Restatements / corrections behavior
   - Change history (when the definition changed last)

6. **Assign owner**
   - One person responsible for the definition
   - Where the canonical definition lives (semantic layer, dbt metric, doc)

## Output Format

```
# Metric: <name>

**Decision it informs:** …
**Formula:**
  numerator = …
  denominator = …
  aggregation = …
  grain = …
  window = …

**Includes:** …
**Excludes:** …
**Source:** <table.column>, …
**Owner:** …
**Last definition change:** YYYY-MM-DD — what changed and why
**Known caveats:** …
```

## Rules

- Two dashboards showing the same metric with different numbers means one (or both) lacks a written definition. Fix the docs first.
- Definitions live in a semantic layer / dbt metrics / single doc — not duplicated across SQL queries.
- Every change to a metric definition must be logged with date, reason, and authors. Restated history must be communicated.
- A metric without an owner will rot.
