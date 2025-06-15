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

I solved this by:

1. Using `COUNT(DISTINCT s.customer_id)` to get a unique count of the number of customers. 

**SQL Statement:**
	
```sql	
SELECT
COUNT(DISTINCT s.customer_id) AS "Unique Customers"

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