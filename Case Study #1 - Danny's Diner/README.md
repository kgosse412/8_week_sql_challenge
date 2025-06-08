# Case Study #1 - Danny's Diner
## Overview
All information related to this case study can be found at [Case Study #1 - Danny's Diner](https://8weeksqlchallenge.com/case-study-1/).

## Questions and Answers
### 1. What is the total amount each customer spent at the restaurant?
______________________________________________________________________

**Overview:**

Two tables will be needed to solve this - sales and menu. Sales will have the information of what each customer bought and menu will have the price of each bought item.

I solved this by:
1. Using a join to connect the two tables
2. Using SUM to add up the prices of the bought items
3. Grouping the above by customer_id

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

Customer A spent $76, customer B spent $74, and customer C spent $36.

### 2. How many days has each customer visited the restaurant?
______________________________________________________________

**Overview:**

Only one table will be needed to solve this - sales - because it has both the customer and the order date.

I solved this by:
1. Using a COUNT DISTINCT on the order_date column (NOTE: You cannot just use COUNT as there are multiple sales on the same day)
2. Grouping the above by customer_id

**SQL Statement:**

```sql
SELECT
s.customer_id AS "Customer"
,COUNT(DISTINCT s.order_date) AS "Num of Visits"

FROM dannys_diner.sales AS s

GROUP BY s.customer_id

ORDER BY s.customer_id
```

**Table Output:**

| Customer | Num of Visits |
| -------- | ------------- |
| A        | 4             |
| B        | 6             |
| C        | 2             |

**Answer:**

Customer A has visited 4 times, customer B has visited 6 times, and customer C has visited 2 times.

### 3. What was the first item from the menu purchased by each customer?
________________________________________________________________________
**Overview**

Two tables will be needed to solve this problem - sales and menu. Sales has the customer_id, order_date, and product_id, while menu has the product_name.

For this problem, I manually calcuated the expected output so I knew exactly what would and wouldn't work to get the right data. For this case, the expected output is:
- A: sushi & curry (both were bought on the same day)
- B: curry
- C: ramen & ramen (both were bought on the same day)

From there, I knew I needed to use either RANK or DENSE_RANK as the ROW_NUMBER function would skip either sushi or curry for A due to labeling one of them as a rank of 2. I chose RANK since I only needed the top 1 values anyway.

So, I solved this by:
1. Using the window function RANK with PARTITION BY on the customer_id
2. Taking the above query and using it as a subquery.
3. Only showing the data where "Rank" is 1

**SQL Statement**

```sql
SELECT
"Customer"
,"Product"

FROM (SELECT
	s.customer_id AS "Customer"
	,s.order_date AS "Order Date"
	,m.product_name AS "Product"
	,RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date ASC) AS "Rank"

	FROM dannys_diner.sales AS s
	LEFT JOIN dannys_diner.menu AS m ON m.product_id = s.product_id
) AS ranked_product

WHERE
"Rank" = 1
```

**Table Output*

| Customer | Product |
| -------- | ------- |
| A        | curry   |
| A        | sushi   |
| B        | curry   |
| C        | ramen   |
| C        | ramen   |

**Answer**

Customer A first bough curry and sushi, customer B first bough curry, and customer C first bought ramen.

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
### 5. Which item was the most popular for each customer?
### 6. Which item was purchased first by the customer after they became a member?
### 7. Which item was purchased just before the customer became a member?
### 8. What is the total items and amount spent for each member before they became a member?
### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?