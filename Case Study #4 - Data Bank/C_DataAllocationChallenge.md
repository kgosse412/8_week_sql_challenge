# Data Allocation Challenge
## Table of Contents

[Running Customer Balance Column that Includes the Impact of Each Transaction](#Running-Customer-Balance-Column-that-Includes-the-Impact-of-Each-Transaction)

[Customer Balance at the End of Each Month](#Customer-Balance-at-the-End-of-Each-Month)

[Minimum, Average, and Maximum Values of the Running Balance for Each Customer](#Minimum-Average-and-Maximum-Values-of-the-Running-Balance-for-Each-Customer)

## Questions and Answers
### Using all of the data available - how much data would have been required for each option on a monthly basis?
___________________________________________________________________________________________________________________________

**Full Question**

To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

- Option 1: data is allocated based off the amount of money at the end of the previous month
- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
- Option 3: data is updated real-time

For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

- running customer balance column that includes the impact each transaction
- customer balance at the end of each month
- minimum, average and maximum values of the running balance for each customer

#### Running Customer Balance Column that Includes the Impact of Each Transaction

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_transactions | Contains all the transactions (txn) made by each customer |

**SQL Statement:**
	
```sql	
WITH txn_amts AS (SELECT
  ct.customer_id AS customer
  ,ct.txn_date
  ,ct.txn_type
  /* When a transaction type is a deposit, we want to show it as a positive amount. Otherwise,
  we want to show a negative amount. */
  ,CASE
      WHEN ct.txn_type = 'deposit' THEN ct.txn_amount
      ELSE -ct.txn_amount
  END AS txn_amount

  FROM data_bank.customer_transactions AS ct

  /* Limit to the first 3 customers just for the sake of seeing if our logic works. */
  WHERE
  ct.customer_id IN (1, 2, 3)

  ORDER BY ct.customer_id ASC, ct.txn_date ASC
)

SELECT
ta.customer AS Customer
,ta.txn_date AS "Transaction Date"
,ta.txn_type AS "Transaction Type"
,ta.txn_amount AS "Amount"
/* Use the sum function as a window function to run a sum grouped by the customer and
ordered by the txn_date. This gives us our balance after each transaction type. */
,SUM(ta.txn_amount) OVER(PARTITION BY ta.customer ORDER BY ta.txn_date) AS Balance

FROM txn_amts AS ta
```

**Table Output:**

| customer | Transaction Date | Transaction Type | Amount | balance |
| -------- | ---------------- | ---------------- | ------ | ------- |
| 1        | 2020-01-02       | deposit          | 312    | 312     |
| 1        | 2020-03-05       | purchase         | -612   | -300    |
| 1        | 2020-03-17       | deposit          | 324    | 24      |
| 1        | 2020-03-19       | purchase         | -664   | -640    |
| 2        | 2020-01-03       | deposit          | 549    | 549     |
| 2        | 2020-03-24       | deposit          | 61     | 610     |
| 3        | 2020-01-27       | deposit          | 144    | 144     |
| 3        | 2020-02-22       | purchase         | -965   | -821    |
| 3        | 2020-03-05       | withdrawal       | -213   | -1034   |
| 3        | 2020-03-19       | withdrawal       | -188   | -1222   |
| 3        | 2020-04-12       | deposit          | 493    | -729    |

**Answer:**

- See table above.

#### Customer Balance at the End of Each Month

> Note: This is the same question and logic as Q4 from Customer Transactions.

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

#### Minimum, Average, and Maximum Values of the Running Balance for Each Customer

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_transactions | Contains all the transactions (txn) made by each customer |

**SQL Statement:**
	
```sql	
WITH txn_amts AS (SELECT
  ct.customer_id AS customer
  ,ct.txn_date
  ,ct.txn_type
  /* When a transaction type is a deposit, we want to show it as a positive amount. Otherwise,
  we want to show a negative amount. */
  ,CASE
      WHEN ct.txn_type = 'deposit' THEN ct.txn_amount
      ELSE -ct.txn_amount
  END AS txn_amount

  FROM data_bank.customer_transactions AS ct

  /* Limit to the first 10 customers just for the sake of seeing if our logic works. */
  WHERE
  ct.customer_id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

  ORDER BY ct.customer_id ASC, ct.txn_date ASC
),
running_balance AS (SELECT
  ta.customer AS Customer
  ,ta.txn_date AS "Transaction Date"
  ,ta.txn_type AS "Transaction Type"
  ,ta.txn_amount AS "Amount"
  /* Use the sum function as a window function to run a sum grouped by the customer and
  ordered by the txn_date. This gives us our balance after each transaction type. */
  ,SUM(ta.txn_amount) OVER(PARTITION BY ta.customer ORDER BY ta.txn_date) AS Balance

  FROM txn_amts AS ta
)

SELECT
rb.customer AS Customer
,MIN(rb.balance) AS "Balance Minimum"
,ROUND(AVG(rb.balance), 2) AS "Balance Average"
,MAX(rb.balance) AS "Balance Maximum"

FROM running_balance AS rb

GROUP BY rb.customer
```

**Table Output:**

| customer | Balance Minimum | Balance Average | Balance Maximum |
| -------- | --------------- | --------------- | --------------- |
| 1        | -640            | -151.00         | 312             |
| 2        | 549             | 579.50          | 610             |
| 3        | -1222           | -732.40         | 144             |
| 4        | 458             | 653.67          | 848             |
| 5        | -2413           | -135.45         | 1780            |
| 6        | -552            | 624.00          | 2197            |
| 7        | 887             | 2268.69         | 3539            |
| 8        | -1029           | 173.70          | 1363            |
| 9        | -91             | 1021.70         | 2030            |
| 10       | -5090           | -2229.83        | 556             |

**Answer:**

- See table above.