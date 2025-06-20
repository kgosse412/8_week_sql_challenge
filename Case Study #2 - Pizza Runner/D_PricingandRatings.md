# Ingredient Optimization
## Table of Contents
[1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?](#1-if-a-meat-lovers-pizza-costs-12-and-vegetarian-costs-10-and-there-were-no-charges-for-changes---how-much-money-has-pizza-runner-made-so-far-if-there-are-no-delivery-fees)

[2. What if there was an additional $1 charge for any pizza extras?](#2-what-if-there-was-an-additional-1-charge-for-any-pizza-extras)

[3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.](#3-the-pizza-runner-team-now-wants-to-add-an-additional-ratings-system-that-allows-customers-to-rate-their-runner-how-would-you-design-an-additional-table-for-this-new-dataset---generate-a-schema-for-this-new-table-and-insert-your-own-data-for-ratings-for-each-successful-customer-order-between-1-to-5)

[4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?](#4-using-your-newly-generated-table---can-you-join-all-of-the-information-together-to-form-a-table-which-has-the-following-information-for-successful-deliveries)

[5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?](#5-if-a-meat-lovers-pizza-was-12-and-vegetarian-10-fixed-prices-with-no-cost-for-extras-and-each-runner-is-paid-030-per-kilometre-traveled---how-much-money-does-pizza-runner-have-left-over-after-these-deliveries)

## Questions and Answers
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
### Ex. Add cheese is $1 extra
________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains information on what pizza was ordered |
| runner_orders_clean | Contains information on which pizzas were delivered vs cancelled |

Expected Results:
- Pizza Runner has made $142 if you add an additional $1 charge for any pizza extras.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
2. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
3. Creating a CTE called `extras_list` that counts the unnested toppings from the `extras` column of the `customer_orders_clean` table by using `COUNT`.
4. Taking the query from the previous question and adding `+ SUM(extl.extras_count)` after the `SUM(CASE... END)` statement. This adds up the count of the extra ingredients from the CTE `extras_list`, and then adds that to the number from `SUM(CASE... END)`.
5. Joining another table to `customer_orders_clean` called `extras_list`. This is how the `extras_count` field is pulled in.

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
),
extras_list AS (SELECT
  uco.row
  ,COUNT(uco.extras) AS extras_count

  FROM (SELECT
    co.row
    ,UNNEST(STRING_TO_ARRAY(co.extras, ','))::INT AS extras

    FROM customer_orders_clean AS co
  ) AS uco
  JOIN pizza_toppings AS pt ON uco.extras = pt.topping_id
  
  GROUP BY uco.row
                
  ORDER BY row
)

SELECT
SUM(CASE
	WHEN co.pizza_id = 1 THEN 12
    WHEN co.pizza_id = 2 THEN 10
    ELSE 0
END) +
SUM(extl.extras_count) AS "Total Pizza Earnings"

FROM customer_orders_clean AS co
LEFT JOIN runner_orders_clean AS ro ON co.order_id = ro.order_id
LEFT JOIN extras_list AS extl ON co.row = extl.row

WHERE
ro.cancellation IS NULL
```

**Table Output:**

| Total Pizza Earnings |
| -------------------- |
| 142                  |

**Answer:**
- Danny's Pizza Runner has made $142.

### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| NONE  | Creating a table, not using an existing one |

Expected Results:
- N\A

I solved this by:

1. Using `DROP TABLE IF EXISTS runner_ratings;` to remove any table called `runner_ratings` from the schema _if_ it exists in the schema.
2. Using `CREATE TABLE` to create a table called `runner_ratings`. This table has 4 columns - `order_id`, `runner_id`, `rating`, and `review`.
3. Adding data to the newly created table by using `INSERT INTO`.
4. Pulling all data from the newly created table.

**SQL Statement:**
	
```sql	
DROP TABLE IF EXISTS runner_ratings;

CREATE TABLE runner_ratings (
  "order_id" INTEGER NOT NULL,
  "runner_id" INTEGER NOT NULL,
  "rating" INTEGER NOT NULL,
  "review" TEXT
);

INSERT INTO runner_ratings
	("order_id","runner_id","rating","review")
VALUES
	(1, 1, 4, 'Pizza was fresh but runner almost delivered to wrong address.'),
    (2, 1, 5, 'Fast delivery, fresh pizza!'),
    (3, 1, 5, 'Couldn''t have gotten here any faster!'),
    (4, 2, 2, 'Runner took a long time. Pizza was cold.'),
    (5, 3, 5, 'I''d rate higher if I could! Super fast delivery!'),
    (7, 2, 5, NULL),
    (8, 2, 5, 'Fastest delivery I''ve every had for any pizza ever!'),
    (10, 1, 3,'Delivery was fast but runner delivered to neighbor''s house.');

SELECT *

FROM runner_ratings
```

**Table Output:**

| order_id | runner_id | rating | review                                                        |
| -------- | --------- | ------ | ------------------------------------------------------------- |
| 1        | 1         | 4      | Pizza was fresh but runner almost delivered to wrong address. |
| 2        | 1         | 5      | Fast delivery, fresh pizza!                                   |
| 3        | 1         | 5      | Couldn't have gotten here any faster!                         |
| 4        | 2         | 2      | Runner took a long time. Pizza was cold.                      |
| 5        | 3         | 5      | I'd rate higher if I could! Super fast delivery!              |
| 7        | 2         | 5      |                                                               |
| 8        | 2         | 5      | Fastest delivery I've every had for any pizza ever!           |
| 10       | 1         | 3      | Delivery was fast but runner delivered to neighbor's house.   |

**Answer:**
- See table above.

### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
________________________________________________________________________________________________________________________
**Format**

- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains the customer_id, order_id, and order_time |
| runner_orders_clean | Contains the runner_id, order_id, and pickup_time    |
| runner_ratings | Contains the order_id, runner_id, and rating |

Expected Results:

| customer_id | order_id | runner_id | rating | order_time          | pickup_time         | Time Between Order and Pickup | duration | Average Speed | Total Number of Pizzas |
| ----------- | -------- | --------- | ------ | ------------------- | ------------------- | ----------------------------- | -------- | ------------- | ---------------------- |
| 101         | 1        | 1         | 4      | 2020-01-01 18:05:02 | 2020-01-01 18:15:34 | 11                            | 32       | 37.5          | 1                      |
| 101         | 2        | 1         | 5      | 2020-01-01 19:00:52 | 2020-01-01 19:10:54 | 10                            | 27       | 44.4          | 1                      |
| 102         | 3        | 1         | 5      | 2020-01-02 23:51:23 | 2020-01-03 00:12:37 | 21                            | 20       | 40.2          | 2                      |
| 103         | 4        | 2         | 2      | 2020-01-04 13:23:46 | 2020-01-04 13:53:03 | 29                            | 40       | 35.1          | 3                      |
| 104         | 5        | 3         | 5      | 2020-01-08 21:00:29 | 2020-01-08 21:10:57 | 10                            | 15       | 40.0          | 1                      |
| 105         | 7        | 2         | 5      | 2020-01-08 21:20:29 | 2020-01-08 21:30:45 | 10                            | 25       | 60.0          | 1                      |
| 102         | 8        | 2         | 5      | 2020-01-09 23:54:33 | 2020-01-10 00:15:02 | 20                            | 15       | 93.6          | 1                      |
| 104         | 10       | 1         | 3      | 2020-01-11 18:34:49 | 2020-01-11 18:50:20 | 16                            | 10       | 60.0          | 2                      |

I solved this by:

1. Using the commands from question 3 to create the `runner_ratings` table.
2. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
3. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
4. Taking the seconds from `pickup_time - order_time` and dividing by 60 to get it into minutes before rounding up to get the `Time Between Order and Pickup`.
5. Figuring out the `Average Speed` by using taking the `distance / duration * 60` and rounding to 1 decmial digit using `ROUND`.
6. Counting the number of pizzas by using `COUNT`.
7. Joining the `runner_orders_table` and the `runner_ratings` tables to `customer_orders_clean`.
8. Specifying we only want successful deliveries by using `WHERE ro.cancellation IS NULL`.
9. Grouping the appropriate columns by using `GROUP BY`.
10. Sorting the output by `order_id` using `ORDER BY`.

**SQL Statement:**
	
```sql	
DROP TABLE IF EXISTS runner_ratings;

CREATE TABLE runner_ratings (
  "order_id" INTEGER NOT NULL,
  "runner_id" INTEGER NOT NULL,
  "rating" INTEGER NOT NULL,
  "review" TEXT
);

INSERT INTO runner_ratings
	("order_id","runner_id","rating","review")
VALUES
	(1, 1, 4, 'Pizza was fresh but runner almost delivered to wrong address.'),
    (2, 1, 5, 'Fast delivery, fresh pizza!'),
    (3, 1, 5, 'Couldn''t have gotten here any faster!'),
    (4, 2, 2, 'Runner took a long time. Pizza was cold.'),
    (5, 3, 5, 'I''d rate higher if I could! Super fast delivery!'),
    (7, 2, 5, NULL),
    (8, 2, 5, 'Fastest delivery I''ve every had for any pizza ever!'),
    (10, 1, 3,'Delivery was fast but runner delivered to neighbor''s house.');

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
co.customer_id
,co.order_id
,ro.runner_id
,rr.rating
,co.order_time
,ro.pickup_time
,ROUND(EXTRACT(EPOCH FROM ro.pickup_time - co.order_time)/60) AS "Time Between Order and Pickup"
,ro.duration
,ROUND((ro.distance / ro.duration) * 60, 1) AS "Average Speed"
,COUNT(*) AS "Total Number of Pizzas"

FROM customer_orders_clean AS co
LEFT JOIN runner_orders_clean AS ro ON co.order_id = ro.order_id
LEFT JOIN runner_ratings AS rr ON co.order_id = rr.order_id

WHERE
ro.cancellation IS NULL

GROUP BY co.customer_id, co.order_id, ro.runner_id, rr.rating, co.order_time, ro.pickup_time, ro.duration, ro.distance

ORDER BY co.order_id ASC
```

**Table Output:**

| customer_id | order_id | runner_id | rating | order_time          | pickup_time         | Time Between Order and Pickup | duration | Average Speed | Total Number of Pizzas |
| ----------- | -------- | --------- | ------ | ------------------- | ------------------- | ----------------------------- | -------- | ------------- | ---------------------- |
| 101         | 1        | 1         | 4      | 2020-01-01 18:05:02 | 2020-01-01 18:15:34 | 11                            | 32       | 37.5          | 1                      |
| 101         | 2        | 1         | 5      | 2020-01-01 19:00:52 | 2020-01-01 19:10:54 | 10                            | 27       | 44.4          | 1                      |
| 102         | 3        | 1         | 5      | 2020-01-02 23:51:23 | 2020-01-03 00:12:37 | 21                            | 20       | 40.2          | 2                      |
| 103         | 4        | 2         | 2      | 2020-01-04 13:23:46 | 2020-01-04 13:53:03 | 29                            | 40       | 35.1          | 3                      |
| 104         | 5        | 3         | 5      | 2020-01-08 21:00:29 | 2020-01-08 21:10:57 | 10                            | 15       | 40.0          | 1                      |
| 105         | 7        | 2         | 5      | 2020-01-08 21:20:29 | 2020-01-08 21:30:45 | 10                            | 25       | 60.0          | 1                      |
| 102         | 8        | 2         | 5      | 2020-01-09 23:54:33 | 2020-01-10 00:15:02 | 20                            | 15       | 93.6          | 1                      |
| 104         | 10       | 1         | 3      | 2020-01-11 18:34:49 | 2020-01-11 18:50:20 | 16                            | 10       | 60.0          | 2                      |

**Answer:**
- See table above.

### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains information on the pizzas ordered |
| runner_orders_clean | Contains information on the distance each runner traveled |

Expected Results:
- Pizza Runner has $94.44 left after deliveries.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
2. Using my cleaned up Common Table Expression (CTE) table called `runner_orders_clean`.
3. Using the query from question 1 to figure out the total pizza earnings.
4. Creating a new query that calculates the total amount paid to the runners via `SUM(ro.distance) * .30`.
5. Subtracting the query from step 4 from the query from step 3.

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
(
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
) -
(
  SELECT
  SUM(ro.distance) * .30 AS "Amount Paid to Runners"

  FROM runner_orders_clean AS ro
) AS "Pizza Earnings"
```

**Table Output:**

| Pizza Earnings |
| -------------- |
| 94.440         |

**Answer:**
- Pizza Runner has $94.44 left over after paying the runners.
