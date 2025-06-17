# Outside the Box Questions
## Table of Contents

[]()

## Questions and Answers
### 1. How would you calculate the rate of growth for Foodie-Fi?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| payment_table_full | Contains information about each month's revenue and the number of customers |

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
cip_cutoff_date AS (SELECT
cip.*
/* Use the LEAD window function to figur out if their is a cutoff date when someone changes their
plan. If they change plans, the cutoff_date is populated. If not, it is left as null. This will
come in handy when we generate our end dates for the payments table. */
,LEAD(cip.start_date) OVER(PARTITION BY cip.customer_id ORDER BY cip.start_date) AS cutoff_date

FROM customer_info_plans AS cip

/* We only want to look at the year 2020 and we don't care about trial plans (since they're
free). */
WHERE
EXTRACT(YEAR FROM cip.start_date) = 2020 AND
cip.plan_name != 'trial'
),
payment_table_temp AS (SELECT
  *
  FROM (SELECT
    cd.customer_id
    ,cd.plan_id
    ,cd.plan_name
    /* Generate a series of dates in 1 month intervals that starts at the start_date and
    ends with either the cutoff_date OR 2020-12-31 (depending on which comes first). */
    ,GENERATE_SERIES(
      cd.start_date,
      /* Grab which date comes first. If cutoff_date is null, then the end date will be
      2020-12-31. If cutoff_date is NOT null, then use cutoff_date. */
      COALESCE(cd.cutoff_date - 1, '2020-12-31'::date),
      '1 month'::interval
    ) AS payment_date
    ,cd.price AS amount

    FROM cip_cutoff_date AS cd

    /* We don't want to generate a series of dates for anything that is churn or
    pro annual. */
    WHERE
    cd.plan_name != 'churn' AND
    cd.plan_name != 'pro annual'

    /* Join the below query, which does look at pro annual plans, to the above query, which
    doesn't look at pro annual plans. This way we aren't generating a montlhy series for
    pro annual plans. */
    UNION ALL

    SELECT
    cip.customer_id
    ,cip.plan_id
    ,cip.plan_name
    ,cip.start_date AS payment_date
    ,cip.price AS amount

    FROM customer_info_plans AS cip

    /* Only look at pro annual plans. */
    WHERE
    cip.plan_name = 'pro annual'
  ) AS pt

  ORDER BY pt.customer_id, pt.payment_date
),
payment_table_full AS (SELECT
  pt_temp.customer_id
  ,pt_temp.plan_id
  ,pt_temp.plan_name
  ,pt_temp.payment_date
  /* Determine when to subtract off the 9.90 from the pro monthly or pro annual plans. */
  ,CASE
      WHEN (
        /* IF the previous plan_name is basic monthly */
        LAG(pt_temp.plan_name) OVER(PARTITION BY pt_temp.customer_id ORDER BY pt_temp.payment_date) = 'basic monthly'
        /* AND the current plan is either a pro monthly or pro annual plan */
        AND pt_temp.plan_name IN ('pro monthly','pro annual')
        /* AND the previous payment_date + 1 month is greater than the current payment_date, meaning
        the plan was upgraded in the middle of the billing cycle */
        AND LAG(pt_temp.payment_date) OVER(PARTITION BY pt_temp.customer_id ORDER BY pt_temp.payment_date) + '1 month'::interval > pt_temp.payment_date
      /* THEN take the current amount and subtract off the basic monthly rate of 9.90 */
      ) THEN pt_temp.amount - 9.90
      /* Otherwise, leave the amount alone */
      ELSE pt_temp.amount
  END AS amount
  /* Use the ROW_NUMBER window function to label each row with a payment_order number grouped
  by the customer_id and ordered by the payment_date. */
  ,ROW_NUMBER() OVER(PARTITION BY pt_temp.customer_id ORDER BY pt_temp.payment_date) AS payment_order

  FROM payment_table_temp AS pt_temp
),
ptf_metrics AS (SELECT
  /* Get the month number from the payment_date */
  EXTRACT(MONTH FROM ptf.payment_date) AS month_num
  /* Convert the payment_date to a month name */
  ,TO_CHAR(ptf.payment_date, 'Month') AS month
  /* Count how many customers there are for each month */
  ,COUNT(ptf.customer_id) AS num_of_customers
  /* Show the previous month's customer count */
  ,LAG(COUNT(ptf.customer_id)) OVER(ORDER BY EXTRACT(MONTH FROM ptf.payment_date)) AS prev_num_of_customers
  /* Find the revenue by adding up the payments for each month */              
  ,SUM(ptf.amount) AS revenue
  /* Show the previous month's revenue */
  ,LAG(SUM(ptf.amount)) OVER(ORDER BY EXTRACT(MONTH FROM ptf.payment_date)) AS prev_revenue

  FROM payment_table_full AS ptf

  GROUP BY month_num, month

  ORDER BY month_num
)

