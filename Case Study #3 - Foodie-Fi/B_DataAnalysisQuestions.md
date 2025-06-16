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
| customer_info_plans | Contains information on the plan and plan start_date |

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
cip.plan_id AS "Plan ID"
,cip.plan_name AS "Plan Name"
,COUNT(cip.plan_name) AS "Plan Count" -- Count the number of plans

FROM customer_info_plans AS cip

WHERE
EXTRACT(YEAR FROM cip.start_date) > 2020 -- Only look at data after the year 2020

GROUP BY "Plan ID", "Plan Name" -- Need to group by Plan ID and Plan Name since we're using the agg. function COUNT

ORDER BY "Plan ID" -- Sort the output by Plan ID
```

**Table Output:**

| Plan ID | Plan Name     | Plan Count |
| ------- | ------------- | ---------- |
| 1       | basic monthly | 8          |
| 2       | pro monthly   | 60         |
| 3       | pro annual    | 63         |
| 4       | churn         | 71         |

**Answer:**

- After 2020, there wer 8 basic monthly plans, 60 pro monthly plans, 63 pro annual plans, and 71 cancellations.

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_info_plans | Contains information on the customers and their plans |

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
COUNT(cip.plan_name) AS "Churn Count" -- The number of churn plans
/* To get the percentage, we want to take the count of churn plans, 
divide by the total number of customers (gotten via the subquery), 
and multiply that by 100. Then we round the result to 1 decimal place. */
,ROUND(100.0 * COUNT(cip.plan_name) /
	 /* This subquery returns the unique number of customers (which we know is 1000).
       We use it so we can get the full number of customers without being impacted
       by the outter query's WHERE clause. */
     (SELECT
     COUNT(DISTINCT s.customer_id)
     
     FROM foodie_fi.subscriptions AS s),
     1
) AS "Churn Percentage"

FROM customer_info_plans AS cip

WHERE
cip.plan_name = 'churn' -- We only want to look at the churn plans
```

**Table Output:**

| Churn Count | Churn Percentage |
| ----------- | ---------------- |
| 307         | 30.7             |

**Answer:**

- 307 plans are cancellations (or churn), which is 30.7% of the plans.

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_info_plans | Contains information on the customer and each plan they have |

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
),
cip_lead AS (SELECT
  /* Use the LEAD window function to label each row with the next plan_id grouped 
  by customer_id and sorted by start_date. This will allow us to look if the row
  we're examining is a trial plan and the next row is churn. */
  LEAD(cip.plan_id) OVER(PARTITION BY cip.customer_id ORDER BY cip.start_date) AS next_plan_id
  ,*

  FROM customer_info_plans AS cip

  ORDER BY cip.customer_id ASC, cip.start_date ASC -- Sort by the customer_id first, then row
)

SELECT
/* Count everything based on the filter in the WHERE clause */
COUNT(*) AS "Total Cancellations After Trial"
/* To get the percentage, we want to take the count of plans churned after the
trial plan, divide by the total number of customers (gotten via the subquery),
and multiply that by 100. Then we round to the nearest whole number using ROUND. */
,ROUND(100 * COUNT(*) /
  /* This subquery returns the unique number of customers (which we know is 1000).
  We use it so we can get the full number of customers without being impacted
  by the outter query's WHERE clause. */
  (SELECT
   COUNT(DISTINCT s.customer_id)
   
   FROM foodie_fi.subscriptions AS s)
) AS "Percentage of Cancellations After Trial"

FROM cip_lead AS cipl

