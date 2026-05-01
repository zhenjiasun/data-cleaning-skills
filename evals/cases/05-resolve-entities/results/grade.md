# Grade — Case 05

## Scoring

| Dimension | Baseline | Skilled | Notes |
| --- | --- | --- | --- |
| Diagnosis depth | 2 | 3 | Baseline got the easy clusters right but auto-merged Acme despite domain mismatch. Skilled scored every pair and flagged the borderlines. |
| Structured output | 1 | 2 | Both produced tables; skilled added a `links` table design. |
| Risk flagging | 1 | 3 | Baseline got the Apple Bank trap right (lucky — name was different enough); silently merged Acme; ignored Alphabet/Google ambiguity. Skilled raised all three explicitly. |
| Avoids destruction | 0 | 2 | Baseline destroyed 8 rows. Skilled preserved all 16 and produced a links mapping. |
| Reproducibility | 1 | 2 | Baseline gave a recipe in prose. Skilled gave thresholds, scoring weights, and an unmerge-able mapping schema. |
| **Total** | **5 / 12** | **12 / 12** | |

## Qualitative notes

This case is the closest call of the five. The baseline's actual *clustering decisions* were mostly correct on the easy ones, and it caught the Apple Bank trap. Two failures stand out:

1. **Acme silently merged**: the names normalize to the same string but the domains differ (`acme.io` vs `acme-corp.io`). Auto-merging without flagging the domain mismatch is the kind of decision that quietly merges two unrelated SaaS startups.
2. **Destructive output shape**: 16 rows in, 8 rows out, originals gone. Once you've shipped that as the canonical export, undoing a wrong merge requires going back to the source.

The skilled response also raised **Alphabet vs Google** as a question rather than picking. That's a use-case decision (legal entity vs corporate group) the user has to make, not the model.

## Verdict

The skill earns its keep mostly by:
1. Forcing a confidence threshold (so the model can't auto-merge low-confidence pairs).
2. Insisting on a non-destructive `links` mapping instead of a row drop.

Both are habits, not facts — exactly what skill-format guidance is good for.
