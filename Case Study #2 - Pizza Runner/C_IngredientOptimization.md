# Ingredient Optimization
## Table of Contents
[1. What are the standard ingredients for each pizza?](#1-what-are-the-standard-ingredients-for-each-pizza)

[2. What was the most commonly added extra?](#2-what-was-the-most-commonly-added-extra)

[3. What was the most common exclusion?](#3-what-was-the-most-common-exclusion)

[4. Generate an order item for each record in the customers_orders table in the format of one of the following:](#4-generate-an-order-item-for-each-record-in-the-customers_orders-table-in-the-format-of-one-of-the-following)

[5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients](#5-generate-an-alphabetically-ordered-comma-separated-ingredient-list-for-each-pizza-order-from-the-customer_orders-table-and-add-a-2x-in-front-of-any-relevant-ingredients)

[6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?](#6-what-is-the-total-quantity-of-each-ingredient-used-in-all-delivered-pizzas-sorted-by-most-frequent-first)

## Questions and Answers
### 1. What are the standard ingredients for each pizza?
________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| pizza_recipes | Contains info on what toppings belong to which pizza |
| pizza_toppings | Contains info on the topping names |
| pizza_names | Contains info on the pizza names |

Expected Results:

| pizza_id | pizza_name  | toppings | topping_name |
| -------- | ----------- | -------- | ------------ |
| 1        | Meat Lovers | 1        | Bacon        |
| 1        | Meat Lovers | 2        | BBQ Sauce    |
| 1        | Meat Lovers | 3        | Beef         |
| 1        | Meat Lovers | 4        | Cheese       |
| 1        | Meat Lovers | 5        | Chicken      |
| 1        | Meat Lovers | 6        | Mushrooms    |
| 1        | Meat Lovers | 8        | Pepperoni    |
| 1        | Meat Lovers | 10       | Salami       |
| 2        | Vegetarian  | 4        | Cheese       |
| 2        | Vegetarian  | 6        | Mushrooms    |
| 2        | Vegetarian  | 7        | Onions       |
| 2        | Vegetarian  | 9        | Peppers      |
| 2        | Vegetarian  | 11       | Tomatoes     |
| 2        | Vegetarian  | 12       | Tomato Sauce |

I solved this by:

1. Using `UNNEST(STRING_TO_ARRAY(pr.toppings, ','))::INT`, I turned the CSV list of toppings from the `pizza_recipes` table into a table where each row has 1 topping id that is converted to an integer.
2. Turning the above into a subquery called upr (for unnested_pizza_recipes).
3. Joining both `pizza_toppings` and `pizza_names` to `upr` to show the `pizza_id`, `pizza_name`, `topping_id` (from the subquery), and `topping_name`.
4. Turning the above into a Common Table Expression (CTE) to be used as needed.

**SQL Statement:**
	
```sql	
WITH pizza_recipes_unnested AS (SELECT
upr.pizza_id
,pn.pizza_name
,upr.topping_id
,pt.topping_name

FROM (SELECT
  pr.pizza_id
  ,UNNEST(STRING_TO_ARRAY(pr.toppings, ','))::INT AS topping_id

  FROM pizza_runner.pizza_recipes AS pr
) AS upr -- unnested_pizza_recipes
JOIN pizza_runner.pizza_toppings AS pt ON upr.topping_id = pt.topping_id
JOIN pizza_runner.pizza_names AS pn ON upr.pizza_id = pn.pizza_id

ORDER BY upr.pizza_id ASC, upr.topping_id ASC
)

SELECT *

FROM pizza_recipes_unnested
```

**Table Output:**

| pizza_id | pizza_name | topping_id | topping_name |
| -------- | ---------- | ---------- | ------------ |
| 1        | Meatlovers | 1          | Bacon        |
| 1        | Meatlovers | 2          | BBQ Sauce    |
| 1        | Meatlovers | 3          | Beef         |
| 1        | Meatlovers | 4          | Cheese       |
| 1        | Meatlovers | 5          | Chicken      |
| 1        | Meatlovers | 6          | Mushrooms    |
| 1        | Meatlovers | 8          | Pepperoni    |
| 1        | Meatlovers | 10         | Salami       |
| 2        | Vegetarian | 4          | Cheese       |
| 2        | Vegetarian | 6          | Mushrooms    |
| 2        | Vegetarian | 7          | Onions       |
| 2        | Vegetarian | 9          | Peppers      |
| 2        | Vegetarian | 11         | Tomatoes     |
| 2        | Vegetarian | 12         | Tomato Sauce |

**Answer:**
- The Meatlovers has bacon, BBQ sauce, beef, cheese, chicken, mushrooms, pepperoni, and salami.
- The Vegetarian has cheese, mushrooms, onions, peppers, tomatoes, and tomato sauce.

### 2. What was the most commonly added extra?
______________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains information on the extra toppings added for each order |
| pizza_toppings | Contains information on the topping names |

Expected Results:
- Bacon is the most commonly added extra with it being added 4 times.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
2. Creating a subquery to unnest the `extras` column and convert it to an integer by using `UNNEST(STRING_TO_ARRAY(co.extras, ','))::INT`.
3. Joining the subquery to the `pizza_recipes_unnested` CTE.
4. Using `COUNT` to count the number of each extra topping.
5. Using `GROUP BY` to group the above by the `"Topping Name"`.
6. Sorting the output by the count in a descending manner by using `ORDER BY "Extras Count" DESC`.
7. Limiting the output to the top 1 by using `LIMIT`.

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
pt.topping_name AS "Topping Name"
,COUNT(uco.extras) AS "Extras Count"

FROM (SELECT
  UNNEST(STRING_TO_ARRAY(co.extras, ','))::INT AS extras

  FROM customer_orders_clean AS co
) AS uco
JOIN pizza_toppings AS pt ON uco.extras = pt.topping_id

GROUP BY "Topping Name"

ORDER BY "Extras Count" DESC

LIMIT 1
```

**Table Output:**

| Topping Name | Extras Count |
| ------------ | ------------ |
| Bacon        | 4            |

**Answer:**
- Bacon is the most commonly added extra.

### 3. What was the most common exclusion?
__________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains information on the extra toppings added for each order |
| pizza_toppings | Contains information on the topping names |

Expected Results:
- Cheese is the most common exclusion with it being excluded 4 times.

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
2. Creating a subquery to unnest the `exclusions` column and convert it to an integer by using `UNNEST(STRING_TO_ARRAY(co.exclusions, ','))::INT`.
3. Joining the subquery to the `pizza_recipes_unnested` CTE.
4. Using `COUNT` to count the number of each excluded topping.
5. Using `GROUP BY` to group the above by the `"Topping Name"`.
6. Sorting the output by the count in a descending manner by using `ORDER BY "Exclusions Count" DESC`.
7. Limiting the output to the top 1 by using `LIMIT`. 

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
pt.topping_name AS "Topping Name"
,COUNT(uco.exclusions) AS "Exclusions Count"

FROM (SELECT
  UNNEST(STRING_TO_ARRAY(co.exclusions, ','))::INT AS exclusions

  FROM customer_orders_clean AS co
) AS uco
JOIN pizza_toppings AS pt ON uco.exclusions = pt.topping_id

GROUP BY "Topping Name"

ORDER BY "Exclusions Count" DESC

LIMIT 1
```

**Table Output:**

| Topping Name | Exclusions Count |
| ------------ | ---------------- |
| Cheese       | 4                |

**Answer:**
- Cheese is the most excluded topping (but why??).

### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
__________________________________________________________________________________________________________________
**Format**
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains information about each order, including extras and excluded toppings |
| exclusions_list | Contains information on the exclusions toppings in CSV text form |
| extras_list | Contains information on the extra toppings in a CSV text form |

Expected Results:

| order_id | pizza |
| -------- | ----- |
| 1        | Meatlovers |
| 2        | Meatlovers |
| 3        | Meatlovers |
| 3        | Vegetarian |
| 4        | Meatlovers - Exclude Cheese |
| 4        | Meatlovers - Exclude Cheese |
| 4        | Vegetarian - Exclude Cheese |
| 5        | Meatlovers - Extra Bacon    |
| 6        | Vegetarian |
| 7        | Vegetarian - Extra Bacon    |
| 8        | Meatlovers |
| 9        | Meatlovers - Exclude Cheese - Extra Bacon, Chicken |
| 10       | Meatlovers |
| 10       | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean`.
2. Creating a CTE called `exclusions_list` that decoupled `exclusions` into individual rows, replaced the `topping_id` with `topping_name`, and converted that back into a CSV list.
3. Creating a CTE called `extras_list` that decoupled `extras` into individual rows, replaced the `topping_id` with `topping_name`, and converted that back into a CSV list.
4. Using `LEFT JOIN` to join `exclusions_list`, `extras_list`, and `pizza_names` to `customer_orders_clean`.
5. Using `CONCAT(pn.pizza_name, COALESCE(' - Exclude ' || excl.exclusions, ''), COALESCE(' - Extra ' || extl.extras, ''))` to join all the parts of the strings together. `COALESCE` checks which value is the first non-null argument, then appends that to the existing string.
6. Sorting the output by the `"Order ID"` by using `ORDER BY`.

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
extras_list AS (SELECT
  uco.row
  ,STRING_AGG(pt.topping_name, ', ') AS extras

  FROM (SELECT
    co.row
    ,UNNEST(STRING_TO_ARRAY(co.extras, ','))::INT AS extras

    FROM customer_orders_clean AS co
  ) AS uco
  JOIN pizza_toppings AS pt ON uco.extras = pt.topping_id

  GROUP BY uco.row

  ORDER BY row
),
exclusions_list AS (SELECT
  uco.row
  ,STRING_AGG(pt.topping_name, ', ') AS exclusions

  FROM (SELECT
    co.row
    ,UNNEST(STRING_TO_ARRAY(co.exclusions, ','))::INT AS exclusions

    FROM customer_orders_clean AS co
  ) AS uco
  JOIN pizza_toppings AS pt ON uco.exclusions = pt.topping_id

  GROUP BY uco.row

  ORDER BY row
)

SELECT
co.order_id AS "Order ID"
,CONCAT(pn.pizza_name,
        COALESCE(' - Exclude ' || excl.exclusions, ''),
        COALESCE(' - Extra ' || extl.extras, '')
) AS Pizza

FROM customer_orders_clean AS co
LEFT JOIN pizza_runner.pizza_names AS pn ON co.pizza_id = pn.pizza_id
LEFT JOIN extras_list AS extl ON co.row = extl.row
LEFT JOIN exclusions_list AS excl ON co.row = excl.row

ORDER BY "Order ID"
```

**Table Output:**

| Order ID | Pizza                                                           |
| -------- | --------------------------------------------------------------- |
| 1        | Meatlovers                                                      |
| 2        | Meatlovers                                                      |
| 3        | Meatlovers                                                      |
| 3        | Vegetarian                                                      |
| 4        | Meatlovers - Exclude Cheese                                     |
| 4        | Meatlovers - Exclude Cheese                                     |
| 4        | Vegetarian - Exclude Cheese                                     |
| 5        | Meatlovers - Extra Bacon                                        |
| 6        | Vegetarian                                                      |
| 7        | Vegetarian - Extra Bacon                                        |
| 8        | Meatlovers                                                      |
| 9        | Meatlovers - Exclude Cheese - Extra Bacon, Chicken              |
| 10       | Meatlovers                                                      |
| 10       | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |

**Answer:**
- See table above.

### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
### For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
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

### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
________________________________________________________________________________________________________________
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