/* We only want to look at the rows where the current plan is a trial
and the next row is expected to be churn */
WHERE
cipl.plan_id = 0 AND cipl.next_plan_id = 4
```

**Table Output:**

| Total Cancellations After Trial | Percentage of Cancellations After Trial |
| ------------------------------- | --------------------------------------- |
| 92                              | 9                                       |

**Answer:**

- 92 customers, or 9% of customers, cancelled their subscription after the trial period.

### 6. What is the number and percentage of customer plans after their initial free trial?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_info_plans | Contains information on the customer and each plan they have |

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
),
cip_lead AS (SELECT
  /* Use the LEAD window function to label each row with the next plan_id grouped 
  by customer_id and sorted by start_date. This will allow us to look if the row
  we're examining is a trial plan and what the next plan_id is. */
  LEAD(cip.plan_id) OVER(PARTITION BY cip.customer_id ORDER BY cip.start_date) AS next_plan_id
  /* Use the LEAD window function to label each row with the next plan_name grouped 
  by customer_id and sorted by start_date. This will allow us to look if the row
  we're examining is a trial plan and what the next plan_name is. */
  ,LEAD(cip.plan_name) OVER(PARTITION BY cip.customer_id ORDER BY cip.start_date) AS next_plan_name
  ,*

  FROM customer_info_plans AS cip

  ORDER BY cip.customer_id ASC, cip.start_date ASC -- Sort by the customer_id first, then row
)

SELECT
cipl.next_plan_id AS "Plan ID"
,cipl.next_plan_name AS "Plan Name"
/* Count every next_plan_id based on the filter in the WHERE clause */
,COUNT(cipl.next_plan_id) AS "Total Customers After Trial"
/* To get the percentage, we want to take the count of plans after the
trial plan, divide by the total number of customers (gotten via the subquery),
and multiply that by 100. Then we round to the first decimal using ROUND. */
,ROUND(100.0 * COUNT(cipl.next_plan_id) /
  /* This subquery returns the unique number of customers (which we know is 1000).
  We use it so we can get the full number of customers without being impacted
  by the outter query's WHERE clause. */
  (SELECT
   COUNT(DISTINCT s.customer_id)
   
   FROM foodie_fi.subscriptions AS s),
   1
) AS "Percentage of Customers After Trial"

FROM cip_lead AS cipl

/* We only want to look at the rows where the current plan is a trial */
WHERE
cipl.plan_id = 0

GROUP BY cipl.next_plan_id, cipl.next_plan_name
```

**Table Output:**

| Plan ID | Plan Name     | Total Customers After Trial | Percentage of Customers After Trial |
| ------- | ------------- | --------------------------- | ----------------------------------- |
| 1       | basic monthly | 546                         | 54.6                                |
| 2       | pro monthly   | 325                         | 32.5                                |
| 3       | pro annual    | 37                          | 3.7                                 |
| 4       | churn         | 92                          | 9.2                                 |

**Answer:**

- 546 customers, or 54.6% of customers, signed up for the basic monthly plan after their trial.
- 325 customers, or 32.5% of customers, signed up for the pro monthly plan after their trial.
- 37 customers, or 3.7% of customers, signed up for the pro annual plan after their trial.
- 92 customers, or 9.2% of customers, cancelled their plan after their trial.

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_info_plans | Contains information on the customer and each plan they have |

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
),
cip_next_date AS (SELECT
  *
  /* Use the LEAD window function to label each row with the next start_date grouped by
  customer_id and ordered by start_date. This will leave the latest start_date (which we
  need for the evaluation) as NULL and allow us to filter on that value */
  ,LEAD(start_date) OVER(PARTITION BY cip.customer_id ORDER BY cip.start_date) AS next_date

  FROM customer_info_plans AS cip
  
  WHERE
  cip.start_date <= '2020-12-31'::timestamp
)

SELECT
cipnd.plan_id AS "Plan ID"
,cipnd.plan_name AS "Plan Name"
/* Count the number of plans based on the WHERE clause */
,COUNT(cipnd.plan_name) AS "Total Plans"
/* To get the percentage, we want to take the count of plans, divide by the
total number of customers (gotten via the subquery), and multiply that by
100. Then we round to the first decimal using ROUND. */
,ROUND(100.0 * COUNT(cipnd.plan_name) /
  /* This subquery returns the unique number of customers (which we know is 1000).
  We use it so we can get the full number of customers without being impacted
  by the outter query's WHERE clause. */
  (SELECT
   COUNT(DISTINCT s.customer_id)
   
   FROM foodie_fi.subscriptions AS s),
   1
) AS "Percentage of Total Plans"

FROM cip_next_date as cipnd

/* Only look at the latest start_date for each customer. See
explanation in the CTE for why this works. */
WHERE
cipnd.next_date IS NULL

GROUP BY "Plan ID", "Plan Name"

