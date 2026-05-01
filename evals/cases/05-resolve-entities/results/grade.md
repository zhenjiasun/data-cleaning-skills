# Grade — Case 05

## Scoring

| Dimension | Baseline | Skilled | Notes |
| --- | --- | --- | --- |
| Diagnosis depth | 2 | 3 | Both got the easy clusters and the Apple Bank trap. Baseline auto-merged Acme despite domain mismatch (it flagged the discrepancy but still merged). Skilled routed Acme to a review queue. |
| Structured output | 1 | 2 | Skilled produced an explicit links table with confidence scores; baseline gave a `merged_ids` column inside the deduped output. |
| Risk flagging | 2 | 3 | Baseline raised Apple Bank, Alphabet/Google, and Acme but acted on the merges anyway. Skilled raised the same three plus made the Acme handling non-destructive. |
| Avoids destruction | 1 | 2 | Baseline collapsed 16 → 8 in its primary output. The original IDs are listed in `merged_ids` so it is not fully destructive — but the canonical export lost the originals as separate rows. Skilled produced a `links` mapping so all 16 originals remain queryable. |
| Reproducibility | 1 | 2 | Skilled documented the methodology (normalization rules, scoring weights, blocking strategy, survivor rule). Baseline did not. |
| **Total** | **7 / 12** | **12 / 12** | |

## Qualitative notes

The largest real lift of the five cases. The baseline did the *clustering decisions* mostly correctly — it caught the Apple Bank trap, kept Alphabet/Google separate, asked the right questions. But it did the things the skill explicitly forbids:

1. **Auto-merged Acme** despite acknowledging the domain mismatch ("Verify the domain discrepancy" — but the merge was already in the output).
2. **Collapsed 16 → 8 in the primary output**, requiring the user to retrieve originals from a `merged_ids` string field rather than a proper mapping table.
3. **Did not specify confidence thresholds** — every merge happened with the same implicit policy.

The skilled response replaced "merge with a flag" with "route to review queue" for low-confidence pairs, and kept all originals in a `links` table. Both differences directly map to the SKILL.md "Rules" section.

## Verdict

The largest lift in the set (+5). Entity resolution is exactly the kind of task where habits matter more than knowledge, which is what skill-format guidance is designed for.
