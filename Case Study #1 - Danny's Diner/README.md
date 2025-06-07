# Case Study #1 - Danny's Diner
## Overview
All information related to this case study can be found at [Case Study #1 - Danny's Diner](https://8weeksqlchallenge.com/case-study-1/).

## Questions and Answers
### 1. What is the total amount each customer spent at the restaurant?
Overview:

Two tables will be needed to solve this - sales and menu. Sales will have the information of what each customer bought and menu will have the price of each bought item.

I solved this by:
1. Using a join to connect the two tables
2. Using SUM to add up the prices of the bought items
3. Grouping the above by the customer

SQL Statement:
	
	SELECT
	s.customer_id AS "Customer"
	,SUM(m.price) AS "Total Spent"

	FROM dannys_diner.sales AS s
	LEFT JOIN dannys_diner.menu AS m ON m.product_id = s.product_id

	GROUP BY s.customer_id

	ORDER BY s.customer_id

Table Output:

| Customer | Total Spent |
| -------- | ----------- |
| A        | 76          |
| B        | 74          |
| C        | 36          |

Answer:

Customer A spent $76, customer B spent $74, and customer C spent $36.

### 2. How many days has each customer visited the restaurant?
### 3. What was the first item from the menu purchased by each customer?
### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
### 5. Which item was the most popular for each customer?
### 6. Which item was purchased first by the customer after they became a member?
### 7. Which item was purchased just before the customer became a member?
### 8. What is the total items and amount spent for each member before they became a member?
### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?