ORDER BY "Plan ID" ASC
```

**Table Output:**

| Plan ID | Plan Name     | Total Plans | Percentage of Total Plans |
| ------- | ------------- | ----------- | ------------------------- |
| 0       | trial         | 19          | 1.9                       |
| 1       | basic monthly | 224         | 22.4                      |
| 2       | pro monthly   | 326         | 32.6                      |
| 3       | pro annual    | 195         | 19.5                      |
| 4       | churn         | 236         | 23.6                      |

**Answer:**

- 19 customers, or 1.9% of customers, are currently signed up with the trial plan.
- 224 customers, or 22.4% of customers, are currently signed up with the basic monthly plan.
- 326 customers, or 32.6% of customers, are currently signed up with the pro monthly plan.
- 195 customers, or 19.5% of customers, are currently signed up with the pro annual plan.
- 236 customers, or 23.6% of custoemrs, have cancelled their plans.

### 8. How many customers have upgraded to an annual plan in 2020?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_info_plans | Contains information on the customer and each plan they have |

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
/* Count all rows based on the WHERE clause criteria. */
COUNT(*) AS "Number of Pro Annual Upgrades"

FROM customer_info_plans AS cip

/* We only want to look at pro annual plans in the year 2020. */
WHERE
cip.plan_name = 'pro annual' AND
EXTRACT(YEAR FROM cip.start_date) = 2020
```

**Table Output:**

| Number of Pro Annual Upgrade |
| ---------------------------- |
| 195                          |

**Answer:**

- 195 customers have upgraded to the pro annual plan in 2020.

### 9. How many days on average does it take for a customer (to upgrade) to an annual plan from the day they join Foodie-Fi?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_info_plans | Contains information on the customer and each plan they have |

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
),
cip_ordered AS (SELECT
  /* Use the ROW_NUMBER window function to label each row grouped by customer_id
  and ordered by the start_date. This will allow us to look at row 1's start_date,
  which is the date the customer joined Foodie-Fi. */
  ROW_NUMBER() OVER(PARTITION BY cip.customer_id ORDER BY cip.start_date) AS row
  ,*

  FROM customer_info_plans AS cip
),
annual_date AS (SELECT
  *
  /* Use the LEAD window function to label each row with the next start_date grouped
  by customer_id and ordered by start_date. This will allow us to get the pro annual
  plan start date because of our WHERE clause. */
  ,LEAD(cipo.start_date) OVER(PARTITION BY cipo.customer_id ORDER BY cipo.start_date) AS annual_date

  FROM cip_ordered AS cipo
  
  /* Only look at row 1 (which is the date the customer joined Foodie-Fi) or when the
  plan is pro annual (because we need the date of when the customer upgraded to this plan. */
  WHERE
  cipo.row = 1 OR
  cipo.plan_name = 'pro annual'
)

SELECT
/* Round the average to the nearest whole number. */
ROUND(
  /* Take the average of the difference between all the dates. */
  AVG(
    /* Subtract the start_date (i.e. the date the customer joined Foodie-Fi) from the
    annual_date (i.e. the date the customer upgraded to the pro annual plan), and get
    the number of days from the difference. */
    EXTRACT(DAY FROM ad.annual_date::timestamp - ad.start_date)
  )
) AS "Number of Days To Upgrade To Pro Annual"

FROM annual_date AS ad

