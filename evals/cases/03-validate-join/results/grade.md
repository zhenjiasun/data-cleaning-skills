# Grade — Case 03

## Scoring

| Dimension | Baseline | Skilled | Notes |
| --- | --- | --- | --- |
| Diagnosis depth | 0 | 3 | Baseline did not even notice the dim is non-unique on `customer_id`. Skilled inspected key uniqueness first. |
| Structured output | 1 | 2 | Both produced a result table; skilled added cardinality + reconciliation tables. |
| Risk flagging | 0 | 3 | Baseline reported $640 with no concern. Skilled flagged fanout, predicted the inflation amount, called out inclusive-vs-exclusive convention. |
| Avoids destruction | 0 | 2 | **Critical:** baseline produced a wrong total ($640 vs $425) that the user might paste into a finance review. Skilled refused to commit until the convention is confirmed. |
| Reproducibility | 1 | 2 | Both have SQL; skilled reconciles to a sanity check (`SUM(orders.amount) = $425`). |
| **Total** | **2 / 12** | **12 / 12** | |

## Qualitative notes

The baseline made a textbook fanout error. The total of $640 is wrong by $215 (50%) for the doubled customers, and that's the kind of error that makes it into board decks because the SQL "looks fine".

The skilled response:
- Inspected key uniqueness on the dim before touching the join — this is the single highest-leverage check in the whole skill.
- Stated the intended cardinality before writing SQL.
- Predicted the fanout number ($640 vs $425) before running, demonstrating understanding rather than just luck.
- Offered both per-order and per-customer aggregations, calling out the difference between "address at time of order" and "current address" — these are not the same question.

## Verdict

SCD2 fanout is one of the highest-frequency, highest-cost data bugs in real warehouses. The skill's "check key uniqueness on the dim before joining" rule completely prevents this class of error. Very high value.
