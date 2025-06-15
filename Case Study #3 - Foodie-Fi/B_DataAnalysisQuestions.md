# Data Analysis Questions
## Table of Contents

[1. How many customers has Foodie-Fi ever had?](#1-how-many-customers-has-foodie-fi-ever-had)

[2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value.](#2-what-is-the-monthly-distribution-of-trial-plan-start_date-values-for-our-dataset---use-the-start-of-the-month-as-the-group-by-value)

[3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.](#3-what-plan-start_date-values-occur-after-the-year-2020-for-our-dataset-show-the-breakdown-by-count-of-events-for-each-plan_name)

[4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?](#4-what-is-the-customer-count-and-percentage-of-customers-who-have-churned-rounded-to-1-decimal-place)

[5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?](#5-how-many-customers-have-churned-straight-after-their-initial-free-trial---what-percentage-is-this-rounded-to-the-nearest-whole-number)

[6. What is the number and percentage of customer plans after their initial free trial?](#6-what-is-the-number-and-percentage-of-customer-plans-after-their-initial-free-trial)

[7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?](#7-what-is-the-customer-count-and-percentage-breakdown-of-all-5-plan_name-values-at-2020-12-31)

[8. How many customers have upgraded to an annual plan in 2020?](#8-how-many-customers-have-upgraded-to-an-annual-plan-in-2020)

[9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?](#9-how-many-days-on-average-does-it-take-for-a-customer-to-an-annual-plan-from-the-day-they-join-foodie-fi)

[10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)?](#10-can-you-further-breakdown-this-average-value-into-30-day-periods-ie-0-30-days-31-60-days-etc)

[11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?](#11-how-many-customers-downgraded-from-a-pro-monthly-to-a-basic-monthly-plan-in-2020)

## Questions and Answers
### 1. How many customers has Foodie-Fi ever had?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| subscriptions | Contains all the customers Foodie-Fi has had |

**SQL Statement:**
	
```sql	
SELECT
COUNT(DISTINCT s.customer_id) AS "Unique Customers" -- Count the unique number of customers

FROM foodie_fi.subscriptions AS s
```

**Table Output:**

| Unique Customers |
| ---------------- |
| 1000             |

**Answer:**

- Foodie-Fi has had 1000 customers.

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value.
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_info_plans | Contains information on the customers, the start_date of each plan, and what plan the customer has on that date | 

**SQL Statement:**
	
```sql	
WITH customer_info_plans AS (SELECT
  s.customer_id
  ,s.plan_id
  ,p.plan_name
  ,p.price
  ,s.start_date

  FROM foodie_fi.subscriptions AS s
  LEFT JOIN foodie_fi.plans AS p ON s.plan_id = p.plan_id
                             
  ORDER BY s.customer_id, s.start_date
)

SELECT
EXTRACT(MONTH FROM cip.start_date) AS "Month Num" -- Get the month number
,TO_CHAR(cip.start_date, 'Month') AS "Month" -- Get the name of the month
,COUNT(cip.customer_id) AS "Plan Count" -- Count of customers (NOTE: DISTINCT isn't needed here because of our WHERE clause)

FROM customer_info_plans AS cip

WHERE
cip.plan_name = 'trial' -- Only look at trial plans

GROUP BY "Month Num", "Month" -- Need to group by these two since we're using the agg. function COUNT

ORDER BY "Month Num" ASC -- Sort the months from 1 -> 12
```

**Table Output:**

| Month Num | Month     | Plan Count |
| --------- | --------- | ---------- |
| 1         | January   | 88         |
| 2         | February  | 68         |
| 3         | March     | 94         |
| 4         | April     | 81         |
| 5         | May       | 88         |
| 6         | June      | 79         |
| 7         | July      | 89         |
| 8         | August    | 88         |
| 9         | September | 87         |
| 10        | October   | 79         |
| 11        | November  | 75         |
| 12        | December  | 84         |

**Answer:**

- See table above.

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:

- 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**


**Answer:**

- 

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:

- 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**


**Answer:**

- 

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:

- 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**


**Answer:**

- 

### 6. What is the number and percentage of customer plans after their initial free trial?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:

- 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**


**Answer:**

- 

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:

- 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**


**Answer:**

- 

### 8. How many customers have upgraded to an annual plan in 2020?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:

- 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**


**Answer:**

- 

### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:

- 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**


**Answer:**

- 

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:

- 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**


**Answer:**

- 

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

Expected Results:

- 

I solved this by:

1. 

**SQL Statement:**
	
```sql	

```

**Table Output:**


**Answer:**

- 