/* Only look at the rows where there exists both a start_date and an annual_date. This
is done by looking at any row where the annual_date is populated and not NULL. */
WHERE
ad.annual_date IS NOT NULL
```

**Table Output:**

| Number of Days To Upgrade To Pro Annual |
| --------------------------------------- |
| 105                                     |

**Answer:**

- It took 105 days on average for customers to upgrade to the pro annual plan.

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_info_plans | Contains information on the customer and each plan they have |

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
),
cip_ordered AS (SELECT
  /* Use the ROW_NUMBER window function to label each row grouped by customer_id
  and ordered by the start_date. This will allow us to look at row 1's start_date,
  which is the date the customer joined Foodie-Fi. */
  ROW_NUMBER() OVER(PARTITION BY cip.customer_id ORDER BY cip.start_date) AS row
  ,*

  FROM customer_info_plans AS cip
),
annual_date AS (SELECT
  *
  /* Use the LEAD window function to label each row with the next start_date grouped
  by customer_id and ordered by start_date. This will allow us to get the pro annual
  plan start date because of our WHERE clause. */
  ,LEAD(cipo.start_date) OVER(PARTITION BY cipo.customer_id ORDER BY cipo.start_date) AS annual_date

  FROM cip_ordered AS cipo
  
  /* Only look at row 1 (which is the date the customer joined Foodie-Fi) or when the
  plan is pro annual (because we need the date of when the customer upgraded to this plan. */
  WHERE
  cipo.row = 1 OR
  cipo.plan_name = 'pro annual'
),
annual_date_diff AS (SELECT
  /* Subtract the start_date (i.e. the date the customer joined Foodie-Fi) from the
  annual_date (i.e. the date the customer upgraded to the pro annual plan), and get
  the number of days from the difference. */
  EXTRACT(DAY FROM ad.annual_date::timestamp - ad.start_date) AS date_diff

  FROM annual_date AS ad

  /* Only look at the rows where there exists both a start_date and an annual_date. This
  is done by looking at any row where the annual_date is populated and not NULL. */
  WHERE
  ad.annual_date IS NOT NULL
)

SELECT
/* Count all date_diff values. */
COUNT(date_diff) AS "Customers"
/* Use a CASE statement to create the various bins we need in 30 day increments.
This will allow us to group the count above into these bins. */
,CASE
	WHEN add.date_diff <= 30 THEN '0 - 30 Days'
    WHEN add.date_diff >= 31 AND add.date_diff <= 60 THEN '31 - 60 Days'
    WHEN add.date_diff >= 61 AND add.date_diff <= 90 THEN '61 - 90 Days'
    WHEN add.date_diff >= 91 AND add.date_diff <= 120 THEN '91 - 120 Days'
    WHEN add.date_diff >= 121 AND add.date_diff <= 150 THEN '121 - 150 Days'
    WHEN add.date_diff >= 151 AND add.date_diff <= 180 THEN '151 - 180 Days'
    WHEN add.date_diff >= 181 AND add.date_diff <= 210 THEN '181 - 210 Days'
    WHEN add.date_diff >= 210 AND add.date_diff <= 240 THEN '211 - 240 Days'
    WHEN add.date_diff >= 241 AND add.date_diff <= 270 THEN '241 - 270 Days'
    WHEN add.date_diff >= 271 AND add.date_diff <= 300 THEN '271 - 300 Days'
    WHEN add.date_diff >= 301 AND add.date_diff <= 330 THEN '301 - 330 Days'
    WHEN add.date_diff >= 331 AND add.date_diff <= 360 THEN '331 - 360 Days'
	ELSE '> 361 Days'
END AS "Bins"
/* Create a CASE statement to sort the above bins by. */
,CASE
	WHEN add.date_diff <= 30 THEN 1
    WHEN add.date_diff >= 31 AND add.date_diff <= 60 THEN 2
    WHEN add.date_diff >= 61 AND add.date_diff <= 90 THEN 3
    WHEN add.date_diff >= 91 AND add.date_diff <= 120 THEN 4
    WHEN add.date_diff >= 121 AND add.date_diff <= 150 THEN 5
    WHEN add.date_diff >= 151 AND add.date_diff <= 180 THEN 6
    WHEN add.date_diff >= 181 AND add.date_diff <= 210 THEN 7
    WHEN add.date_diff >= 210 AND add.date_diff <= 240 THEN 8
    WHEN add.date_diff >= 241 AND add.date_diff <= 270 THEN 9
    WHEN add.date_diff >= 271 AND add.date_diff <= 300 THEN 10
    WHEN add.date_diff >= 301 AND add.date_diff <= 330 THEN 11
    WHEN add.date_diff >= 331 AND add.date_diff <= 360 THEN 12
	ELSE 13
END AS "Bins Sort"

FROM annual_date_diff AS add

GROUP BY "Bins", "Bins Sort" -- Group our count into the "Bins" and "Bins Sort" columns

ORDER BY "Bins Sort" -- Sort our output by "Bins Sort"
```

**Table Output:**

| Customers | Bins           | Bins Sort |
| --------- | -------------- | --------- |
| 49        | 0 - 30 Days    | 1         |
| 24        | 31 - 60 Days   | 2         |
| 34        | 61 - 90 Days   | 3         |
| 35        | 91 - 120 Days  | 4         |
| 42        | 121 - 150 Days | 5         |
| 36        | 151 - 180 Days | 6         |
| 26        | 181 - 210 Days | 7         |
| 4         | 211 - 240 Days | 8         |
| 5         | 241 - 270 Days | 9         |
| 1         | 271 - 300 Days | 10        |
| 1         | 301 - 330 Days | 11        |
| 1         | 331 - 360 Days | 12        |

**Answer:**

- See table above.

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |

**SQL Statement:**
	
```sql	

```

**Table Output:**


**Answer:**

- 