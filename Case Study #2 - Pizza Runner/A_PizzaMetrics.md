# Table of Contents
[1. How many pizzas were ordered?](#1-How-many-pizzas-were-ordered)

[2. How many unique customer orders were made?](https://github.com/kgosse412/8_week_sql_challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/A_PizzaMetrics.md#2-how-many-unique-customer-orders-were-made)

[3. How many successful orders were delivered by each runner?](https://github.com/kgosse412/8_week_sql_challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/A_PizzaMetrics.md#3-how-many-successful-orders-were-delivered-by-each-runner)

[4. How many of each type of pizza was delivered?](https://github.com/kgosse412/8_week_sql_challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/A_PizzaMetrics.md#4-how-many-of-each-type-of-pizza-was-delivered)

[5. How many Vegetarian and Meatlovers were ordered by each customer?](https://github.com/kgosse412/8_week_sql_challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/A_PizzaMetrics.md#5-how-many-vegetarian-and-meatlovers-were-ordered-by-each-customer)

# Question and Answers
### 1. How many pizzas were ordered?
____________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains all the pizzas ordered by each customer |

Expected Results:
- There were 14 pizzas ordered.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
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
| customer_orders_clean | Contains each unique order by a customer |

Expected Results:
- There are 10 unique orders.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
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
| runner_orders_clean | Contains information about each runner and the order they ran |

Expected Results:
- Runner 1 ran 4 orders.
- Runner 2 ran 3 orders.
- Runner 3 ran 1 order.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
2. Using `COUNT` to count up how many orders were ran.
3. Using `GROUP BY` on the above to group the orders by the `runner_id`.
4. Using `WHERE` to only look at the orders where `cancellation` is `NULL`.

**SQL Statement:**
	
```sql	
WITH runner_orders_clean AS (SELECT
  ro.order_id
  ,ro.runner_id
  ,CASE
      WHEN ro.pickup_time = 'null' OR ro.pickup_time = '' THEN NULL
      ELSE ro.pickup_time::TIMESTAMP
  END AS pickup_time
  ,CASE
      WHEN ro.distance = 'null' OR ro.distance = '' THEN NULL
      WHEN ro.distance LIKE '%km' THEN TRIM('km' FROM ro.distance)::DECIMAL
      ELSE ro.distance::DECIMAL
  END AS distance
  ,CASE
      WHEN ro.duration = 'null' OR ro.duration = '' THEN NULL
      WHEN ro.duration LIKE '%minutes' THEN TRIM('minutes' FROM ro.duration)::INT
      WHEN ro.duration LIKE '%minute' THEN TRIM('minute' FROM ro.duration)::INT
      WHEN ro.duration LIKE '%mins' THEN TRIM ('mins' FROM ro.duration)::INT
      WHEN ro.duration LIKE '%min' THEN TRIM ('min' FROM ro.duration)::INT
      ELSE ro.duration::INT
  END AS duration
  ,CASE
      WHEN ro.cancellation = 'null' OR ro.cancellation = '' THEN NULL
      ELSE ro.cancellation
  END AS cancellation

  FROM pizza_runner.runner_orders AS ro
)

SELECT
runner_id AS "Runner"
,COUNT(order_id) AS "Order Count"

FROM runner_orders_clean

WHERE
cancellation IS NULL

GROUP BY "Runner"
```

**Table Output:**

| Runner | Order Count |
| ------ | ----------- |
| 1      | 4           |
| 2      | 3           |
| 3      | 1           |

**Answer:**
- Runner 1 ran 4 orders.
- Runner 2 ran 3 orders.
- Runner 3 ran 1 order.

### 4. How many of each type of pizza was delivered?
____________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains the pizza that was ordered |
| runner_orders_clean   | Contains the orders that were run   |
| pizza_names    | Contains the name of each pizza     |

Expected Results:
- Meat Lovers was successfully delivered 9 times.
- Vegetarian was successfully delievered 3 times.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
2. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
3. Using a `LEFT JOIN` to join the above two tables together AND another `LEFT JOIN` to join the `pizza_names` table.
4. Using `COUNT` to count up the number of `pizza_id` delivered.
5. Using `GROUP BY` to group the above by the `pizza_name`.
6. Using `WHERE` to exclude any cancelled deliveries since we only want to see what was actually delivered.

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
),
runner_orders_clean AS (SELECT
  ro.order_id
  ,ro.runner_id
  ,CASE
      WHEN ro.pickup_time = 'null' OR ro.pickup_time = '' THEN NULL
      ELSE ro.pickup_time::TIMESTAMP
  END AS pickup_time
  ,CASE
      WHEN ro.distance = 'null' OR ro.distance = '' THEN NULL
      WHEN ro.distance LIKE '%km' THEN TRIM('km' FROM ro.distance)::DECIMAL
      ELSE ro.distance::DECIMAL
  END AS distance
  ,CASE
      WHEN ro.duration = 'null' OR ro.duration = '' THEN NULL
      WHEN ro.duration LIKE '%minutes' THEN TRIM('minutes' FROM ro.duration)::INT
      WHEN ro.duration LIKE '%minute' THEN TRIM('minute' FROM ro.duration)::INT
      WHEN ro.duration LIKE '%mins' THEN TRIM ('mins' FROM ro.duration)::INT
      WHEN ro.duration LIKE '%min' THEN TRIM ('min' FROM ro.duration)::INT
      ELSE ro.duration::INT
  END AS duration
  ,CASE
      WHEN ro.cancellation = 'null' OR ro.cancellation = '' THEN NULL
      ELSE ro.cancellation
  END AS cancellation

  FROM pizza_runner.runner_orders AS ro
)

SELECT
pn.pizza_name AS "Pizza Name"
,COUNT(co.pizza_id) AS "Delivered Pizzas"

FROM customer_orders_clean AS co
LEFT JOIN runner_orders_clean AS ro ON co.order_id = ro.order_id
LEFT JOIN pizza_names as pn ON co.pizza_id = pn.pizza_id

WHERE
ro.cancellation IS NULL

GROUP BY "Pizza Name"
```

**Table Output:**

| Pizza Name | Delivered Pizzas |
| ---------- | ---------------- |
| Meatlovers | 9                |
| Vegetarian | 3                |

**Answer:**
- 9 Meatlovers were delivered.
- 3 Vegetarians were delivered.

### 5. How many Vegetarian and Meatlovers were ordered by each customer?
________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains each customer and the pizzas they ordered |
| pizza_names | Contains the names of each pizza |

Expected Results:
- Customer 101 ordered 2 Meatlovers and 1 Vegetarian.
- Customer 102 ordered 2 Meatlovers and 1 Vegetarian.
- Customer 103 ordered 3 Meatlovers and 1 Vegetarian.
- Customer 104 ordered 3 Meatlovers and 0 Vegetarian.
- Customer 105 ordered 1 Meatlovers and 0 Vegetarian.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
2. Using a `LEFT JOIN` to join the above table and the `pizza_names` table.
3. Using `COUNT` to count up the `pizza_id` data.
4. Using `GROUP BY` to group `customer_id` and `pizza_names` since we want to see how many of each pizza where ordered by each customer.
5. Using `ORDER BY` to clean up the results a bit and showcase them as ordered by first the customer, then the pizza name.

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
co.customer_id AS "Customer"
,pn.pizza_name AS "Pizza Name"
,COUNT(co.pizza_id) AS "Delivered Pizzas"

FROM customer_orders_clean AS co
LEFT JOIN pizza_names as pn ON co.pizza_id = pn.pizza_id

GROUP BY "Customer", "Pizza Name"

ORDER BY "Customer" ASC, "Pizza Name" ASC
```

**Table Output:**

| Customer | Pizza Name | Delivered Pizzas |
| -------- | ---------- | ---------------- |
| 101      | Meatlovers | 2                |
| 101      | Vegetarian | 1                |
| 102      | Meatlovers | 2                |
| 102      | Vegetarian | 1                |
| 103      | Meatlovers | 3                |
| 103      | Vegetarian | 1                |
| 104      | Meatlovers | 3                |
| 105      | Vegetarian | 1                |

**Answer:**
- Customer 101 ordered 2 Meatlovers and 1 Vegetarian.
- Customer 102 ordered 2 Meatlovers and 1 Vegetarian.
- Customer 103 ordered 3 Meatlovers and 1 Vegetarian.
- Customer 104 only ordered 3 Meatlovers.
- Customer 105 only ordered 1 Vegetarian.

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