---
name: write-data-contract
description: Define a producer/consumer agreement covering schema, semantics, freshness, ownership, and change process.
---

# Write Data Contract

Use this skill when an upstream team's changes keep breaking your pipelines, or when you are setting up a new producer-consumer relationship and want to prevent that.

## What a contract contains

```yaml
producer: <team / service>
consumer(s): <teams / pipelines>
asset: <table or topic>
owner: <person / on-call rotation>

schema:
  - name: customer_id
    type: string
    nullable: false
    description: Primary identifier; stable for the lifetime of the customer
  - name: signup_at
    type: timestamptz
    nullable: false
    description: UTC timestamp when account was created
  - name: country
    type: string
    nullable: true
    allowed_values: ISO 3166-1 alpha-2
    description: Country at signup; not updated later

semantics:
  grain: one row per customer
  primary_key: customer_id
  immutability: customer_id and signup_at never change

freshness:
  cadence: hourly
  sla: <= 30 minutes lag
  freshness_column: updated_at

quality:
  - unique(customer_id)
  - not_null(customer_id, signup_at)
  - accepted_values(country, ISO-3166-1-alpha-2)

change_process:
  - additive changes (new optional column): announce 7 days in advance
  - breaking changes (rename, type change, removal): require migration with consumer sign-off
  - emergency rollback: <runbook link>
```

## Process

1. **Talk to producer and consumers** before writing — a contract written in isolation will not be honored.
2. **Capture current behavior**, not aspirational behavior. Honest baselines first; tighten later.
3. **Decide what is *contract* vs *implementation*** — contract describes *what*, not *how*.
4. **Wire it as code**
   - Schema in dbt source / OpenLineage / a registry
   - Quality rules as automated tests
   - Freshness as a monitor
5. **Version it.** Treat the contract like a public API. Major version bump for breaking changes.

## Rules

- A contract not enforced by tests is a wish.
- The producer owns the contract; consumers can request changes but cannot unilaterally tighten.
- Breaking changes must include a migration window during which both old and new shapes work.
- If a producer cannot meet the contract, raise it before the deadline; do not silently degrade.
