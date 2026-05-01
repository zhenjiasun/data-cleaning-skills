---
name: validate-train-test-split
description: Confirm the train/test split mirrors deployment reality — temporal, grouped, stratified, and free of cross-set entity leakage.
---

# Validate Train/Test Split

Use this skill before reporting any model performance number. A wrong split makes good metrics meaningless.

## Process

1. **Pick split strategy by data type**
   - **Random** — only for IID, non-temporal, single-row-per-entity data
   - **Time-based** — required for any temporal prediction problem
   - **Group-aware** — required when one entity has multiple rows (split by entity, not row)
   - **Stratified** — preserve class balance, but never let stratification break temporal order

2. **Check for entity leakage**
   - Same `customer_id` / `device_id` / `user_id` in train and test? Almost always wrong.
   - Use `GroupKFold` / `GroupShuffleSplit` or a manual entity split.

3. **Check for temporal leakage**
   - Is every train timestamp before every test timestamp?
   - Did any feature use a window that crosses the split boundary?
   - Is the holdout the *most recent* data, not a random week from the middle?

4. **Check distribution consistency**
   - Class balance similar across splits?
   - Feature distributions similar (or, for time-based splits, plausibly different — drift is expected)
   - Source mix (region, platform) representative

5. **Match production reality**
   - The test set should look like the data the model will see in production.
   - If production receives data from a new region, evaluate on that region too.

6. **Backtest, not just split**
   - For time series, walk-forward / rolling-origin evaluation > a single train/test split.

## Output Format

```
Split strategy: …
Train rows: …  span: …
Val rows: …    span: …
Test rows: …   span: …

Entity leakage check: …
Temporal leakage check: …
Class balance: train=…, val=…, test=…

Issues:
- …
```

## Rules

- Random splits on time-series data are the most common silent leakage bug.
- Group-aware splits are not optional when entities span multiple rows.
- Holdout = future, not "a random slice".
- If test performance >> production performance, suspect the split before the model.
