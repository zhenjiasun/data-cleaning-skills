# Rubric — Case 02

## What's actually wrong

Three timestamp columns with **different meanings** and the user calls them all "messy":

- `timestamp` — likely the bar time (event time). 9 of 10 are formatted strings, 1 is unix epoch.
- `event_time` — unix epoch seconds. UTC by definition. Decoding row 1: `1710062400` = `2024-03-10 13:30:00 UTC` = `09:30 ET`. Bar timestamps in `timestamp` column are **already ET**, not UTC.
- `loaded_at` — when ETL ingested the row.

Time zone trap: if you treat the `timestamp` column as UTC and "convert to ET" you'll subtract 4 hours and corrupt every row. If you `tz_localize("US/Eastern")` instead of `tz_convert`, you'll relabel without converting and end up wrong by 4 hours.

The 16:30 bar on 2024-03-10 is **after market close** (16:00 ET) — either a regular-hours error or extended-hours data.

## What "good" looks like

- Distinguishes `timestamp` / `event_time` / `loaded_at` semantically — they are not interchangeable
- Verifies the time zone of `timestamp` by cross-referencing the `event_time` epoch (UTC)
- Notes the mixed format in `timestamp` row 8 (`1710174600` is an epoch in a string column)
- Calls out the 16:30 bar (after market close)
- Recommends storing UTC in storage, converting to ET on display
- Flags the **localize vs convert** trap explicitly
- Does not silently overwrite or convert without confirming the source zone

## Anti-patterns to penalize

- Assumes `timestamp` is UTC and "fixes" it by converting to ET (4-hour shift in the wrong direction)
- Calls the three columns "duplicates" and drops two of them
- Silently uses `tz_localize` instead of `tz_convert`
- Doesn't notice the unix epoch row mixed in the string column

## Score key — same dimensions as Case 01.
