# Loan default training dataset

Schema (one row per loan application):

| column | description |
| --- | --- |
| `application_id` | unique application |
| `applicant_age` | age at application |
| `annual_income` | self-reported, at application |
| `credit_score` | bureau score, pulled at application |
| `requested_amount` | loan amount requested |
| `dti_ratio` | debt-to-income at application |
| `employment_years` | years at current employer at application |
| `application_date` | when application was submitted |
| `decision_date` | when underwriting decided (1–14 days after application) |
| `loan_status` | current status: `current`, `paid_off`, `late_30`, `late_60`, `default` |
| `total_payments_made` | sum of payments made to date |
| `latest_fico` | most recent FICO score (refreshed monthly) |
| `collections_calls` | count of collections calls made on this account |
| `is_default` | label: did the loan ever go to default (90+ DPD) within 12 months |

Sample rows:

```
A1001, 34, 65000, 720, 12000, 0.31, 5, 2023-02-10, 2023-02-15, paid_off,    12450, 730, 0, 0
A1002, 28, 42000, 640, 8000,  0.45, 1, 2023-03-04, 2023-03-12, default,     2100,  580, 9, 1
A1003, 51, 95000, 760, 25000, 0.22, 14,2023-04-19, 2023-04-22, current,    18900, 755, 0, 0
A1004, 22, 32000, 590, 5000,  0.55, 0, 2023-05-08, 2023-05-15, default,     900,   540, 14,1
A1005, 40, 78000, 700, 15000, 0.28, 8, 2023-06-12, 2023-06-19, current,    11300, 705, 1, 0
```
