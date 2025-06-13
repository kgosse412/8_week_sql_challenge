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
| customer_orders_clean | Contains information on each customer order, including the type of pizza and any toppings excluded or added as extra |
| exclusions_list | Contains the list of excluded toppings |
| extras_list | Contains the list of extra toppings
| pizza_recipes_unnested | Contains all the toppings in an alphabetical list |

Expected Results:

| order_id | ingredients |
| -------- | ----------- |
| 1        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 2        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 3        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 3        | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce |
| 4        | Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami |
| 4        | Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami |
| 4        | Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce |
| 5        | 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 6        | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce |
| 7        | Bacon, Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce |
| 8        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 9        | 2x Bacon, BBQ Sauce, Beef, 2x Chicken, Mushrooms, Pepperoni, Salami |
| 10       | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 10       | 2x Bacon, Beef, 2x Cheese, Chicken, Pepperoni, Salami |

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean` with a `ROW` calcuation.
2. Using my CTE table called `pizza_recipes_unnested` from question 1.
3. Creating a CTE called `exclusions_list`. This CTE has a subquery that unnests the comma separated list of `exclusions` from `customer_orders_clean` so each value has its own row. Then the subquery is joined to `pizza_toppings` to pull in the `topping_id` and `topping_name` so we know what each exclusion is in text form.
4. Creating a CTE called `extras_list`. This CTE has a subquery that unnests the comma separated list of `extras` from `customer_orders_clean` so each value has its own row. Then the subquery is joined to `pizza_toppings` to pull in the `topping_id` and `topping_name` so we know what each extra is in text form.
5. Creating a CTE called `ingredients_list_full` that creates an entire list of every ingredient used and excluded. The `UNION ALL` allows us to combine the extras to the original ingredients list so we can later use that to count the number of each ingredient per order.
6. Creating a CTE called `ingredient_list_count` that does two major things - one, it removes any excluded ingredients from our list via `WHERE ilf.exclusion IS NULL` and two, it counts how many of each ingredient is used for each order via `COUNT(*)`. The query also groups the non-aggregate fields by using `GROUP BY`, and then sorts the output using an `ORDER BY`.
7. Creating a final CTE called `ingredients_list_csv` that takes the counted list from `ingredient_list_count` and turns it into an alphabetical, comma separated list. The `CASE` statement decides whether a count should be in from of the `topping_name` or not.
8. Creating a query that joins the `customer_orders_clean` table to the above `ingredients_list_csv` table on `row` so we can show each order's CSV list of ingredient toppings. The output is then sorted by `"Order ID"` using a `ORDER BY`.

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
pizza_recipes_unnested AS (SELECT
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
),
exclusions_list AS (SELECT
  uco.row
  ,pt.topping_id
  ,pt.topping_name AS exclusion

  FROM (SELECT
    co.row
    ,UNNEST(STRING_TO_ARRAY(co.exclusions, ','))::INT AS exclusions

    FROM customer_orders_clean AS co
  ) AS uco
  JOIN pizza_toppings AS pt ON uco.exclusions = pt.topping_id

  ORDER BY row
),
extras_list AS (SELECT
  uco.row
  ,pt.topping_id
  ,pt.topping_name AS extra

  FROM (SELECT
    co.row
    ,UNNEST(STRING_TO_ARRAY(co.extras, ','))::INT AS extras

    FROM customer_orders_clean AS co
  ) AS uco
  JOIN pizza_toppings AS pt ON uco.extras = pt.topping_id

  ORDER BY row
),
ingredients_list_full AS (SELECT
  co.row
  ,pru.topping_id
  ,pru.topping_name
  ,excl.exclusion

  FROM customer_orders_clean AS co
  LEFT JOIN pizza_recipes_unnested AS pru ON co.pizza_id = pru.pizza_id
  LEFT JOIN exclusions_list AS excl ON co.row = excl.row AND pru.topping_name = excl.exclusion

  UNION ALL

  SELECT
  extl.row
  ,extl.topping_id
  ,extl.extra
  ,NULL AS exclusion

  FROM extras_list AS extl
),
ingredient_list_count AS (SELECT
  ilf.row
  ,ilf.topping_id
  ,ilf.topping_name
  ,COUNT(*) AS topping_count

  FROM ingredients_list_full ilf

  WHERE
  ilf.exclusion IS NULL

  GROUP BY ilf.row, ilf.topping_id, ilf.topping_name
                          
  ORDER BY ilf.row, ilf.topping_id
),
ingredients_list_csv AS (SELECT
  ilc.row
  ,STRING_AGG(
      CASE
          WHEN ilc.topping_count = 1 THEN ilc.topping_name
          ELSE ilc.topping_count::TEXT || 'x ' || ilc.topping_name
      END,
      ', '
      ORDER BY ilc.topping_name
  ) AS ingredients_csv

  FROM ingredient_list_count AS ilc

  GROUP BY ilc.row
)

SELECT
co.order_id AS "Order ID"
,ilcsv.ingredients_csv AS "Ingredients"

FROM customer_orders_clean AS co
JOIN ingredients_list_csv AS ilcsv ON co.row = ilcsv.row

ORDER BY "Order ID" ASC
```

