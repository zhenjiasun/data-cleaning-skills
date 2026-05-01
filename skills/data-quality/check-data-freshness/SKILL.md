---
name: check-data-freshness
description: Determine when a table was last updated, whether the delay is normal, and where freshness broke if it is not.
---

# Check Data Freshness

Use this skill when a dashboard looks stale, a metric froze, or you suspect an upstream pipeline has failed silently.

## Process

1. **Establish the freshness contract**
   - Expected update cadence (hourly, daily 06:00 UTC, etc.)
   - Acceptable lag SLA
   - The "freshness column" — usually `updated_at`, `loaded_at`, or `event_time`. They are not interchangeable.

2. **Measure actual freshness**
   ```sql
   SELECT
     MAX(updated_at) AS latest,
     CURRENT_TIMESTAMP - MAX(updated_at) AS lag,
     COUNT(*) FILTER (WHERE updated_at > CURRENT_TIMESTAMP - INTERVAL '24 hour') AS rows_last_24h
   FROM table_name;
   ```

3. **Compare lag to SLA** and to historical lag distribution. A lag of "4 hours" is fine for some tables and a P0 for others.

4. **If late, walk upstream**
   - Source system delivery time
   - Ingestion job logs
   - Transformation job logs
   - Orchestrator status (Airflow, Dagster, dbt Cloud)

5. **Distinguish failure modes**
   - Pipeline failed → fix and rerun
   - Pipeline succeeded but no new source data → upstream issue
   - Pipeline ran on partial data → backfill required
   - Freshness column itself is wrong → use a different column

## Output Format

```
Table: …
Expected cadence: …
SLA: …
Latest row: … (lag = …)
Status: ON TIME | LATE | STALLED
Suspected cause: …
Recommended action: …
```

## Rules

- Do not confuse `event_time` (when the thing happened), `created_at` (when row was inserted), and `loaded_at` (when ETL ran). Pick one explicitly.
- A 0-row table can be "fresh" by some definitions and dead by others — check row counts alongside max timestamps.
- If `MAX(updated_at)` jumps to the future, you have a clock skew or timezone problem, not freshness.
