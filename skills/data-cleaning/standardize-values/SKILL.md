---
name: standardize-values
description: Map messy categorical values (country names, status codes, gender, etc.) into a canonical vocabulary.
---

# Standardize Values

Use this skill when a categorical column has many spellings, casings, or near-synonyms for the same concept.

## Process

1. **Inventory variants**
   - Count distinct values
   - Group obvious clusters: `{NY, ny, N.Y., New York, new york}`
   - Compute string-similarity to find more clusters

2. **Pick a canonical vocabulary**
   - For known domains (countries, currencies, languages, US states), use a published standard:
     - Countries: ISO 3166-1 alpha-2 or alpha-3
     - Currencies: ISO 4217
     - Languages: BCP 47
     - US states: USPS 2-letter
   - For domain-specific values, define the canonical list explicitly.

3. **Build a mapping**
   ```
   raw_value → canonical_value (with confidence and rule)
   ```
   - High-confidence: exact normalization (`trim().lower()`)
   - Medium: alias table (`USA, U.S., United States → US`)
   - Low: fuzzy match — sample for human review before committing

4. **Apply and validate**
   - Coverage: % of rows mapped
   - Unmapped tail: list values you could not map
   - Drop nothing; route unmapped to `Other` or quarantine

5. **Maintain the mapping**
   - Store as a versioned seed/file, not as inline `CASE WHEN` sprawl
   - Re-run periodically; new variants will appear

## Output Format

```
| Raw value | Canonical | Mapping rule | Confidence | Rows |
```

Plus a separate "unmapped" table for values needing human review.

## Rules

- Canonical lists belong in version control, not in a one-off notebook.
- Never silently map low-confidence matches without review.
- Preserve the raw value alongside the canonical value (`country_raw`, `country`) — useful for debugging.
- Treat empty strings, `"unknown"`, and `NULL` as distinct unless you have proof they mean the same thing.
