---
name: document-cleaning-decisions
description: Make every cleaning rule auditable — what changed, why, how many rows, with samples and an owner.
---

# Document Cleaning Decisions

Use this skill so that 6 months later, someone (including future-you) can answer "why does this column look like that?".

## For every cleaning rule, capture

```
Rule ID: <short stable id>
Title: <one line>
Layer: raw | staging | intermediate | mart
Table.column: …
Rule (English): …
Rule (code / SQL): …
Rows affected: <count + %>
Before / after sample: 5 examples
Owner: …
Date introduced: YYYY-MM-DD
Reason: …
Risk if removed: …
Linked tests: …
```

## Process

1. **One rule = one record.** Don't bundle.
2. **Write the English first**, code second. The English explains why.
3. **Always include before/after samples** so reviewers can sanity-check.
4. **Track impact** — % of rows changed and known downstream effects.
5. **Store in version control**, alongside the code that implements the rule. A README, a table in dbt docs, or a YAML file — but in repo, not Notion.
6. **Link to a test** — the rule should be enforceable, not just documented.

## Output Format

A `cleaning-rules.md` (or per-model `_cleaning_rules.yml`) per project, listing all rules with the fields above.

## Rules

- Every cleaning step in production must have a rule record.
- "Trim whitespace" still gets a record, even if trivial — because someone will eventually want to know why.
- Rules without owners get deleted at the next refactor; owners survive.
- A change to a rule is a new record (versioned), not an in-place edit.
