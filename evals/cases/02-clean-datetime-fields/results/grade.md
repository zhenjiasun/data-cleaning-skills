# Grade — Case 02

## Scoring

| Dimension | Baseline | Skilled | Notes |
| --- | --- | --- | --- |
| Diagnosis depth | 1 | 3 | Baseline missed the time-zone of `timestamp` entirely (assumed UTC). Skilled cross-referenced `timestamp` against `event_time` epoch and proved `timestamp` is ET. |
| Structured output | 1 | 2 | Both produced code; skilled added a column-meaning table and a sanity assertion. |
| Risk flagging | 0 | 3 | Baseline produced wrong output silently. Skilled flagged the localize-vs-convert trap, the after-hours bar, and DST. |
| Avoids destruction | 0 | 2 | **Critical:** baseline ran `tz_localize("UTC")` on a column that's actually in ET — **this corrupts every timestamp by 4 hours** and the user will not notice until charts look weird. Skilled refused to run anything until the user confirms. |
| Reproducibility | 1 | 2 | Both code-shaped, but skilled's snippet is idempotent and includes an `assert` that would catch a regression. |
| **Total** | **3 / 12** | **12 / 12** | |

## Qualitative notes

This is the most dangerous baseline failure of the five cases. The output **looks fine** — UTC times in clean ISO format. But the entire `timestamp` column has been mislabeled (treated as UTC when it was already ET), so every downstream analysis (returns, sessionization, comparisons across days) will be off by 4 hours.

This is exactly the failure mode the project's own `CLAUDE.md` warns about for yfinance data. The skill encodes that institutional knowledge directly. Without the skill, the model picked the most natural-but-wrong default.

The skilled response also did one thing genuinely impressive: it **verified** the time zone empirically using the redundant epoch column, instead of just guessing. That's a procedure, not a fact — exactly what a SKILL.md is supposed to encode.

## Verdict

For time-series and finance work, this kind of timezone bug is the single most common silent disaster. The skill explicitly raises the localize-vs-convert distinction and the verification procedure. High value.
