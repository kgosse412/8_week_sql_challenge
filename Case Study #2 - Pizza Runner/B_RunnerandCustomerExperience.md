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