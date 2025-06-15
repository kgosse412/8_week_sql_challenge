# Customer Journey
## Questions and Answers
### Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.
___________________________________________________________________________________________________________________________
> Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| subscriptions | Contains each customer's subscription plan_id and start_date |
| plans | Contains information about each plan |

I solved this by:

1. Creating a Common Table Expression (CTE) called `customer_info_plans` that joins both the `subscription` and the `plans` tables together.

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

SELECT *

FROM customer_info_plans
```

**Table Output:**

| customer_id | plan_id | plan_name     | price  | start_date |
| ----------- | ------- | ------------- | ------ | ---------- |
| 1           | 0       | trial         | 0.00   | 2020-08-01 |
| 1           | 1       | basic monthly | 9.90   | 2020-08-08 |
| 2           | 0       | trial         | 0.00   | 2020-09-20 |
| 2           | 3       | pro annual    | 199.00 | 2020-09-27 |
| 11          | 0       | trial         | 0.00   | 2020-11-19 |
| 11          | 4       | churn         |        | 2020-11-26 |
| 13          | 0       | trial         | 0.00   | 2020-12-15 |
| 13          | 1       | basic monthly | 9.90   | 2020-12-22 |
| 13          | 2       | pro monthly   | 19.90  | 2021-03-29 |
| 15          | 0       | trial         | 0.00   | 2020-03-17 |
| 15          | 2       | pro monthly   | 19.90  | 2020-03-24 |
| 15          | 4       | churn         |        | 2020-04-29 |
| 16          | 0       | trial         | 0.00   | 2020-05-31 |
| 16          | 1       | basic monthly | 9.90   | 2020-06-07 |
| 16          | 3       | pro annual    | 199.00 | 2020-10-21 |
| 18          | 0       | trial         | 0.00   | 2020-07-06 |
| 18          | 2       | pro monthly   | 19.90  | 2020-07-13 |
| 19          | 0       | trial         | 0.00   | 2020-06-22 |
| 19          | 2       | pro monthly   | 19.90  | 2020-06-29 |
| 19          | 3       | pro annual    | 199.00 | 2020-08-29 |

**Answer:**

- Customer 1 started with a trial before signing up for the basic monthly plan after the trial ended.
- Customer 2 started with a trial before signing up for the pro annual plan after the trial ended.
- Customer 11 started with the trial and cancelled their trial before it ended.
- Customer 13 started with the trial before signing up for the basic monthly plan after the trial ended. Then they upgraded to the pro monthly plan several months later.
- Customer 15 started with the trial beore signing up for the pro monthly plan after the trial ended. Then they cancelled their subscrption about a month later.
- Customer 16 started with the trial before signing up for the basic monthly plan after the trial ended. Then they upgraded to the pro annual plan a few months later.
- Customer 18 started with the trial before signing up for the pro monthly plan right before their trial ended.
- Customer 19 started with the trial before signing up for the pro monthly plan after their trial ended. Then they upgraded to the pro annual plan a couple months later.