### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains information on what pizza was ordered |
| runner_orders_clean | Contains information on which pizzas were delivered vs cancelled |

Expected Results:
- Pizza Runner has made $138 if you just look at delivered pizzas. 

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
2. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
3. Using a `CASE` statement to decided when a pizza is $12 or $10, then adding up the values using `SUM`.
4. Joining the `customer_orders_clean` table to the `runner_orders_clean` table so I can use `WHERE ro.cancellation IS NULL` to prevent pulling in any pizza orders that were cancelled.

**SQL Statement:**
	
```sql	
WITH customer_orders_clean AS (SELECT
	ROW_NUMBER() OVER() AS row
    ,co.order_id
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
SUM(CASE
	WHEN co.pizza_id = 1 THEN 12
    WHEN co.pizza_id = 2 THEN 10
    ELSE 0
END) AS "Total Pizza Earnings"

FROM customer_orders_clean AS co
LEFT JOIN runner_orders_clean AS ro ON co.order_id = ro.order_id

WHERE
ro.cancellation IS NULL
```

**Table Output:**

| Total Pizza Earnings |
| -------------------- |
| 138                  |

**Answer:**
- Danny's Pizza Runner has made $138 so far.

### 2. What if there was an additional $1 charge for any pizza extras?
### Add cheese is $1 extra
________________________________________________________________________________________________________________________
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

3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
________________________________________________________________________________________________________________________
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

### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
### customer_id
### order_id
### runner_id
### rating
### order_time
### pickup_time
### Time between order and pickup
### Delivery duration
### Average speed
### Total number of pizzas
________________________________________________________________________________________________________________________
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

### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
________________________________________________________________________________________________________________________
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
