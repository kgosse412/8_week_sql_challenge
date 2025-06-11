# Runner and Customer Experience
## Table of Contents
[1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)](#1-how-many-runners-signed-up-for-each-1-week-period-ie-week-starts-2021-01-01)

[2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?](#2-what-was-the-average-time-in-minutes-it-took-for-each-runner-to-arrive-at-the-pizza-runner-hq-to-pickup-the-order)

[3. Is there any relationship between the number of pizzas and how long the order takes to prepare?](#3-is-there-any-relationship-between-the-number-of-pizzas-and-how-long-the-order-takes-to-prepare)

[4. What was the average distance travelled for each customer?](#4-what-was-the-average-distance-travelled-for-each-customer)

[5. What was the difference between the longest and shortest delivery times for all orders?](#5-what-was-the-difference-between-the-longest-and-shortest-delivery-times-for-all-orders)

[6. What was the average speed for each runner for each delivery and do you notice any trend for these values?](#6-what-was-the-average-speed-for-each-runner-for-each-delivery-and-do-you-notice-any-trend-for-these-values)

[7. What is the successful delivery percentage for each runner?](#7-what-is-the-successful-delivery-percentage-for-each-runner)

## Questions and Answers
### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
_______________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| runners | Contains information about the runner and when they signed up |

Expected Results:
- 2 runners signed up week 1.
- 1 runner signed up week 2.
- 1 runner signed up week 3. 

I solved this by:

1. Using `EXTRACT` with `WEEK` to get the week number.
2. Using a `CASE` statement to set the weeks to show WW53 as WW01. This is because we want the week to start on 2021-01-01.
3. Using `COUNT` to count the number of `runner_id`s.
4. Using `GROUP BY` to group the above by each `"Week"`.
5. Sorting the output by `"Week"` by using `ORDER BY`.

**SQL Statement:**
	
```sql	
SELECT
CASE 
	WHEN EXTRACT(WEEK FROM r.registration_date) = 53 THEN 1
	ELSE EXTRACT(WEEK FROM r.registration_date) + 1
END AS "Week"
,COUNT(r.runner_id) AS "Runner Signup"

FROM pizza_runner.runners AS r

GROUP BY "Week"

ORDER BY "Week"
```

**Table Output:**

| Week | Runner Signup |
| ---- | ------------- |
| 1    | 2             |
| 2    | 1             |
| 3    | 1             |

**Answer:**
- 2 runners signed up week 1.
- 1 runner signed up week 2.
- 1 runner signed up week 3.

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
_________________________________________________________________________________________________________________________

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains information on when the order was placed |
| runner_orders_clean | Contains information on when the order was picked up |

Expected Results:
- Runner 1 takes about 16 minutes.
- Runner 2 takes about 24 minutes.
- Runner 3 takes about 10 minutes.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
2. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
3. Creating a new CTE called `date_diff`. In this CTE, I use `EXTRACT(EPOCH FROM ro.pickup_time - co.order_time)` to figure out the time difference in seconds between when the pizza order is placed and when it is picked up (I chose seconds because figuring out the difference in minutes doesn't include seconds and can underestimate our average). I also used a `WHERE` to exclude any cancelled orders.
4. Querying `date_diff` to get the rounded average of the seconds calculated in `date_diff` divided by 60 (to convert it to minutes). This is done using `ROUND(AVG(dd.time_diff)/60)`.
5. Grouping the above by the `"Runner"` by using `GROUP BY`.
6. Sorting the output by `"Runner"` by using `ORDER BY`.

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
),
date_diff AS (SELECT
	ro.runner_id
	,ro.pickup_time
	,co.order_time
	,EXTRACT(EPOCH FROM ro.pickup_time - co.order_time) AS time_diff

	FROM customer_orders_clean AS co
	INNER JOIN runner_orders_clean AS ro ON co.order_id = ro.order_id

	WHERE
	ro.pickup_time IS NOT NULL

	ORDER BY ro.runner_id ASC
)

SELECT
dd.runner_id AS "Runner"
,ROUND(AVG(dd.time_diff)/60) AS "Avg Minutes"

FROM date_diff AS dd

GROUP BY "Runner"

ORDER BY "Runner"
```

**Table Output:**

| Runner | Avg Minutes |
| ------ | ----------- |
| 1      | 16          |
| 2      | 24          |
| 3      | 10          |

**Answer:**
- Runner 1 takes about 16 minutes to pick up an order.
- Runner 2 takes about 24 minutes to pick up an order.
- Runner 3 takes about 10 minutes to pick up an order.

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
_________________________________________________________________________________________________________________________

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

### 4. What was the average distance travelled for each customer?
_________________________________________________________________

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

### 5. What was the difference between the longest and shortest delivery times for all orders?
______________________________________________________________________________________________

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

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
_________________________________________________________________________________________________________________

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

### 7. What is the successful delivery percentage for each runner?
__________________________________________________________________

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