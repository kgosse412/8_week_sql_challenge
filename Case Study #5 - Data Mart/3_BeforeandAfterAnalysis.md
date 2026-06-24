# Before and After Analysis
## Table of Contents

[1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?](#1-what-is-the-total-sales-for-the-4-weeks-before-and-after-2020-06-15-what-is-the-growth-or-reduction-rate-in-actual-values-and-percentage-of-sales)

[2. What about the entire 12 weeks before and after?](#2-what-about-the-entire-12-weeks-before-and-after)

[3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?](#3-how-do-the-sale-metrics-for-these-2-periods-before-and-after-compare-with-the-previous-years-in-2018-and-2019)

## Questions and Answers
### 1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
/* Fix the date so it has a full 4 digit year for later conversion */
WITH date_fix AS (
  SELECT
  ws.*
  /* Fix the year to be a full 4 digits before converting to date format*/
  ,TO_DATE(LEFT(ws.week_date, LENGTH(ws.week_date) - 2) || '20' || RIGHT(ws.week_date, 2), 'DD-MM-YYYY') AS wk_date
  
  FROM data_mart.weekly_sales AS ws
),
clean_weekly_sales AS (
  SELECT
  df.wk_date AS week_date
  ,EXTRACT('week' FROM df.wk_date) AS week_number
  ,EXTRACT('month' FROM df.wk_date) AS month_number
  ,EXTRACT('year' FROM df.wk_date) AS calendar_year
  ,df.region
  ,df.platform
  /* Change the null value in segment to unknown */
  ,CASE
  	WHEN df.segment = 'null' THEN 'unknown'
  	ELSE df.segment
  END AS segment
  /* Create a new column from segment called age_band. Based only on the number from the segment column. */
  ,CASE
  	WHEN RIGHT(df.segment, 1) = '1' THEN 'Young Adults'
    WHEN RIGHT(df.segment, 1) = '2' THEN 'Middle Aged'
    WHEN RIGHT(df.segment, 1) = '3' OR RIGHT(df.segment, 1) = '4' THEN 'Retirees'
  	ELSE 'unknown'
  END AS age_band
  /* Create a new column from segment called demographic. Based only on the letter from the segment column. */
  ,CASE
  	WHEN LEFT(df.segment, 1) = 'C' THEN 'Couples'
    WHEN LEFT(df.segment, 1) = 'F' THEN 'Families'
  	ELSE 'unknown'
  END AS demographic
  ,df.customer_type
  ,df.transactions
  ,df.sales
  /* Create new column called avg_transactions using the sales and transactions columns. Round to 2 decimal places. */
  ,ROUND(df.sales::numeric / df.transactions, 2) AS avg_transaction
    
  FROM date_fix AS df
)
,pkg_sales AS (
  SELECT
  cws.week_date
  ,cws.week_number
  /* Get the total sales by each week_date. */
  ,SUM(cws.sales) AS total_sales

  FROM clean_weekly_sales AS cws

  WHERE
  /* Filter week_date so it is 4 weeks before the given date and 4 weeks after. Remember that the given date is part of the after, so we only need an additional 3 weeks of data after the given date. */
  cws.week_date >= ('2020-06-15'::date - INTERVAL '4 weeks') AND
  cws.week_date <= ('2020-06-15'::date + INTERVAL '3 weeks')

  GROUP BY cws.week_date, cws.week_number

  ORDER BY cws.week_date ASC
)
,before_and_after_pkg_sales AS (
  SELECT
  /* Add the total sales we calculated previously together IF we're looking at the previous 4 weeks. This way we can get the before package sales amount. */
  SUM(
    CASE
      WHEN ps.week_date >= ('2020-06-15'::date - INTERVAL '4 weeks') AND ps.week_date < '2020-06-15'::date THEN ps.total_sales
      ELSE 0
  END) AS before_pkg_sales
  /* Add the total sales we calculated previously together IF we're looking at the after 4 weeks. This way we can get the after package sales amount. */
  ,SUM(
    CASE
      WHEN ps.week_date >= '2020-06-15'::date AND ps.week_date <= ('2020-06-15'::date + INTERVAL '3 weeks') THEN ps.total_sales
      ELSE 0
  END) AS after_pkg_sales

  FROM pkg_sales AS ps
)

SELECT
pkgs.*
/* To get the value growth or reduction, we just subtract the before value from the after value. */
,pkgs.after_pkg_sales - pkgs.before_pkg_sales AS value_rate
/* To get the percentage growth or reduction, we want to take the value_rate (see above) and divide by the before package sales value. Multiply by 100 to get a percentage and round to 2 decimal places for cleanness. */
,ROUND(
  (pkgs.after_pkg_sales - pkgs.before_pkg_sales) / pkgs.before_pkg_sales * 100,
  2) AS percentage_rate

FROM before_and_after_pkg_sales AS pkgs;
```

**Table Output:**
| before_pkg_sales | after_pkg_sales | value_rate | percentage_rate |
| ---------------- | --------------- | ---------- | --------------- |
| 2345878357       | 2318994169      | -26884188  | -1.15           |

**Answer:**

There is a reduction rate of -26884188 with a percentage of -1.15%.

### 2. What about the entire 12 weeks before and after?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
/* Fix the date so it has a full 4 digit year for later conversion */
WITH date_fix AS (
  SELECT
  ws.*
  /* Fix the year to be a full 4 digits before converting to date format*/
  ,TO_DATE(LEFT(ws.week_date, LENGTH(ws.week_date) - 2) || '20' || RIGHT(ws.week_date, 2), 'DD-MM-YYYY') AS wk_date
  
  FROM data_mart.weekly_sales AS ws
),
clean_weekly_sales AS (
  SELECT
  df.wk_date AS week_date
  ,EXTRACT('week' FROM df.wk_date) AS week_number
  ,EXTRACT('month' FROM df.wk_date) AS month_number
  ,EXTRACT('year' FROM df.wk_date) AS calendar_year
  ,df.region
  ,df.platform
  /* Change the null value in segment to unknown */
  ,CASE
  	WHEN df.segment = 'null' THEN 'unknown'
  	ELSE df.segment
  END AS segment
  /* Create a new column from segment called age_band. Based only on the number from the segment column. */
  ,CASE
  	WHEN RIGHT(df.segment, 1) = '1' THEN 'Young Adults'
    WHEN RIGHT(df.segment, 1) = '2' THEN 'Middle Aged'
    WHEN RIGHT(df.segment, 1) = '3' OR RIGHT(df.segment, 1) = '4' THEN 'Retirees'
  	ELSE 'unknown'
  END AS age_band
  /* Create a new column from segment called demographic. Based only on the letter from the segment column. */
  ,CASE
  	WHEN LEFT(df.segment, 1) = 'C' THEN 'Couples'
    WHEN LEFT(df.segment, 1) = 'F' THEN 'Families'
  	ELSE 'unknown'
  END AS demographic
  ,df.customer_type
  ,df.transactions
  ,df.sales
  /* Create new column called avg_transactions using the sales and transactions columns. Round to 2 decimal places. */
  ,ROUND(df.sales::numeric / df.transactions, 2) AS avg_transaction
    
  FROM date_fix AS df
)
,pkg_sales AS (
  SELECT
  cws.week_date
  ,cws.week_number
  /* Get the total sales by each week_date. */
  ,SUM(cws.sales) AS total_sales

  FROM clean_weekly_sales AS cws

  WHERE
  /* Filter week_date so it is 12 weeks before the given date and 12 weeks after. Remember that the given date is part of the after, so we only need an additional 11 weeks of data after the given date. */
  cws.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND
  cws.week_date <= ('2020-06-15'::date + INTERVAL '11 weeks')

  GROUP BY cws.week_date, cws.week_number

  ORDER BY cws.week_date ASC
)
,before_and_after_pkg_sales AS (
  SELECT
  /* Add the total sales we calculated previously together IF we're looking at the previous 4 weeks. This way we can get the before package sales amount. */
  SUM(
    CASE
      WHEN ps.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND ps.week_date < '2020-06-15'::date THEN ps.total_sales
      ELSE 0
  END) AS before_pkg_sales
  /* Add the total sales we calculated previously together IF we're looking at the after 4 weeks. This way we can get the after package sales amount. */
  ,SUM(
    CASE
      WHEN ps.week_date >= '2020-06-15'::date AND ps.week_date <= ('2020-06-15'::date + INTERVAL '11 weeks') THEN ps.total_sales
      ELSE 0
  END) AS after_pkg_sales

  FROM pkg_sales AS ps
)

SELECT
pkgs.*
/* To get the value growth or reduction, we just subtract the before value from the after value. */
,pkgs.after_pkg_sales - pkgs.before_pkg_sales AS value_rate
/* To get the percentage growth or reduction, we want to take the value_rate (see above) and divide by the before package sales value. Multiply by 100 to get a percentage and round to 2 decimal places for cleanness. */
,ROUND(
  (pkgs.after_pkg_sales - pkgs.before_pkg_sales) / pkgs.before_pkg_sales * 100,
  2) AS percentage_rate

FROM before_and_after_pkg_sales AS pkgs;	
```

**Table Output:**
| before_pkg_sales | after_pkg_sales | value_rate | percentage_rate |
| ---------------- | --------------- | ---------- | --------------- |
| 7126273147       | 6973947753      | -152325394 | -2.14           |

**Answer:**

There is a reduction rate of -152325394 with a percentage rate of -2.14%. It appears that sales are in decline overall based on this and the previous questions data.

### 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql	
```

**Table Output:**

**Answer:**