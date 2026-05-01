---
name: encode-categorical-features
description: Choose and apply a categorical encoding strategy (one-hot, target, frequency, hash, embedding) safely.
---

# Encode Categorical Features

Use this skill when preparing categorical features for a model. The wrong encoding is a common source of silent leakage.

## Choosing an encoding

| Strategy | Good for | Pitfalls |
| --- | --- | --- |
| **One-hot** | Low cardinality (< ~20), tree or linear models | Explodes dimensionality at high cardinality |
| **Ordinal / label** | Genuinely ordinal categories | Imposes false ordering on nominal categories |
| **Frequency** | Tree models, when frequency carries signal | Loses the identity of the category |
| **Target / mean** | Tabular GBM with high-cardinality features | High leakage risk if not done out-of-fold |
| **Hash (feature hashing)** | Very high cardinality, online systems, memory-bound | Collisions; hard to interpret |
| **Embedding** | Deep models, very high cardinality with semantic structure | Needs enough data; needs validation |

## Process

1. **Bucket rare categories first** (`Other`, threshold by min count or top-N)
2. **Pick encoding** per the table above
3. **Fit on train only.** This is the most common mistake.
   - For target encoding: use *out-of-fold* targets to compute means; otherwise the encoding is leakage
   - For embeddings: train them within the model, not on the full dataset
4. **Persist the encoder** (categories, mapping table, hashing seed, target stats) as part of the model artifact
5. **Handle unseen categories** at inference
   - Map to `Other`, or to a learned `<UNK>` embedding
   - Never throw or silently drop the row in production
6. **Validate**
   - Output dimensionality
   - Coverage (% of categories seen at training)
   - Effect on validation metric vs simpler encoding

## Output Format

```
Feature: …
Cardinality (train): …
Encoding: …
Rare-bucket threshold: …
Out-of-fold strategy (if target encoding): …
Unseen-category handling: …
Persisted artifact: …
```

## Rules

- Target encoding without out-of-fold computation **leaks the label**. Treat as a leakage bug.
- One-hot at very high cardinality bloats the model. Bucket or switch encoding.
- The encoding artifact (vocabulary / target stats / hash params) ships with the model. Train-serve skew here is brutal.
- Unseen categories happen in production. Decide your fallback up front.
