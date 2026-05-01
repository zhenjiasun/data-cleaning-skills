# Grade — Case 02

## Scoring

| Dimension | Baseline | Skilled | Notes |
| --- | --- | --- | --- |
| Diagnosis depth | 3 | 3 | Both caught the epoch leak in row 8, the Sunday-date issue, the DST trap, and the after-hours bar. Surprising — I expected the baseline to miss DST. |
| Structured output | 2 | 2 | Both used tables. |
| Risk flagging | 2 | 3 | Baseline flagged DST and weekend dates but did not call out the localize-vs-convert trap explicitly. Skilled made it the closing rule of the response. |
| Avoids destruction | 2 | 2 | Both produced advisory output rather than auto-converting. Baseline did not actually convert timestamps in code; skilled offered a defensive Python script that does explicit cross-validation. |
| Reproducibility | 1 | 2 | Baseline was diagnostic only. Skilled provided runnable Python with an `epoch_vs_string_delta` validation column and explicit weekend filtering. |
| **Total** | **10 / 12** | **12 / 12** | |

## Qualitative notes

I underestimated the baseline. Real Claude Opus 4.7 caught the DST bug on its own and traced it to "your upstream pipeline is probably applying a hardcoded `−5` offset" — a genuinely good diagnosis without the skill.

Where the skill earned its keep:
- Explicit **"use `tz_convert` not `tz_localize`"** callout — the single most common pandas time-zone bug, and the one the project's `CLAUDE.md` warns about.
- Defensive code with assertions and prints rather than just SQL.
- Saying out loud what to *trust* (the epoch) and what to *discard* (the string column).

## Verdict

Smaller lift than I'd guessed (+2). The baseline is genuinely strong; the skill adds discipline at the edge cases that matter most for production code.
