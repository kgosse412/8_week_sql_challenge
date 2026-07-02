# Transaction Analysis
## Table of Contents

[1. How many unique transactions were there?](#1-how-many-unique-transactions-were-there)

[2. What is the average unique products purchased in each transaction?](#2-what-is-the-average-unique-products-purchased-in-each-transaction)

[3. What are the 25th, 50th, and 75th percentile values for the revenue per transaction?](#3-what-are-the-25th-50th-and-75th-percentile-values-for-the-revenue-per-transaction)

[4. What is the average discount value per transaction?](#4-what-is-the-average-discount-value-per-transaction)

[5. What is the percentage split of all transactions for members vs non-members?](#5-what-is-the-percentage-split-of-all-transactions-for-members-vs-non-members)

[6. What is the average revenue for member transactions and non-member transactions?](#6-what-is-the-average-revenue-for-member-transactions-and-non-member-transactions)

## Questions and Answers
### 1. How many unique transactions were there?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
SELECT
COUNT(DISTINCT sales.txn_id) AS txn_count

FROM balanced_tree.sales AS sales;
```

**Table Output:**
| txn_count |
| --------- |
| 2500      |

**Answer:**

There are 2500 unique transactions.

### 2. What is the average unique products purchased in each transaction?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
WITH txn_data AS (
  SELECT
  sales.txn_id
  ,COUNT(sales.prod_id) AS txn_count

  FROM balanced_tree.sales AS sales

  GROUP BY sales.txn_id
)

SELECT
ROUND(AVG(txn_data.txn_count)) AS avg_purchases_per_txn

FROM txn_data;
```

**Table Output:**
| avg_purchases_per_txn |
| --------------------- |
| 6                     |

**Answer:**

There are an average of 6 unique purchases per transaction.

### 3. What are the 25th, 50th, and 75th percentile values for the revenue per transaction?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
WITH txn_revenue AS (
  SELECT
  sales.txn_id
  ,ROUND(
    SUM(sales.price * (1 - sales.discount / 100) * sales.qty),
    2
  ) AS revenue

  FROM balanced_tree.sales AS sales
  
  GROUP BY sales.txn_id
)

SELECT
percentile_cont(.25) WITHIN GROUP (ORDER BY rev.revenue) AS percentile_25
,percentile_cont(.5) WITHIN GROUP (ORDER BY rev.revenue) AS median
,percentile_cont(.75) WITHIN GROUP (ORDER BY rev.revenue) AS percentile_75

FROM txn_revenue AS rev;
```

**Table Output:**
| percentile_25 | median | percentile_75 |
| ------------- | ------ | ------------- |
| 375.75        | 509.5  | 647           |

**Answer:**

See table for answer.

### 4. What is the average discount value per transaction?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
WITH txn_data AS (
  SELECT
  sales.txn_id
  ,ROUND(
    SUM(sales.price * (sales.discount::numeric / 100) * sales.qty),
    2
  ) AS total_discount

  FROM balanced_tree.sales AS sales
  
  GROUP BY sales.txn_id
)

SELECT
ROUND(AVG(txn_data.total_discount), 2) AS avg_discount_per_txn

FROM txn_data;
```

**Table Output:**
| avg_discount_per_txn |
| -------------------- |
| 62.49                |

**Answer:**
$62.49 is the average discount per transaction.

### 5. What is the percentage split of all transactions for members vs non-members?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
```

**Table Output:**

**Answer:**

### 6. What is the average revenue for member transactions and non-member transactions?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
```

**Table Output:**

**Answer:**
