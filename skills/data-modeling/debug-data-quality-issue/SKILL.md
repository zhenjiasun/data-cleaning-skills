---
name: debug-data-quality-issue
description: Root-cause an unexpected metric move, bad row, or failed quality test using a disciplined investigation loop.
---

# Debug Data Quality Issue

Use this skill when something is wrong and the cause is not yet known. Resist the urge to "just clean it up" — find the cause, then fix it at the source.

## Process

1. **Identify the symptom precisely**
   - Which metric / dashboard / table?
   - What value is wrong?
   - What value is expected?
   - When did it start?

2. **Reproduce**
   - Pull the offending rows
   - Re-run the query
   - Confirm the issue is stable, not flaky

3. **Locate the affected slice**
   - Which sources, which partitions, which time range?
   - Is it specific to a region, version, or pipeline run?

4. **Compare before/after the regression window**
   - Schema changes
   - Code changes (`git log` on the relevant models)
   - Pipeline run history
   - Source-system changes
   - Library / dependency upgrades

5. **Walk upstream** (`trace-data-lineage`)
   - Test each stage's output for the same symptom
   - Stop where the symptom first appears — that is the root cause

6. **Confirm root cause**
   - Hypothesis must explain *all* observed symptoms, not just the obvious one
   - Reproduce on a small sample if possible

7. **Fix at the root**
   - Producer fix > pipeline fix > consumer fix
   - Pick the highest-leverage layer

8. **Add a regression test** (`add-data-quality-tests`)
   - The same bug must never silently return.

9. **Backfill / restate** affected downstream data and notify consumers.

## Output Format

```
# Incident: <short title>

Symptom: …
First seen: …
Affected scope: …

Investigation log:
- step 1 …
- step 2 …
- root cause hypothesis: …
- evidence: …

Fix:
  layer: …
  change: …
  PR: …

Regression test: …
Backfill plan: …
Comms: …
```

## Rules

- Do not patch the symptom. Patch the cause.
- Every incident produces a test; otherwise the bug returns.
- Communicate restated metrics to consumers explicitly. Silently fixing history erodes trust.
- A "we don't know yet" is better than a wrong root cause; keep investigating.
