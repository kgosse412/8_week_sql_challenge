# Case Study #1 - Danny's Diner
## Overview
All information related to this case study can be found at [Case Study #1 - Danny's Diner](https://8weeksqlchallenge.com/case-study-1/).

All solutions used PostgreSQL v17.

## Questions and Answers
### 1. What is the total amount each customer spent at the restaurant?
______________________________________________________________________

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| sales | Contains the information of what each customer bought |
| menu  | Contains the price of each bought item |

Expected Results:
- Customer A spent $76
- Customer B spent $74
- Customer C spent $36

I solved this by:
1. Using a join to connect the two tables (I chose left join but inner join would work as well).
2. Using SUM to add up the prices of the bought items.
3. Grouping the above by customer_id.

**SQL Statement:**
	
```sql	
SELECT
s.customer_id AS "Customer"
,SUM(m.price) AS "Total Spent"

FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m ON m.product_id = s.product_id

GROUP BY s.customer_id

ORDER BY s.customer_id
```

**Table Output:**

| Customer | Total Spent |
| -------- | ----------- |
| A        | 76          |
| B        | 74          |
| C        | 36          |

**Answer:**

- Customer A spent $76
- Customer B spent $74
- Customer C spent $36