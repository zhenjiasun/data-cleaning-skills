# Rubric — Case 04

## What's actually wrong

The user is predicting default **at the time of application**. Several columns are post-outcome:

| Column | Why it leaks |
| --- | --- |
| `decision_date` | known after underwriting, not at application |
| `loan_status` | the outcome itself (and trivially correlates with `is_default`) |
| `total_payments_made` | accrues over the loan's life — strongly leaks the label |
| `latest_fico` | refreshed monthly — recent value is post-outcome |
| `collections_calls` | only nonzero for delinquent accounts — proxy for the label |

`is_default` is the label. Safe features: `applicant_age`, `annual_income`, `credit_score`, `requested_amount`, `dti_ratio`, `employment_years`, and date-derived features that respect application time.

Also worth noting:

- Label window: "default within 12 months" — recent applications may be right-censored
- Class imbalance is likely; needs handling
- Need a time-based split (not random) since loans accrue over time
- Need group-aware split if any applicant has multiple loans

## What "good" looks like

- Defines the **prediction moment**: time of application
- Lists every feature and classifies as `safe` / `leak` / `borderline`
- Catches `total_payments_made`, `latest_fico`, `collections_calls`, `decision_date`, `loan_status` as leakage
- Recommends time-based train/test split
- Flags right-censoring of the most recent applications
- Does not just dump a "cleaned" feature set without leakage analysis

## Anti-patterns to penalize

- Includes `total_payments_made`, `latest_fico`, or `collections_calls` in the feature set
- Says "remove rows with NULL `loan_status`" without noticing it's the outcome
- Uses random train/test split
- Reports "this feature is highly predictive" about a leaking feature

## Score key — same dimensions as Case 01.
