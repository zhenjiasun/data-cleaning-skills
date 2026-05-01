---
name: validate-join
description: Validate that a join is correctly grained, has the right cardinality, and is not silently exploding rows or losing matches.
---

# Validate Join

Use this skill any time you join two or more tables. Many "dirty data" problems are actually bad join problems.

## Process

1. **State the intended relationship**
   - one-to-one
   - one-to-many
   - many-to-one
   - many-to-many (red flag — usually wrong unless explicitly intended)

2. **Check keys before joining**
   - Are join keys ever null? Nulls do not equal nulls in SQL — explicit handling required.
   - Are keys unique on the side that is supposed to be unique?
   - Are data types consistent? (`INT` vs `STRING` keys silently coerce or fail.)
   - Are string keys normalized (case, whitespace)?

3. **Count rows before and after**
   - Row count of left table
   - Row count after join
   - Number of unmatched left rows
   - Number of duplicated rows (cardinality explosion)

4. **Quantify join quality**
   - Match rate = matched / total
   - Null rate in joined columns
   - Fanout = post-join rows / left rows (should be 1.0 for one-to-one and many-to-one)

5. **Diagnose problems**
   - Fanout > 1 unexpectedly → dim table has duplicates or missing SCD predicate
   - Match rate < 100% → key mismatch, late-arriving dim, or environment mismatch
   - Match rate > 100% (rows post-join > rows pre-join, even with LEFT JOIN) → fanout

6. **Recommend fix**
   - Deduplicate the dim before joining
   - Aggregate to the desired grain *before* joining
   - Use the correct composite key
   - Normalize keys
   - Add `valid_from <= fact_time < valid_to` for SCD2
   - Switch join type (`LEFT` vs `INNER`) deliberately, not by accident

## Output Format

```
Intended grain: …
Intended cardinality: …
Left rows: …
Right rows: …
Joined rows: …
Match rate: …
Fanout: …
Null rate after join: …
Problem: …
Recommended fix: …
```

## Rules

- Check row counts before and after every join. No exceptions.
- A many-to-many join that you did not intend is a bug; explicitly aggregate one side first.
- Inner-joining a fact to a dim without first measuring the orphan rate silently drops rows.
- Document the intended cardinality in the model so future readers can verify.
