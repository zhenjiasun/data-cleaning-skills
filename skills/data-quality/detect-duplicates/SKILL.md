---
name: detect-duplicates
description: Find exact and near-duplicate records in a dataset using full-row, primary-key, composite-key, and fuzzy strategies.
---

# Detect Duplicates

Use this skill when you suspect repeated records — duplicate customers, transactions, SKUs, or event logs from retried API calls.

## Process

1. **Define what "duplicate" means in this dataset**
   - Same row exactly?
   - Same business key, different surrogate IDs?
   - Same entity with slightly different spelling?
   - Same event recorded twice (retry, replay)?

2. **Check exact duplicates**
   - Full-row duplicates (`SELECT *, COUNT(*) GROUP BY all columns HAVING > 1`)
   - Primary-key duplicates (a sign the PK is wrong or unenforced)

3. **Check composite-key duplicates**
   - Examples:
     - `customer_id, transaction_date, amount`
     - `email, signup_date`
     - `order_id, line_item_id`
   - Pick keys based on how the source system actually identifies an entity, not what is convenient.

4. **Check near-duplicates (fuzzy)**
   - Normalize before comparing: lowercase, trim, strip punctuation
   - Compare names with Levenshtein / Jaro-Winkler
   - Compare emails after normalizing dots and `+aliases` for gmail
   - Compare addresses after USPS-style standardization

5. **Investigate cause before deduping**
   - Retry storm in upstream pipeline?
   - Two ingestion paths for the same source?
   - Slowly changing dimension being appended instead of merged?
   - Human entry error?

6. **Recommend resolution**
   - Drop exact duplicates
   - Pick a winner (most recent, most complete) for near-duplicates
   - Send low-confidence matches to a human review queue (`resolve-entities`)
   - Add a uniqueness test upstream so it does not recur

## Output Format

```
# Duplicate Audit: <dataset>

## Definition of duplicate
…

## Findings
| Strategy | Key(s) | Duplicate count | % of rows | Examples |
| Full-row | all columns | … | … | … |
| PK        | id          | … | … | … |
| Composite | (a, b, c)   | … | … | … |
| Fuzzy     | name        | … | … | … |

## Likely cause
…

## Recommended action
…
```

## Rules

- Never silently dedupe. Report first, act second.
- Fuzzy matching needs a confidence threshold and a sample for human review — no threshold, no merge.
- Preserve duplicates in the raw layer; only resolve in staging or downstream.
- Adding a uniqueness test upstream is more valuable than the one-time cleanup.
