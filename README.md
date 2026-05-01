# Data Cleaning Skills

Skills for data engineers and data scientists working with messy, untrusted data.

Inspired by [mattpocock/skills](https://github.com/mattpocock/skills). Each skill is a small, task-specific operating procedure — a `SKILL.md` you can give to an agent (Claude, Cursor, etc.) or use as a checklist for yourself.

## Why

Most "dirty data" problems are not exotic. They are the same patterns showing up over and over: bad joins, wrong grain, silent leakage, sentinel values, ambiguous time zones. These skills capture repeatable procedures for diagnosing and fixing them — so you stop relearning the lesson.

## How to use

- Browse the catalog below and pick the skill that matches your situation.
- Read the matching `SKILL.md`. It tells you **when to use it**, the **process** to follow, the **output format** to produce, and the **rules** that keep you out of trouble.
- For agents: drop the `SKILL.md` into your agent's context (or load it as a [Claude Skill](https://docs.claude.com/en/docs/claude-code/skills)) and ask the agent to follow it.
- For humans: use it as a review checklist before merging a PR or shipping a model.

## Catalog

### Data Quality (8)
What is in the data, and is it trustworthy enough to use?

| Skill | When to use |
| --- | --- |
| [profile-dataset](skills/data-quality/profile-dataset/SKILL.md) | First contact with a new table, file, or query result |
| [detect-duplicates](skills/data-quality/detect-duplicates/SKILL.md) | Suspect repeated customers, transactions, or events |
| [detect-outliers](skills/data-quality/detect-outliers/SKILL.md) | Suspicious numeric values, sentinel codes, impossible dates |
| [validate-business-rules](skills/data-quality/validate-business-rules/SKILL.md) | Confirm data obeys domain logic (ship_date >= order_date, etc.) |
| [check-referential-integrity](skills/data-quality/check-referential-integrity/SKILL.md) | Orphan records, broken joins, late-arriving dimensions |
| [check-data-freshness](skills/data-quality/check-data-freshness/SKILL.md) | Dashboard looks stale, pipeline may have failed |
| [add-data-quality-tests](skills/data-quality/add-data-quality-tests/SKILL.md) | Wire automated checks into a pipeline (dbt / GE / Soda) |
| [detect-data-drift](skills/data-quality/detect-data-drift/SKILL.md) | Production model or dashboard behaving differently than expected |

### Data Cleaning (8)
Turning raw, messy fields into consistent, typed, usable values.

| Skill | When to use |
| --- | --- |
| [infer-schema](skills/data-cleaning/infer-schema/SKILL.md) | Weak typing, messy CSV, inconsistent warehouse columns |
| [clean-data-types](skills/data-cleaning/clean-data-types/SKILL.md) | Currency strings, percentages, booleans-as-text |
| [clean-datetime-fields](skills/data-cleaning/clean-datetime-fields/SKILL.md) | Mixed formats, time zone ambiguity, event vs ingestion time |
| [standardize-values](skills/data-cleaning/standardize-values/SKILL.md) | Normalize country names, gender, status codes, etc. |
| [diagnose-missing-values](skills/data-cleaning/diagnose-missing-values/SKILL.md) | Decide *why* values are missing before treating them |
| [treat-missing-values](skills/data-cleaning/treat-missing-values/SKILL.md) | Apply imputation / indicator / drop after diagnosis |
| [resolve-entities](skills/data-cleaning/resolve-entities/SKILL.md) | Same customer/company/product appears under multiple spellings |
| [clean-time-series](skills/data-cleaning/clean-time-series/SKILL.md) | Missing dates, irregular frequency, late-arriving records |

### Data Modeling (9)
Shaping data so analytics and joins do not silently lie to you.

| Skill | When to use |
| --- | --- |
| [detect-table-grain](skills/data-modeling/detect-table-grain/SKILL.md) | Before aggregating, joining, or modeling any table |
| [validate-join](skills/data-modeling/validate-join/SKILL.md) | Any join — especially many-to-many or composite key |
| [validate-aggregation](skills/data-modeling/validate-aggregation/SKILL.md) | Computing rates, ratios, averages, distinct counts |
| [define-metric](skills/data-modeling/define-metric/SKILL.md) | Stakeholders disagree on a number; metric needs documentation |
| [build-clean-data-model](skills/data-modeling/build-clean-data-model/SKILL.md) | Designing raw → staging → intermediate → mart layers |
| [trace-data-lineage](skills/data-modeling/trace-data-lineage/SKILL.md) | Metric changed, dashboard broke, want upstream impact |
| [write-data-contract](skills/data-modeling/write-data-contract/SKILL.md) | Producer team keeps changing schema without warning |
| [document-cleaning-decisions](skills/data-modeling/document-cleaning-decisions/SKILL.md) | Make every transformation auditable |
| [debug-data-quality-issue](skills/data-modeling/debug-data-quality-issue/SKILL.md) | Root-cause an unexpected metric move or bad rows |

### Machine Learning Data (5)
Ensuring training data does not lie to your model.

| Skill | When to use |
| --- | --- |
| [detect-data-leakage](skills/machine-learning-data/detect-data-leakage/SKILL.md) | Before training — features may use post-outcome info |
| [audit-label-quality](skills/machine-learning-data/audit-label-quality/SKILL.md) | Labels are delayed, noisy, or proxy-defined |
| [validate-train-test-split](skills/machine-learning-data/validate-train-test-split/SKILL.md) | Time-based, grouped, or stratified evaluation setup |
| [clean-features](skills/machine-learning-data/clean-features/SKILL.md) | Prepare numeric/categorical features for modeling |
| [encode-categorical-features](skills/machine-learning-data/encode-categorical-features/SKILL.md) | Choose between one-hot, target, hash, embedding |

## Does this actually help?

See [`evals/`](evals/) for a small head-to-head comparing each skill against a plain "clean this for me" prompt on the same dirty dataset. Single-shot results scored against a 12-point rubric:

| Case | Baseline | Skilled |
| --- | --- | --- |
| profile-dataset | 3/12 | 12/12 |
| clean-datetime-fields | 3/12 | 12/12 |
| validate-join | 2/12 | 12/12 |
| detect-data-leakage | 3/12 | 12/12 |
| resolve-entities | 5/12 | 12/12 |

[Read the summary](evals/summary.md) — the lift comes mostly from blocking five recurring failure patterns: silent fabrication, wrong-default time zones, silent join fanout, label leakage, and destructive deduplication. Caveats and methodology are documented; a runner script is included so you can re-run with multiple samples and other models.

## Recommended starter set

If you are bootstrapping this for a team, start with these twelve. They cover the highest-frequency dirty-data problems in real engineering and ML work:

1. `profile-dataset`
2. `infer-schema`
3. `diagnose-missing-values`
4. `clean-data-types`
5. `detect-duplicates`
6. `standardize-values`
7. `clean-datetime-fields`
8. `detect-table-grain`
9. `validate-join`
10. `validate-business-rules`
11. `detect-data-leakage`
12. `add-data-quality-tests`

## Contributing

Each skill is a single `SKILL.md` with YAML frontmatter (`name`, `description`) and four sections: **Process**, **Output Format**, **Rules**, and (optional) **Examples**. Open a PR with a new skill or improvements to an existing one.

## License

MIT
