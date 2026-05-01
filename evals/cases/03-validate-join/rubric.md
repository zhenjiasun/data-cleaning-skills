# Rubric — Case 03

## What's actually wrong

This is a classic SCD2 (slowly-changing dimension type 2) join. The customer table has multiple rows per customer with `valid_from / valid_to`. A naive `JOIN ON customer_id` will fan out — each order matches every address version of that customer.

For customer 101 (2 address versions) and 2 orders, naive join produces **4 rows**, doubling revenue.
For customer 103 (2 address versions) and 2 orders, naive join produces **4 rows**, doubling revenue.

Total naive sum = 50+50+75+75 + 120 + 30+30+60+60 + 90 = $640.
Correct sum = 50+75+120+30+60+90 = $425.

The fix is the SCD2 predicate: `JOIN ON c.customer_id = o.customer_id AND o.order_date >= c.valid_from AND o.order_date < c.valid_to` (or `<=` depending on convention).

There is also a subtle issue with order 9006 (customer 104, order_date 2023-10-12) — customer 104's first address is `valid_from 2023-09-01`, so this works. But if any order pre-dated the earliest address version, you'd have an orphan; worth checking.

## What "good" looks like

- Counts rows before and after the join
- Checks the cardinality of the dim table (it's not unique on `customer_id`!)
- Recognizes the SCD2 pattern from `valid_from / valid_to`
- Either uses the SCD2 predicate, or pre-aggregates orders to customer first then joins to "current" address
- Verifies post-join row count equals pre-join order count (6)
- Reconciles the revenue total

## Anti-patterns to penalize

- Naive `JOIN customers ON customer_id` without the validity predicate (silent doubling)
- Drops "duplicate" customer rows with `DISTINCT customer_id` (loses address history)
- Reports a total that's wrong but plausible-looking ($640 instead of $425)
- Doesn't check row counts at all

## Score key — same dimensions as Case 01.
