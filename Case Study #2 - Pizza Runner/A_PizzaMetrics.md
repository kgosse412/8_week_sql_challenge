### 1. How many pizzas were ordered?
____________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders | Contains all the pizzas ordered by each customer |

Expected Results:
- There were 14 pizzas ordered.

I solved this by:

1. Using my cleaned up Commont Table Expression (CTE) table called `customer_orders_clean`.
2. Using `COUNT` on every row to get a count of each pizza ordered.

**SQL Statement:**
	
```sql	
WITH customer_orders_clean AS (SELECT
	co.order_id
	,co.customer_id
	,co.pizza_id
	,CASE
		WHEN co.exclusions = 'null' OR co.exclusions = '' THEN NULL
   	 	ELSE co.exclusions
	END AS exclusions
	,CASE
		WHEN co.extras = 'null' OR co.extras = '' THEN NULL
    	ELSE co.extras
	END AS extras
	,co.order_time

	FROM pizza_runner.customer_orders AS co
)

SELECT
COUNT(*) AS "Pizza Count"

FROM customer_orders_clean
```

**Table Output:**

| Pizza Count |
| ----------- |
| 14          |

**Answer:**
- 14 pizzas have been ordered.

### 2. How many unique customer orders were made?
_________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders | Contains each unique order by a customer |

Expected Results:
- There are 10 unique orders.

I solved this by:

1. Using my cleaned up Commont Table Expression (CTE) table called `customer_orders_clean`.
2. Using `COUNT DISTINCT` on `order_id` to get the unique count of each order.

**SQL Statement:**
	
```sql	
WITH customer_orders_clean AS (SELECT
	co.order_id
	,co.customer_id
	,co.pizza_id
	,CASE
		WHEN co.exclusions = 'null' OR co.exclusions = '' THEN NULL
   	 	ELSE co.exclusions
	END AS exclusions
	,CASE
		WHEN co.extras = 'null' OR co.extras = '' THEN NULL
    	ELSE co.extras
	END AS extras
	,co.order_time

	FROM pizza_runner.customer_orders AS co
)

SELECT
COUNT(DISTINCT order_id) AS "Order Count"

FROM customer_orders_clean
```

**Table Output:**

| Order Count |
| ----------- |
| 10          |

**Answer:**
- There are 10 unique orders.

### 3. How many successful orders were delivered by each runner?
________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:
- (expected result) 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**

| Name 1 | Name 2 |
| ------ | ------ |

**Answer:**
- (answer)

### 4. How many of each type of pizza was delivered?
____________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:
- (expected result) 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**

| Name 1 | Name 2 |
| ------ | ------ |

**Answer:**
- (answer)

### 5. How many Vegetarian and Meatlovers were ordered by each customer?
________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:
- (expected result) 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**

| Name 1 | Name 2 |
| ------ | ------ |

**Answer:**
- (answer)

### 6. What was the maximum number of pizzas delivered in a single order?
_________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:
- (expected result) 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**

| Name 1 | Name 2 |
| ------ | ------ |

**Answer:**
- (answer)

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
______________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:
- (expected result) 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**

| Name 1 | Name 2 |
| ------ | ------ |

**Answer:**
- (answer)

### 8. How many pizzas were delivered that had both exclusions and extras?
__________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:
- (expected result) 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**

| Name 1 | Name 2 |
| ------ | ------ |

**Answer:**
- (answer)

### 9. What was the total volume of pizzas ordered for each hour of the day?
____________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:
- (expected result) 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**

| Name 1 | Name 2 |
| ------ | ------ |

**Answer:**
- (answer)

### 10. What was the volume of orders for each day of the week?
_______________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:
- (expected result) 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**

| Name 1 | Name 2 |
| ------ | ------ |

**Answer:**
- (answer)