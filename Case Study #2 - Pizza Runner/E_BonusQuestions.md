# Bonus Questions
## Table of Contents

[If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?](#if-danny-wants-to-expand-his-range-of-pizzas---how-would-this-impact-the-existing-data-design-write-an-insert-statement-to-demonstrate-what-would-happen-if-a-new-supreme-pizza-with-all-the-toppings-was-added-to-the-pizza-runner-menu)

## Questions and Answers
### If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

Tables Used:

| Table | Why |
| ----- | --- |
| NONE  | Inserting new data into existing tables |

Expected Results:

| pizza_id | pizza_name  |
| -------- | ----------- |
| 1        | Meat Lovers |
| 2        | Vegetarian  |
| 3        | Supreme     |

| pizza_id | toppings |
| -------- | -------- |
| 1        | 1, 2, 3, 4, 5, 6, 8, 10 |
| 2        | 4, 6, 7, 9, 11, 12 |
| 3        | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12 |

I solved this by:

1. Using `INSERT INTO` to update the `pizza_names` table with the values `3` and `Supreme`.
2. Using `INSERT INTO` to update the `pizza_recipes` table with the values `3` and `1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12`.
3. Querying the tables `pizza_names` and `pizza_recipes` to show how the schema changes with the inserts. 

**SQL Statement:**
	
```sql	
INSERT INTO pizza_names
  ("pizza_id", "pizza_name")
VALUES
  (3, 'Supreme');

INSERT INTO pizza_recipes
  ("pizza_id", "toppings")
VALUES
  (3, '1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12');
  
SELECT *

FROM pizza_runner.pizza_names;

SELECT *

FROM pizza_runner.pizza_recipes;
```

**Table Output:**

| pizza_id | pizza_name |
| -------- | ---------- |
| 1        | Meatlovers |
| 2        | Vegetarian |
| 3        | Supreme    |

| pizza_id | toppings                              |
| -------- | ------------------------------------- |
| 1        | 1, 2, 3, 4, 5, 6, 8, 10               |
| 2        | 4, 6, 7, 9, 11, 12                    |
| 3        | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12 |

**Answer:**
- See tables above.