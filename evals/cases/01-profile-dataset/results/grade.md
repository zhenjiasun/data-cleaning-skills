# Grade — Case 01

## Scoring

| Dimension | Baseline | Skilled | Notes |
| --- | --- | --- | --- |
| Diagnosis depth | 1 | 3 | Baseline missed: `2023-13-01` and `2024-02-30` (impossible dates), `1900-01-01` sentinel, `9999.99` and `99999999` sentinels, `250` impossible, mixed date formats. Skilled caught all. |
| Structured output | 0 | 2 | Baseline = prose + CSV dump. Skilled = profile report with column-level table. |
| Risk flagging | 1 | 3 | Baseline silently picked `2023-01-13` for `2023-13-01` (assumed day/month confusion — could be wrong). Skilled flagged it as impossible and asked. Skilled also raised PK uniqueness question and sentinel ambiguity. |
| Avoids destruction | 0 | 2 | **Critical:** baseline silently dropped row 1007 (the duplicate), silently filled `name` as "Unknown", silently coerced `2023-13-01` → `2023-01-13` (day/month swap, **possibly wrong**), silently coerced `2024-02-30` → `2024-03-01` (**fabricated data**), silently kept `9999.99` and `99999999` as legitimate values. Skilled changed nothing. |
| Reproducibility | 1 | 2 | Baseline mentioned conventions in prose. Skilled produced a list of automated tests. |
| **Total** | **3 / 12** | **12 / 12** | |

## Qualitative notes

The baseline response is the worst kind of data cleaning: confident-looking output with several **silent fabrications**.

- `2023-13-01` → `2023-01-13`: this assumes day/month was swapped, but the source could equally have been a typo for `2023-12-01` or a corrupt write. Without asking, this is a guess masquerading as a fix.
- `2024-02-30` → `2024-03-01`: pure fabrication. Feb 30 is not a real date; nothing in the row tells you what the real date was.
- Keeping `lifetime_value = 99999999` as a real value will poison every aggregate downstream.
- Dropping the duplicate without flagging it removes evidence of an upstream bug.

The skilled response is also longer (~3×), but it is *honest* — every claim is grounded, every decision is deferred to the owner. The "Questions for the data owner" section is the hardest part to elicit from a model without a skill.

## Verdict

The skill flips the model from "eager intern who breaks data" to "cautious analyst who diagnoses first". This is the highest-leverage difference in the whole repo, because silent cleaning is the dominant failure mode in real LLM-assisted data work.