**Table Output:**

| Order ID | Ingredients                                                              |
| -------- | ------------------------------------------------------------------------ |
| 1        | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 2        | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes               |
| 4        | BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 4        | BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 4        | Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                       |
| 5        | BBQ Sauce, 2x Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 6        | Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes               |
| 7        | Bacon, Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes        |
| 8        | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 9        | BBQ Sauce, 2x Bacon, Beef, 2x Chicken, Mushrooms, Pepperoni, Salami      |
| 10       | BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 10       | 2x Bacon, Beef, 2x Cheese, Chicken, Pepperoni, Salami                    |

**Answer:**
- See table above.

### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_orders_clean | Contains information on the pizza ordered, including any toppings that were excluded or added as extras |
| runner_orders_clean | Contains information on what orders were delivered v cancelled |
| pizza_recipes_unnested | Contains a row by row list of the ingredients |
| exclusions_list | Contains a list of the excluded toppings |
| extras_list | Contains a list of the extra toppings |
| ingredients_list_full | Contains the full row by row list of the ingredients used per each order |

Expected Results:

| topping_name | topping_count |
| ------------ | ------------- |
| Bacon        | 12            |
| Mushrooms    | 11            |
| Cheese       | 10            |
| Chicken      | 9             |
| Beef         | 9             |
| Pepperoni    | 9             |
| Salami       | 9             |
| BBQ Sauce    | 8             |
| Onions       | 3             |
| Peppers      | 3             |
| Tomato Sauce | 3             |
| Tomatoes     | 3             |

I solved this by:

1. Using my cleaned up Common Table Expression (CTE) table called `customer_orders_clean` with a `ROW` calcuation.
2. Using my cleaned up CTE table called `runner_orders_clean`.
2. Using my CTE `pizza_recipes_unnested` from question 5.
3. Using my CTE `exclusions_list` from question 5.
4. Using my CTE `extras_list` from question 5 with an added column called `order_id`.
5. Using my CTE `ingredients_list_full` from question 5 with added columns called `order_id`.
6. Creating a query that counts the `topping_id` by using `COUNT` after removing any excluded toppings using `WHERE ilf.exclusion IS NULL`. It also removes any pizzas that weren't delivered by joining the tabl `ingredients_list_full` to the table `runner_orders_clean` and using `WHERE ro.cancellation IS NULL`.  This is then grouped by the `"Topping Name"` using `GROUP BY`, and the output is sorted first by `"Topping ID Count"` in descending order (to get the most commonly used ingredient first) and then by `"Topping Name"` in ascending order to get anything with the same count in alphabetical order. 

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
pizza_recipes_unnested AS (SELECT
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
),
exclusions_list AS (SELECT
  uco.row
  ,pt.topping_id
  ,pt.topping_name AS exclusion

  FROM (SELECT
    co.row
    ,UNNEST(STRING_TO_ARRAY(co.exclusions, ','))::INT AS exclusions

    FROM customer_orders_clean AS co
  ) AS uco
  JOIN pizza_toppings AS pt ON uco.exclusions = pt.topping_id

  ORDER BY row
),
extras_list AS (SELECT
  uco.row
  ,uco.order_id
  ,pt.topping_id
  ,pt.topping_name AS extra

  FROM (SELECT
    co.row
    ,co.order_id
    ,UNNEST(STRING_TO_ARRAY(co.extras, ','))::INT AS extras

    FROM customer_orders_clean AS co
  ) AS uco
  JOIN pizza_toppings AS pt ON uco.extras = pt.topping_id

  ORDER BY row
),
ingredients_list_full AS (SELECT
  co.row
  ,co.order_id
  ,pru.topping_id
  ,pru.topping_name
  ,excl.exclusion

  FROM customer_orders_clean AS co
  LEFT JOIN pizza_recipes_unnested AS pru ON co.pizza_id = pru.pizza_id
  LEFT JOIN exclusions_list AS excl ON co.row = excl.row AND pru.topping_name = excl.exclusion

  UNION ALL

  SELECT
  extl.row
  ,extl.order_id
  ,extl.topping_id
  ,extl.extra
  ,NULL AS exclusion

  FROM extras_list AS extl
)

SELECT
ilf.topping_name "Topping Name"
,COUNT(ilf.topping_id) AS "Topping ID Count"

FROM ingredients_list_full AS ilf
LEFT JOIN runner_orders_clean AS ro ON ilf.order_id = ro.order_id

WHERE
ilf.exclusion IS NULL AND
ro.cancellation IS NULL

GROUP BY "Topping Name"

ORDER BY "Topping ID Count" DESC, "Topping Name" ASC
```

**Table Output:**

| Topping Name | Topping ID Count |
| ------------ | ---------------- |
| Bacon        | 12               |
| Mushrooms    | 11               |
| Cheese       | 10               |
| Beef         | 9                |
| Chicken      | 9                |
| Pepperoni    | 9                |
| Salami       | 9                |
| BBQ Sauce    | 8                |
| Onions       | 3                |
| Peppers      | 3                |
| Tomato Sauce | 3                |
| Tomatoes     | 3                |

**Answer:**
- See above table.