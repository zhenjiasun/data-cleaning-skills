---
name: infer-schema
description: Recommend the correct logical type for each column in a weakly-typed dataset and explain why.
---

# Infer Schema

Use this skill when raw data has weak typing — a CSV import, a third-party API extract, a warehouse landing zone where everything is `VARCHAR`.

## Process

1. **For each column, sample diverse values** (head, tail, random rows) and inspect.

2. **Check for type coercion traps**
   - Leading-zero strings: `"00123"` is a string, not an integer (zip codes, account IDs)
   - Currency/percent text: `"$1,200.50"`, `"12%"`
   - Dates as strings: `"2024-01-05"`, `"5/1/24"`, `"20240105"`
   - Booleans-as-text: `Y/N/Yes/No/1/0/true/false`
   - JSON or array embedded in a string column

3. **Recommend a logical type** with confidence:
   - `string` (free text, identifiers with leading zeros)
   - `integer` / `bigint`
   - `decimal(p,s)` for money — never `float`
   - `boolean`
   - `date`, `timestamp`, `timestamp with time zone`
   - `enum`/categorical with allowed values
   - `json`/`struct`

4. **Flag mixed-type columns** — these almost always indicate a parsing error or two sources merged together.

5. **Document the format** alongside the type: date format, currency, decimal separator, time zone.

## Output Format

```
| Column | Current | Recommended | Confidence | Reason | Sample values |
```

## Rules

- Money is `decimal`, not `float`. Never trust binary float for currency.
- IDs are strings unless they are guaranteed sequential and never zero-padded.
- A column with mixed types should be split or rejected, not coerced.
- Always pair date/timestamp recommendations with a time zone decision (`clean-datetime-fields`).
