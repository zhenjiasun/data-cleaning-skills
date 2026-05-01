# Grade — Case 03

## Scoring

| Dimension | Baseline | Skilled | Notes |
| --- | --- | --- | --- |
| Diagnosis depth | 3 | 3 | Both recognized SCD2 and used the validity-window predicate. Both produced the correct $425 total. **The baseline did not fall into the fanout trap.** |
| Structured output | 2 | 2 | Both clean. Skilled added an explicit join-validation block (grain, fanout, match rate). |
| Risk flagging | 2 | 2 | Baseline framed the per-customer ambiguity as "tension"; skilled framed it as "you must choose a grain". Roughly equivalent rigor. |
| Avoids destruction | 2 | 2 | Both offered Option A / Option B rather than picking. |
| Reproducibility | 1 | 2 | Skilled added a row-count validation block at the top — the kind of check that catches a future regression. Baseline did not. |
| **Total** | **10 / 12** | **11 / 12** | |

## Qualitative notes

This is the case I expected to show the largest lift. It did not. Real Claude Opus 4.7 used the SCD2 predicate without prompting, computed the correct revenue, and offered both grain options.

The honest result: the skill's "check key uniqueness on the dim" rule is **already internalized** by a strong frontier model on a clean, small example. The skill probably matters more on:
- larger, messier dim tables (where the SCD2 nature is not obvious)
- joins where naming conventions hide the validity columns
- joins through 3+ tables where fanout compounds

The +1 lift came entirely from one thing: skilled produced an explicit pre-join validation block (`Joined rows: 6 ✓ Fanout: 1.0 ✓`), which is the artifact a future maintainer can use to detect regressions.

## Verdict

Small lift. The skill's value here is preventative — it adds machinery (row-count checks) that's cheap to keep and useful when the data gets messier than this 12-row example.
