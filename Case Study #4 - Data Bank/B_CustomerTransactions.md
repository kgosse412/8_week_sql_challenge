# Customer Transactions
## Table of Contents

[1. What is the unique count and total amount for each transaction type?](#1-what-is-the-unique-count-and-total-amount-for-each-transaction-type)

[2. What is the average total historical deposit counts and amounts for all customers?](#2-what-is-the-average-total-historical-deposit-counts-and-amounts-for-all-customers)

[3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?](#3-for-each-month---how-many-data-bank-customers-make-more-than-1-deposit-and-either-1-purchase-or-1-withdrawal-in-a-single-month)

[4. What is the closing balance for each customer at the end of the month?](#4-what-is-the-closing-balance-for-each-customer-at-the-end-of-the-month)

[5. What is the percentage of customers who increase their closing balance by more than 5%?](#5-what-is-the-percentage-of-customers-who-increase-their-closing-balance-by-more-than-5)

## Questions and Answers
### 1. What is the unique count and total amount for each transaction type?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_transactions | Contains information on the transaction types |

**SQL Statement:**
	
```sql	
SELECT
ct.txn_type AS "Transaction Type"
/* Assumes that each transaction is unique on it's own, meaning each customer_id, txn_date,
txn_type, and txn_amount is different from all other values in the table. */
,COUNT(*) AS "Unique Count"
/* Add up the transaction amounts. */
,SUM(ct.txn_amount) AS "Transaction Amount Total"

FROM data_bank.customer_transactions AS ct

GROUP BY "Transaction Type" -- Group by our transaction type

ORDER BY "Transaction Type" -- Sort by transaction type (to make the output prettier)
```

**Table Output:**

| Transaction Type | Unique Count | Transaction Amount Total |
| ---------------- | ------------ | ------------------------ |
| deposit          | 2671         | 1359168                  |
| purchase         | 1617         | 806537                   |
| withdrawal       | 1580         | 793003                   |

**Answer:**

- See table above.

### 2. What is the average total historical deposit counts and amounts for all customers?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_transactions | Contains deposit information |

**SQL Statement:**
	
```sql	
WITH deposit_metrics AS (SELECT
  ct.customer_id
  /* Assumes that each transaction is unique on it's own, meaning each customer_id, txn_date,
  txn_type, and txn_amount is different from all other values in the table. */
  ,COUNT(*) AS deposit_count
  /* Average the transaction amounts. */
  ,ROUND(AVG(ct.txn_amount), 2) AS avg_deposits

  FROM data_bank.customer_transactions AS ct
  
  /* Only look at deposit transaction types. */
  WHERE
  ct.txn_type = 'deposit'

  GROUP BY ct.customer_id -- Group by Customer
)

SELECT
/* Average the number of deposits and round to the nearest whole number. */
ROUND(AVG(dm.deposit_count)) AS "Average Deposit Count"
/* Average the average deposit amount and round to 2 decimal places. */
,ROUND(AVG(dm.avg_deposits), 2) AS "Average Deposit Amount"

FROM deposit_metrics AS dm
```

**Table Output:**

| Average Deposit Count | Average Deposit Amount |
| --------------------- | ---------------------- |
| 5                     | 508.61                 |

**Answer:**

- On average, 5 deposits are made per month with an average of $508.61 per deposit.

### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_transactions | Contains transaction type information |

**SQL Statement:**
	
```sql	
WITH txn_type_amts AS (SELECT
  ct.customer_id
  ,EXTRACT(MONTH FROM ct.txn_date::timestamp) AS month_num
  /* Add up all the times a customer makes a deposit. */
  ,SUM(
    CASE
      WHEN ct.txn_type = 'deposit' THEN 1
      ELSE 0
    END
  ) AS total_deposits
  /* Add up all the times a customer makes a purchase. */
  ,SUM(
    CASE
      WHEN ct.txn_type = 'purchase' THEN 1
      ELSE 0
    END
  ) AS total_purchases
  /* Add up all the times a customer makes a withdrawal. */
  ,SUM(
    CASE
      WHEN ct.txn_type = 'withdrawal' THEN 1
      ELSE 0
    END
  ) AS total_withdrawals
  
  FROM data_bank.customer_transactions AS ct

  GROUP BY 1, 2 -- Group the aggregrates by customer_id and txn_date
)

SELECT
amts.month_num AS "Month Num"
/* Count all the customers */
,COUNT(DISTINCT amts.customer_id) AS "Customer Count"

FROM txn_type_amts AS amts

/* Only look at data where deposits is greater than 1 and
withdrawal or purchases is equal to 1. */
WHERE
amts.total_deposits > 1 AND
(amts.total_purchases = 1 OR amts.total_withdrawals = 1)

GROUP BY "Month Num"

ORDER BY "Month Num"
```

**Table Output:**

| Month Num | Customer Count |
| --------- | -------------- |
| 1         | 115            |
| 2         | 108            |
| 3         | 113            |
| 4         | 50             |

**Answer:**

- See table above.

### 4. What is the closing balance for each customer at the end of the month?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_transactions | Contains all the transactions (txn) made by each customer and the month they were made. |

**SQL Statement:**
	
```sql	
WITH monthly_balance AS (SELECT
  ct.customer_id AS customer
  /* Get the month number from txn_date. This is for sorting purposes. */
  ,EXTRACT(MONTH FROM ct.txn_date) AS month_num
  /* Grab the date for the end of the month. DATE_TRUNC('MONTH', ct.txn_date) truncates
  the date to the beginning of the month, then we add 1 MONTH and subtract 1 DAY to get
  to the end of the month. */
  ,(DATE_TRUNC('MONTH', ct.txn_date) + INTERVAL '1 MONTH - 1 DAY') AS end_of_month
  /* Add up all the txn_amount when txn_type is deposit. This gets us the total amount
  in deposits for each month. */
  ,SUM(
    CASE
     WHEN ct.txn_type = 'deposit' THEN ct.txn_amount
     ELSE 0
   END
  ) AS deposits
  /* Add up all the negative txn_amount when txn_type is NOT a deposit. This gets us
  the total amount in purchases or withdrawals for each month. */
  ,SUM(
    CASE
      WHEN ct.txn_type <> 'deposit' THEN -ct.txn_amount
      ELSE 0
    END
  ) AS purchases_or_withdrawals
  /* Add up the deposits and the withdrawals / purchases to find what the monthly
  change is. This doesn't look at the balance at the end of each month, but rather
  what the net change is for the month. */
  ,SUM(
    CASE
      WHEN ct.txn_type = 'deposit' THEN ct.txn_amount
      ELSE -ct.txn_amount
    END
  ) AS monthly_change

  FROM data_bank.customer_transactions AS ct

  /* I'm limiting to just the first 5 customers. Otherwise the table would be huge! */
  WHERE
  ct.customer_id IN (1,2,3,4,5)

  GROUP BY customer, month_num, end_of_month

  ORDER BY customer ASC, month_num ASC
),
/* all_months creates a series of months to use since the query monthly_balance only looks
at the months a transaction was made. For example, customer_id 1 only has transactions for
January and March, so the months February and April are missing for them. We fill in those
gaps here. */
all_months AS (SELECT
  mb.customer
  /* Take the end of the minimum month (i.e. 2020-01-31), and add a series of months to it
  to get each month we need. GENERATE_SERIES creates a series from 0 - 3 to multiple by the 1
  MONTH interval to figure out what we need to add to the end of the minimum month. This works
  the following way:
   - January is found by taking the minimum month (i.e. 2020-01-31), and adding 0 months to it.
   - Then, February is found by taking the minimum month and adding 1 month.
   - After that, March is found by taking the minimum month and adding 2 months.
   - This continues until the end of GENERATE_SERIES (i.e. 3 Months).
  */
  ,MIN(mb.end_of_month) + (interval '1' MONTH * GENERATE_SERIES(0, 3)) AS monthly_series

  FROM monthly_balance AS mb

  GROUP BY mb.customer
),
end_of_month_balances AS (SELECT
  am.customer
  ,am.monthly_series
  /* If deposits is null, then replace it with a 0. This just shows us that no depoits
  were made for a specific month. */
  ,COALESCE(mb.deposits, 0) AS deposits
  /* If purchases_or_withdrawals is null, then replace it with a 0. This just shows us that no
  purchases or withdrawals were made for a specific month. */
  ,COALESCE(mb.purchases_or_withdrawals, 0) AS purchases_or_withdrawals
  /* If monthly_change is null, then replace it with a 0. This just shows us that there wasn't
  a positive or negative net change for a specific month. This will also come in handy when we
  add up the changes to see a balance month by month. */
  ,COALESCE(mb.monthly_change, 0) AS monthly_change

  /* Bring in all the info, including nulls, for each month provided by the series we generated
  in all_months. */
  FROM all_months AS am
  LEFT JOIN monthly_balance AS mb 
      ON am.customer = mb.customer
      AND am.monthly_series = mb.end_of_month
)

SELECT
emb.customer AS "Customer"
,emb.monthly_series AS "End of Month"
,emb.deposits AS "Total Deposits"
,emb.purchases_or_withdrawals AS "Total Purchases or Withdrawals"
/* This is where the magic happens. We want to add up the monthly changes (i.e. SUM(emb.monthly_change))
grouped by the customer and ordered by the monthly series, but we want our bounds to be all the previous rows up to the current row. This will take January's monthly_change and add it to February's monthly_change to get a balance for February. For March, it will take January's monthly_change, add it to Febrary's monthly_change, and then add it to March's monthly_change get a balance for March. So on and so on until the end of monthly_series. */
,SUM(emb.monthly_change) OVER(PARTITION BY emb.customer ORDER BY emb.monthly_series ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS "Balance"

FROM end_of_month_balances AS emb

ORDER BY "Customer" ASC, "End of Month" ASC
```

**Table Output:**

| Customer | End of Month           | Total Deposits | Total Purchases or Withdrawals | Balance |
| -------- | ---------------------- | -------------- | ------------------------------ | ------- |
| 1        | 2020-01-31 00:00:00+00 | 312            | 0                              | 312     |
| 1        | 2020-02-29 00:00:00+00 | 0              | 0                              | 312     |
| 1        | 2020-03-31 00:00:00+00 | 324            | -1276                          | -640    |
| 1        | 2020-04-30 00:00:00+00 | 0              | 0                              | -640    |
| 2        | 2020-01-31 00:00:00+00 | 549            | 0                              | 549     |
| 2        | 2020-02-29 00:00:00+00 | 0              | 0                              | 549     |
| 2        | 2020-03-31 00:00:00+00 | 61             | 0                              | 610     |
| 2        | 2020-04-30 00:00:00+00 | 0              | 0                              | 610     |
| 3        | 2020-01-31 00:00:00+00 | 144            | 0                              | 144     |
| 3        | 2020-02-29 00:00:00+00 | 0              | -965                           | -821    |
| 3        | 2020-03-31 00:00:00+00 | 0              | -401                           | -1222   |
| 3        | 2020-04-30 00:00:00+00 | 493            | 0                              | -729    |
| 4        | 2020-01-31 00:00:00+00 | 848            | 0                              | 848     |
| 4        | 2020-02-29 00:00:00+00 | 0              | 0                              | 848     |
| 4        | 2020-03-31 00:00:00+00 | 0              | -193                           | 655     |
| 4        | 2020-04-30 00:00:00+00 | 0              | 0                              | 655     |
| 5        | 2020-01-31 00:00:00+00 | 1780           | -826                           | 954     |
| 5        | 2020-02-29 00:00:00+00 | 0              | 0                              | 954     |
| 5        | 2020-03-31 00:00:00+00 | 1130           | -4007                          | -1923   |
| 5        | 2020-04-30 00:00:00+00 | 0              | -490                           | -2413   |

**Answer:**

- See table above.

### 5. What is the percentage of customers who increase their closing balance by more than 5%?
___________________________________________________________________________________________________________________________
> NOTE: It's not clear from the question if the closing balance is just the final month of April or if we're looking at each month v the previous month's balance, so I have to decided to look at each month v the previous month's balance. If at any point between January and April a customer has increased their balance by greater than 5%, then they count towards the percentage in question.

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_transactions | Contains all the transactions (txn) made by each customer and the month they were made. |

**SQL Statement:**
	
```sql	
WITH monthly_balance AS (SELECT
  ct.customer_id AS customer
  /* Get the month number from txn_date. This is for sorting purposes. */
  ,EXTRACT(MONTH FROM ct.txn_date) AS month_num
  /* Grab the date for the end of the month. DATE_TRUNC('MONTH', ct.txn_date) truncates
  the date to the beginning of the month, then we add 1 MONTH and subtract 1 DAY to get
  to the end of the month. */
  ,(DATE_TRUNC('MONTH', ct.txn_date) + INTERVAL '1 MONTH - 1 DAY') AS end_of_month
  /* Add up all the txn_amount when txn_type is deposit. This gets us the total amount
  in deposits for each month. */
  ,SUM(
    CASE
     WHEN ct.txn_type = 'deposit' THEN ct.txn_amount
     ELSE 0
   END
  ) AS deposits
  /* Add up all the negative txn_amount when txn_type is NOT a deposit. This gets us
  the total amount in purchases or withdrawals for each month. */
  ,SUM(
    CASE
      WHEN ct.txn_type <> 'deposit' THEN -ct.txn_amount
      ELSE 0
    END
  ) AS purchases_or_withdrawals
  /* Add up the deposits and the withdrawals / purchases to find what the monthly
  change is. This doesn't look at the balance at the end of each month, but rather
  what the net change is for the month. */
  ,SUM(
    CASE
      WHEN ct.txn_type = 'deposit' THEN ct.txn_amount
      ELSE -ct.txn_amount
    END
  ) AS monthly_change

  FROM data_bank.customer_transactions AS ct

  GROUP BY customer, month_num, end_of_month

  ORDER BY customer ASC, month_num ASC
),
/* all_months creates a series of months to use since the query monthly_balance only looks
at the months a transaction was made. For example, customer_id 1 only has transactions for
January and March, so the months February and April are missing for them. We fill in those
gaps here. */
all_months AS (SELECT
  mb.customer
  /* Take the end of the minimum month (i.e. 2020-01-31), and add a series of months to it
  to get each month we need. GENERATE_SERIES creates a series from 0 - 3 to multiple by the 1
  MONTH interval to figure out what we need to add to the end of the minimum month. This works
  the following way:
   - January is found by taking the minimum month (i.e. 2020-01-31), and adding 0 months to it.
   - Then, February is found by taking the minimum month and adding 1 month.
   - After that, March is found by taking the minimum month and adding 2 months.
   - This continues until the end of GENERATE_SERIES (i.e. 3 Months).
  */
  ,MIN(mb.end_of_month) + (interval '1' MONTH * GENERATE_SERIES(0, 3)) AS monthly_series

  FROM monthly_balance AS mb

  GROUP BY mb.customer
),
end_of_month_balances AS (SELECT
  am.customer
  ,am.monthly_series
  /* If deposits is null, then replace it with a 0. This just shows us that no depoits
  were made for a specific month. */
  ,COALESCE(mb.deposits, 0) AS deposits
  /* If purchases_or_withdrawals is null, then replace it with a 0. This just shows us that no
  purchases or withdrawals were made for a specific month. */
  ,COALESCE(mb.purchases_or_withdrawals, 0) AS purchases_or_withdrawals
  /* If monthly_change is null, then replace it with a 0. This just shows us that there wasn't
  a positive or negative net change for a specific month. This will also come in handy when we
  add up the changes to see a balance month by month. */
  ,COALESCE(mb.monthly_change, 0) AS monthly_change

  /* Bring in all the info, including nulls, for each month provided by the series we generated
  in all_months. */
  FROM all_months AS am
  LEFT JOIN monthly_balance AS mb 
      ON am.customer = mb.customer
      AND am.monthly_series = mb.end_of_month
),
current_balances AS (SELECT
  emb.customer
  ,emb.monthly_series AS end_of_month
  ,emb.deposits
  ,emb.purchases_or_withdrawals
  /* This is where the magic happens. We want to add up the monthly changes (i.e. SUM(emb.monthly_change))
  grouped by the customer and ordered by the monthly series, but we want our bounds to be all the previous rows up to the current row. This will take January's monthly_change and add it to February's monthly_change to get a balance for February. For March, it will take January's monthly_change, add it to Febrary's monthly_change, and then add it to March's monthly_change get a balance for March. So on and so on until the end of monthly_series. */
  ,SUM(emb.monthly_change) OVER(PARTITION BY emb.customer ORDER BY emb.monthly_series ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS balance

  FROM end_of_month_balances AS emb

  ORDER BY customer ASC, end_of_month ASC
),
prev_balances AS(SELECT
  cb.customer
  ,cb.end_of_month
  /* Get the previous month's balance using the LAG window function. Group by customer and
   order by end_of_month so we make sure we're getting the previous balance. */
  ,LAG(cb.balance) OVER(PARTITION BY cb.customer ORDER BY cb.end_of_month) AS prev_balance

  FROM current_balances AS cb
),
five_percent_check AS (SELECT
  cb.*
  ,pb.prev_balance
  /* Check if the current balance is greater than the previous balance when 5% is
  added to it. If it is, then set our flag to 1. Otherwise, set the flag to 0. */
  ,CASE
    WHEN cb.balance > (pb.prev_balance + (0.05 * abs(pb.prev_balance))) THEN 1
    ELSE 0
  END AS balance_more_than_5_percent_flag

  /* Join our current_balances CTE to our prev_balances CTE on both the customer and
  end_of_month columns. */
  FROM current_balances AS cb
  LEFT JOIN prev_balances AS pb ON cb.customer = pb.customer AND cb.end_of_month = pb.end_of_month
)

SELECT
/* To get the percentage, we want to take the count of customers that have the
balance_more_than_5_percent_flag set to 1 (meaning they have at least one closing balance
that has increased by more than 5%), divide by the total number of customers (gotten via
the subquery), and multiply that by 100. Then round to 2 decimal places. */
ROUND(100.0 * COUNT(DISTINCT fpc.customer)/
      /* This subquery returns the total unique number of customers. We use it so we can get
      the total number of customers without being impacted by the outter query's WHERE
      clause. */
      (SELECT
       COUNT(DISTINCT mb.customer)
       
       FROM monthly_balance AS mb
      ),
2) AS "Percentage of Customers"

FROM five_percent_check AS fpc

WHERE
fpc.balance_more_than_5_percent_flag = 1 -- Only look at the rows where the flag is set to 1
```

**Table Output:**

| Percentage of Customers |
| ----------------------- |
| 67.40                   |

**Answer:**

- 67.40% of customers have had at least one balance that has increased by more than 5%.