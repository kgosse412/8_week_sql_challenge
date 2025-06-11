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

### 3. What was the most common exclusion?
__________________________________________
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

### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
### - Meat Lovers
### - Meat Lovers - Exclude Beef
### - Meat Lovers - Extra Bacon
### - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
__________________________________________________________________________________________________________________
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