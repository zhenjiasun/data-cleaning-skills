---
name: audit-label-quality
description: Check whether ML labels are reliably defined, observed in time, and free of proxy and noise issues.
---

# Audit Label Quality

Use this skill before trusting any model evaluation. A model is only as good as its labels.

## Process

1. **Pin the label definition**
   - What real-world event does the label represent?
   - Example: "churn" — is it inactive 30 days, canceled subscription, no purchase 90 days, or NPS detractor?
   - Is the definition the same across train, validation, test, and production?

2. **Pin the observation window**
   - When does the label become observable after the prediction time?
   - Are recent rows artificially negative because the window has not closed yet? (Right-censoring)
   - Should those rows be excluded from training?

3. **Check label noise**
   - Inter-rater agreement for human-labeled data
   - Sample 50 rows and re-label by hand; compare
   - For automatic labels, check the label-generating logic for bugs

4. **Check for proxy labels**
   - Are you actually predicting the thing you care about, or a stand-in?
   - `clicked` ≠ `purchased`
   - `logged_in` ≠ `engaged`
   - `support_ticket_opened` ≠ `unhappy`

5. **Check class balance and base rate**
   - Severe imbalance → consider sampling, class weights, or different evaluation metrics
   - Verify the production base rate matches training; otherwise calibration breaks

6. **Check label leakage** (separate from feature leakage)
   - The label must be derivable only from post-outcome ground truth
   - Not from a column that updates in place

## Output Format

```
Label name: …
Definition (English): …
Observation window: …
Source of truth: …

Quality checks:
- noise rate (sample): …
- proxy risk: …
- right-censoring: …
- class balance: …
- production base rate vs training: …

Issues / recommendations:
- …
```

## Rules

- Two papers, two dashboards, two models with the same metric name and different definitions = three projects, not one.
- If the label takes T days to materialize, the most recent T days of rows have unreliable labels.
- Proxies are fine if you accept what you are actually optimizing — make it explicit.
- Label noise puts a ceiling on model performance. Improving labels often beats improving the model.