SELECT
ptfm.month_num AS "Month Num"
,ptfm.month AS "Month"
,ptfm.num_of_customers AS "Number of Customers"
/* You find the customer growth by taking the current period's (in our case it's in months) number
of customers, subtracting the previous period's number of customers, then dividing by the previous
period's number of customers before multiplying by 100. I used COALESCE in this case because our first
month has a NULL value for the previous month's num_of_customers, and this will select the current month's
num_of_customers in it's place, giving us a 0% growth rate. */
,ROUND(
  100.0 * 
  (ptfm.num_of_customers - COALESCE(ptfm.prev_num_of_customers, ptfm.num_of_customers)) / 
  COALESCE(ptfm.prev_num_of_customers, ptfm.num_of_customers),
  2
) AS "Customer Growth"
,ptfm.revenue AS "Revenue"
/* You find the revenue growth by taking the current period's (in our case it's in months) revenue,
subtracting the previous period's revenue, then dividing by the previous period's revenue before
multiplying by 100. I used COALESCE in this case because our first month has a NULL value for the
previous month's revenue, and this will select the current month's revenue in it's place, giving us
a 0% growth rate. */
,ROUND(
  100.0 * 
  (ptfm.revenue - COALESCE(ptfm.prev_revenue, ptfm.revenue)) / 
  COALESCE(ptfm.prev_revenue, ptfm.revenue),
  2
) AS "Revenue Growth"

FROM ptf_metrics AS ptfm
```

**Table Output:**

| Month Num | Month     | Number of Customers | Customer Growth | Revenue  | Revenue Growth |
| --------- | --------- | ------------------- | --------------- | -------- | -------------- |
| 1         | January   | 86                  | 0.00            | 5988.70  | 0.00           |
| 2         | February  | 145                 | 68.60           | 6145.80  | 2.62           |
| 3         | March     | 212                 | 46.21           | 5895.40  | -4.07          |
| 4         | April     | 274                 | 29.25           | 8272.20  | 40.32          |
| 5         | May       | 319                 | 16.42           | 6848.20  | -17.21         |
| 6         | June      | 374                 | 17.24           | 8170.40  | 19.31          |
| 7         | July      | 418                 | 11.76           | 9603.00  | 17.53          |
| 8         | August    | 479                 | 14.59           | 11244.20 | 17.09          |
| 9         | September | 513                 | 7.10            | 12218.90 | 8.67           |
| 10        | October   | 548                 | 6.82            | 14050.10 | 14.99          |
| 11        | November  | 555                 | 1.28            | 12049.10 | -14.24         |
| 12        | December  | 588                 | 5.95            | 12635.80 | 4.87           |

**Answer:**

- To show growth, I would showcase the number of customers, growth rate of customers, revenue, and revenue growth per each month.

### 2. What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?
___________________________________________________________________________________________________________________________
**Answer:**

- I would have Foodie-Fi track the following:
	- Customer growth rate
	- Revenue growth rate
	- Number of customers per plan
	- How many customers convert from trial to a paid membership
	- How many customers upgrade or downgrade their plans and why
	- How many customers cancel and why

### 3. What are some key customer journeys or experiences that you would analyse further to improve customer retention?
___________________________________________________________________________________________________________________________
**Answer:**

- Why customers cancel their plan
- Why customers don't start a membership after the trial period
- Why customers change their plan by either upgrading or downgrading
- What is the reason for their plan choice

### 4. If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?
___________________________________________________________________________________________________________________________
**Answer:**

- 

### 5. What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?
___________________________________________________________________________________________________________________________
**Answer:**

- 