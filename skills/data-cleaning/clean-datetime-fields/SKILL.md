---
name: clean-datetime-fields
description: Resolve mixed date formats, time zone ambiguity, and event-vs-ingestion confusion in date/time columns.
---

# Clean Datetime Fields

Use this skill when timestamps look inconsistent, time zones are unclear, or you are about to do anything time-based (sessionization, time-series, train/test splits).

## Process

1. **Identify what each timestamp means**
   - `event_time` — when the thing happened in the world
   - `created_at` — when the row was first written
   - `updated_at` — when the row was last modified
   - `loaded_at` — when ETL inserted the row downstream
   - These are not interchangeable.

2. **Resolve format**
   - Detect mixed formats: `"2024-01-05"`, `"5/1/24"`, `"20240105"`, `"Jan 5, 2024"`
   - Watch for day/month confusion (`05/01/2024` is Jan 5 in US, May 1 in EU)
   - Unix epoch in seconds vs milliseconds — check magnitude

3. **Resolve time zone**
   - Is the column timezone-aware (`timestamptz`) or naive (`timestamp`)?
   - If naive, what zone was it captured in? Source-system local? UTC? User-local?
   - **Convention**: store everything in UTC; convert to a display zone at the edge.
   - **Pitfall** (real example): a yfinance 5-min bar comes back in UTC. Use `tz_convert("US/Eastern")`. Calling `tz_localize` instead silently re-stamps UTC times as ET, corrupting every timestamp by hours.

4. **Validate the date range**
   - Min/max look plausible?
   - Future dates where unexpected? (often a default like `9999-12-31`)
   - Year `1900` or `1970-01-01` epoch zero — almost always a sentinel
   - DST transition gaps and overlaps if data is in a DST zone

5. **Standardize**
   - Cast to `timestamptz` in UTC
   - Document the source zone
   - Add helper columns for downstream (`event_date_local`, `hour_of_day_local`) only if needed

## Output Format

```
| Column | Logical meaning | Source format | Source TZ | Stored as | Issues | Action |
```

## Rules

- **Convert, do not localize**, when the timestamp is already in a known zone but stored naive. Localizing relabels without converting.
- Store UTC, display in user/business zone. Mixing the two in storage causes subtle off-by-N-hours bugs.
- Do not use `float` for "minutes since X" if precision matters — use real timestamps.
- For ML, the timestamp used for splitting must be `event_time`, not `loaded_at`. Otherwise you leak the future.
