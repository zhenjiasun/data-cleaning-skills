---
name: clean-data-types
description: Convert dirty text fields into clean numeric, date, boolean, and categorical values with documented conventions.
---

# Clean Data Types

Use this skill after `infer-schema` recommended a target type. This skill performs the actual conversion.

## Process

1. **Numeric strings**
   - Strip currency symbols: `"$1,234.50"` → `1234.50`
   - Strip thousands separators (locale-aware: `1.234,50` in EU)
   - Convert percent strings to a chosen convention: `"12%"` → `0.12` *or* `12` — pick one and document
   - Use `decimal`/`numeric`, never `float`, for money

2. **Boolean strings**
   - Map `{TRUE, true, T, Y, Yes, 1}` → `true`
   - Map `{FALSE, false, F, N, No, 0}` → `false`
   - Anything else → `null` and log

3. **Dates / timestamps** → see `clean-datetime-fields` for the full procedure.

4. **Categorical strings**
   - Trim whitespace
   - Normalize casing
   - Standardize variants (see `standardize-values`)

5. **Validate after conversion**
   - Conversion failure rate per column
   - Value range still plausible
   - No silent data loss (count rows before/after)

6. **Make the conversion idempotent and tested**
   - Same input, same output, every time
   - Add a unit test for each tricky case
   - Document units, conventions, and locale assumptions

## Output Format

For each column:

```
Column: <name>
Source format: <example>
Target type: <type>
Convention: <unit / locale / scale / time zone>
Conversion failures: <count, %, examples>
```

## Rules

- Document units: is `weight_kg` actually kg, or pounds mislabeled? Is `revenue` gross or net?
- Document the percent convention (`0.12` vs `12`) and stick with it project-wide.
- Failed conversions should land in a quarantine table or `_dq_failed` column, not be silently dropped.
- Never coerce a string ID to integer if leading zeros are meaningful.
