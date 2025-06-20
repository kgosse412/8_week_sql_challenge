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

**SQL Statement:**
	
```sql	

```

**Table Output:**

**Answer:**

-

### 5. What is the percentage of customers who increase their closing balance by more than 5%?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

**SQL Statement:**
	
```sql	

```

**Table Output:**

**Answer:**

-