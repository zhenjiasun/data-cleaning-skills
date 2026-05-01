---
name: clean-time-series
description: Fix gaps, duplicates, irregular frequency, and late-arriving records in sequential data.
---

# Clean Time Series

Use this skill before forecasting, sessionization, or any analysis that assumes regular cadence.

## Process

1. **Establish the expected frequency**
   - 1-minute, 5-minute, hourly, daily, business-day, weekly?
   - Are weekends/holidays valid (markets closed → no bar) or should they be filled?

2. **Detect gaps**
   - Generate the expected timeline and compare
   - Distinguish: source-side gap (no data was emitted) vs ingestion-side gap (data lost in pipeline)

3. **Detect duplicates**
   - Multiple rows for the same `(entity, timestamp)` — usually retries or two ingestion paths
   - Decide which row to keep (latest `loaded_at`, highest `version`, etc.)

4. **Detect irregularity**
   - Bars at unexpected offsets (`09:31:07` instead of `09:31:00`)
   - Daylight Saving Time transitions
   - Holiday calendars

5. **Decide repair strategy**
   - **Forward-fill** for "last known value" semantics (price levels, account balances)
   - **Zero-fill** for counts where absence means zero (sales, events)
   - **Interpolate** only where the underlying signal is smooth and continuous
   - **Leave gap** when the gap itself is information (market closed)

6. **Distinguish backfill vs late-arriving**
   - **Backfill** — historical correction; rerun downstream
   - **Late-arriving** — appears with a stale timestamp; downstream metrics may need recomputation

7. **Validate**
   - Expected vs actual row count per period
   - Distribution of timestamps mod frequency (should cluster on the grid)

## Output Format

```
Series: <entity, metric>
Expected frequency: …
Actual frequency: …
Gaps: <count, examples>
Duplicates: <count, examples>
Repair strategy: …
Late-arriving rows: <count>
```

## Rules

- Never blindly forward-fill across a known closure (markets, holidays). It manufactures data that did not exist.
- For finance, `tz_convert` to ET; do not `tz_localize`. Yfinance 5-min bars come back in UTC and must be converted.
- Late-arriving records require a recompute window, not silent insertion at `loaded_at`.
- Document the calendar (trading days, holidays) alongside the data — same series, different calendar = different cleaning.
