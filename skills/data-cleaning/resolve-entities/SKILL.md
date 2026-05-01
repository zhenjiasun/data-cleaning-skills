---
name: resolve-entities
description: Merge records that refer to the same real-world entity (customer, company, product) using normalization, blocking, and similarity matching.
---

# Resolve Entities

Use this skill when the same real-world thing appears under multiple representations:

```
"Apple Inc." | "Apple" | "APPLE COMPUTER INC"
"Sunny Sun" | "Zhen Jia Sun" | "Z. Sun"
```

## Process

1. **Define the entity and its identity attributes**
   - Customer: name + email + phone + address
   - Company: legal name + domain + tax ID
   - Product: SKU + manufacturer + model number

2. **Normalize fields**
   - Lowercase, trim, strip punctuation
   - Strip corporate suffixes (`Inc.`, `LLC`, `Ltd`, `GmbH`)
   - Standardize addresses (USPS / libpostal)
   - Normalize emails (gmail dot/`+alias` collapsing)
   - Normalize phones to E.164

3. **Apply blocking** to avoid O(N²)
   - Block by: first character of normalized name, soundex, postal code, email domain
   - Compare only within blocks

4. **Score similarity**
   - String: Levenshtein, Jaro-Winkler, token-based (Jaccard, cosine)
   - Embedding similarity for semantic matches
   - Combine multiple field scores with weights or a learned classifier

5. **Decide and merge**
   - High confidence (e.g. >0.95) → auto-merge
   - Medium → human review queue
   - Low → leave separate
   - Always preserve raw records in a `links` table; do not destroy the originals

6. **Pick a survivor record**
   - Most recent
   - Most complete
   - Highest-trust source
   - Document the rule

## Output Format

```
| Cluster ID | Member IDs | Score | Decision | Survivor | Reviewer |
```

Plus a `links(raw_id, canonical_id, score, decided_at)` table.

## Rules

- Never destroy raw records. Resolution lives in a separate mapping layer.
- A merge needs an audit trail; an unmerge must be possible.
- Low-confidence matches must be reviewed by a human, not auto-merged.
- Re-run resolution as new records arrive; today's stranger is tomorrow's duplicate.
