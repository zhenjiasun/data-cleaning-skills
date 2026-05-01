# Rubric — Case 01

## What "good" looks like

A strong response should, before doing any cleaning:

- Identify the **duplicate** row (1001 and 1007 are identical)
- Flag **inconsistent country names** (`US`, `USA`, `U.S.`, `CN`, `unknown`) and propose a canonical vocabulary
- Detect **invalid dates**: `2023-13-01` (month 13), `2024-02-30` (Feb 30), `1900-01-01` (sentinel), `04/12/2023` (mixed format)
- Detect **sentinel values**: `age = 250`, `age = -1`, `lifetime_value = -1.00`, `lifetime_value = 99999999`, `lifetime_value = 9999.99`
- Detect **missing values**: blank `name` (1005), missing `age` (1012), `country = unknown` (1012)
- Note **email casing inconsistency** (`JOHN.DOE` vs lowercase elsewhere)
- Note **status field is constant** (only `active`/`closed`) — low cardinality, possible enum
- Refuse to silently drop or impute — ask first, or output a separate "questions for owner" list
- Suggest **automated tests** to keep clean

## Anti-patterns to penalize

- Just outputs a "cleaned" CSV with silent decisions (e.g., dropping the duplicate, picking one country format)
- Imputes missing values without asking
- Treats `9999.99`, `-1`, or `1900-01-01` as legitimate values
- Coerces dates without flagging the impossible ones (`2023-13-01`)

## Score key

| Dimension | 0 | 1 | 2 | 3 |
| --- | --- | --- | --- | --- |
| Diagnosis | misses most issues | catches obvious ones | catches most | catches all incl. sentinels & impossible dates |
| Structured output | free prose only | some sections | clear table per column | n/a |
| Risk flagging | none | mentions a few | flags most silent decisions | also raises questions for owner |
| Avoids destruction | dumps cleaned CSV | partially silent | doesn't modify without confirmation | n/a |
| Reproducibility | no convention docs | mentions a few | full conventions + suggested tests | n/a |
