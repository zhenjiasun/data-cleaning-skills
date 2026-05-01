# Grade — Case 01

## Scoring

| Dimension | Baseline | Skilled | Notes |
| --- | --- | --- | --- |
| Diagnosis depth | 3 | 3 | Both caught everything: duplicate, impossible dates (`2023-13-01`, `2024-02-30`), sentinels (`-1`, `9999.99`, `99999999`, `1900-01-01`, `unknown`), missing fields, casing. |
| Structured output | 2 | 2 | Both used issue tables. Tied. |
| Risk flagging | 2 | 3 | Baseline asked 6 follow-up questions but still produced a "Cleaned CSV" silently. Skilled refused to produce a cleaned CSV at all and listed 8 questions for the owner. |
| Avoids destruction | 0 | 2 | Baseline silently dropped row 1007 (the duplicate) and nullified ambiguous values in its output CSV before user could weigh in. Skilled changed nothing. |
| Reproducibility | 1 | 2 | Baseline did not propose automated tests; skilled did. |
| **Total** | **8 / 12** | **12 / 12** | |

## Qualitative notes

The real baseline was much more thorough than the illustrative one I'd guessed at — it caught all the impossible dates and sentinels. But it still made the central failure I designed the case to test: it produced a "cleaned" CSV that silently dropped rows and nullified values, ahead of the data owner's input.

The skilled version's discipline was the difference: profile and ask, do not modify. That is exactly what the SKILL.md "Rules" block enforces ("Do not modify data unless explicitly asked").

## Verdict

The skill's main lift here is **restraint**. Diagnosis quality was already high without it.
