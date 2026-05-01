---
name: detect-data-drift
description: Detect whether feature distributions, missingness, or category mix have shifted over time and assess impact.
---

# Detect Data Drift

Use this skill when a production model's predictions look off, a metric moved without an obvious cause, or a dashboard distribution changed.

## Process

1. **Pick a baseline window** (training period, last quarter, prior month) and a **comparison window** (recent data).

2. **Per-feature drift checks**
   - Numeric: PSI, KS statistic, Wasserstein distance, mean/stddev/percentile shifts
   - Categorical: PSI, JS divergence, top-K category mix shift, new/disappeared categories
   - Missingness: change in null rate per column

3. **Population-level checks**
   - Volume drift (row count changes)
   - Schema drift (new/removed columns, type changes)
   - Source mix drift (share of traffic per source/region/platform)

4. **Diagnose the cause**
   - Real-world change (seasonality, promo, new market)
   - Upstream pipeline change (new source, schema change, new defaults)
   - Definition change (label or feature semantics moved)
   - Sampling change (different filter applied upstream)

5. **Assess impact**
   - For ML: retrain, re-threshold, or roll back
   - For dashboards: annotate, recompute, or fix upstream
   - For pipelines: alert producers, update contract

## Output Format

```
| Feature | Baseline mean | Recent mean | PSI | KS | Verdict | Likely cause | Action |
```

Plus a top-line summary: "X of Y features drifted; severity = …; suspected cause = …".

## Rules

- Drift ≠ bug. Real-world change is also drift; do not auto-rollback without diagnosis.
- A new category appearing is drift even if PSI is small — flag it explicitly.
- For ML, log the training-time distribution once so you can compare against it forever.
- Do not chase drift on every feature; prioritize features the model weights heavily.
