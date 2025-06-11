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
| customer_orders_clean | Contains information on when the order was placed |
| runner_orders_clean | Contains information on when the order was picked up |

Expected Results:
- 1 pizza takes about 12 min.
- 2 pizzas take about 18 min.
- 3 pizzas take about 29 min.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
2. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
3. Creating a new CTE called `date_diff`. In this CTE, I use `EXTRACT(EPOCH FROM ro.pickup_time - co.order_time)` to figure out the time difference in seconds between when the pizza order is placed and when it is picked up (I chose seconds because figuring out the difference in minutes doesn't include seconds and can underestimate our average). I used `COUNT` to figure out how many pizzas were included in each order, and I also used a `WHERE` to exclude any cancelled orders.
4. Querying `date_diff` to get the rounded average of the seconds calculated in `date_diff` divided by 60 (to convert it to minutes). This is done using `ROUND(AVG(dd.time_diff)/60)`.
5. Grouping the above by the `"Number of Pizzas"` by using `GROUP BY`.
6. Sorting the output by `"Number of Pizzas"` by using `ORDER BY`.

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
	co.order_id
    ,COUNT(co.order_id) AS order_cnt
	,ro.pickup_time
	,co.order_time
	,EXTRACT(EPOCH FROM ro.pickup_time - co.order_time) AS time_diff

	FROM customer_orders_clean AS co
	INNER JOIN runner_orders_clean AS ro ON co.order_id = ro.order_id

	WHERE
	ro.pickup_time IS NOT NULL
    
    GROUP BY co.order_id, ro.pickup_time, co.order_time
)

SELECT
dd.order_cnt AS "Number of Pizzas"
,ROUND(AVG(dd.time_diff)/60) AS "Minutes to Prep"

FROM date_diff AS dd

GROUP BY "Number of Pizzas"

ORDER BY "Number of Pizzas"
```

**Table Output:**

| Number of Pizzas | Minutes to Prep |
| ---------------- | --------------- |
| 1                | 12              |
| 2                | 18              |
| 3                | 29              |

**Answer:**
- 1 pizza takes 12 minutes.
- 2 pizzas take 18 minutes total, or 9 minutes each.
- 3 pizzas take 29 minutes total, or ~ 10 minutes each.
- This shows there is a correlation between the number of pizzas and the time it takes to prep them, with 2 pizzas being the best prep time.

### 4. What was the average distance travelled for each customer?
_________________________________________________________________

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains information on when the customer |
| runner_orders_clean | Contains information on when the distance to the customers place |

Expected Results:
- Customer 101 is roughly 20 km away.
- Customer 102 is roughly 17 km away.
- Customer 103 is roughly 23 km away.
- Customer 104 is roughly 10 km away.
- Customer 105 is roughly 25 km away.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
2. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
3. Using `ROUND(AVG(ro.distance))` to figure out the average rounded distance of each customer.
4. Grouping the above by `"Customer"` by using `GROUP BY`.
5. Sorting the output by `"Customer"` by using `ORDER BY`.

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
co.customer_id AS "Customer"
,ROUND(AVG(ro.distance)) AS "Avg Distance"

FROM customer_orders_clean AS co
INNER JOIN runner_orders_clean AS ro ON co.order_id = ro.order_id

WHERE
ro.cancellation IS NULL

GROUP BY "Customer"

ORDER BY "Customer"
```

**Table Output:**

| Customer | Avg Distance |
| -------- | ------------ |
| 101      | 20           |
| 102      | 17           |
| 103      | 23           |
| 104      | 10           |
| 105      | 25           |

**Answer:**
- Customer 101 is roughly 20 km away.
- Customer 102 is roughly 17 km away.
- Customer 103 is roughly 23 km away.
- Customer 104 is roughly 10 km away.
- Customer 105 is roughly 25 km away.

### 5. What was the difference between the longest and shortest delivery times for all orders?
______________________________________________________________________________________________

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| runner_orders_clean | Contains the delivery times of each order |

Expected Results:
- 30 minutes

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
2. Finding the longest duration time by using `MAX` and finding the shortest duration time by using `MIN`, then subtracting min from max.

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
MAX(ro.duration) - MIN(ro.duration) AS "Duration Difference"

FROM runner_orders_clean as ro
```

**Table Output:**

| Duration Difference |
| ------------------- |
| 30                  |

**Answer:**
- The difference between the longest and shortest delivery times is 30 minutes.

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
_________________________________________________________________________________________________________________

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| runner_orders_clean | Contains the distance and delivery times of each order for each runner |

Expected Results:
| Order ID | Runner | Avg Speed |
| -------- | ------ | --------- |
| 1        | 1      | 37.5      |
| 2        | 1      | 44.4      |
| 3        | 1      | 40.2      |
| 4        | 2      | 35.1      |
| 5        | 3      | 40.0      |
| 7        | 2      | 60.0      |
| 8        | 2      | 93.6      |
| 10       | 1      | 60.0      |

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
2. Using `ROUND((ro.distance / ro.duration) * 60, 1)` to figure out the runner's speed in km / hr and rounding it to 1 decimal.
3. Using `WHERE` to exclude any cancelled orders.

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
ro.order_id AS "Order ID"
,ro.runner_id AS "Runner"
,ROUND((ro.distance / ro.duration) * 60, 1) AS "km / hr"

FROM runner_orders_clean as ro

WHERE
ro.cancellation IS NULL
```

**Table Output:**

| Order ID | Runner | km / hr |
| -------- | ------ | ------- |
| 1        | 1      | 37.5    |
| 2        | 1      | 44.4    |
| 3        | 1      | 40.2    |
| 4        | 2      | 35.1    |
| 5        | 3      | 40.0    |
| 7        | 2      | 60.0    |
| 8        | 2      | 93.6    |
| 10       | 1      | 60.0    |

**Answer:**
- See table above.

### 7. What is the successful delivery percentage for each runner?
__________________________________________________________________

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| runner_orders_clean | Contains the runners and the number of runner deliveries |

Expected Results:
- Runner 1 has a 100% success rate (no cancellations).
- Runner 2 has a 75% success rate (1 cancellation).
- Runner 3 has a 50% success rate (1 cancellation).

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
2. Creating a subquery that creates a column with a 1 for every successful delivery and 0 for every cancelled order.
3. Using `SUM(run_count.successful_run) * 100 / COUNT(run_count.runner_id)` to figure out the percentage of successful deliveries. This adds up the number of successful deliveries, multiplies it by 100 (to make it a percentage), and divides by the number of deliveries the runner had.
4. Using `GROUP BY` to group the above by `"Runner"`.
5. Sorting the output by `"Runner"` by using `ORDER BY`.

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
run_count.runner_id AS "Runner"
,SUM(run_count.successful_run) * 100 / COUNT(run_count.runner_id) AS "Successful Delivery %"

FROM (SELECT
	ro.runner_id
	,CASE
		WHEN ro.cancellation IS NULL THEN 1
    	ELSE 0
	END AS successful_run

	FROM runner_orders_clean as ro
) AS run_count

GROUP BY "Runner"

ORDER BY "Runner"
```

**Table Output:**

| Runner | Successful Delivery % |
| ------ | --------------------- |
| 1      | 100                   |
| 2      | 75                    |
| 3      | 50                    |

**Answer:**
- Runner 1 has a 100% success rate.
- Runner 2 has a 75% success rate.
- Runner 3 has a 50% success rate.