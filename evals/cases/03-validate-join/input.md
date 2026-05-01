# Two tables

## `customers` (one row per customer per address-on-file version)

| customer_id | address           | valid_from   | valid_to     |
| ----------- | ----------------- | ------------ | ------------ |
| 101         | 5 Main St         | 2022-01-01   | 2023-06-30   |
| 101         | 18 Oak Ave        | 2023-07-01   | 9999-12-31   |
| 102         | 12 Pine Rd        | 2022-03-15   | 9999-12-31   |
| 103         | 7 Elm St          | 2022-05-01   | 2023-12-31   |
| 103         | 22 Birch Ln       | 2024-01-01   | 9999-12-31   |
| 104         | 99 Cedar Ct       | 2023-09-01   | 9999-12-31   |

## `orders` (one row per order)

| order_id | customer_id | order_date  | amount  |
| -------- | ----------- | ----------- | ------- |
| 9001     | 101         | 2023-04-10  | 50.00   |
| 9002     | 101         | 2023-08-15  | 75.00   |
| 9003     | 102         | 2023-10-01  | 120.00  |
| 9004     | 103         | 2023-11-20  | 30.00   |
| 9005     | 103         | 2024-02-05  | 60.00   |
| 9006     | 104         | 2023-10-12  | 90.00   |

## What I want

Total revenue per customer, with their **address at the time of order**.
