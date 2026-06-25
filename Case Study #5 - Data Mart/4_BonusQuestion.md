# Bonus Question

## Questions and Answers
### 1. Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period:
### --> region
### --> platform
### --> age_band
### --> demographic
### --> customer_type
### Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?
___________________________________________________________________________________________________________________________
#### ~~ Region Analysis ~~
 
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
  ,cws.calendar_year
  ,cws.region
  ,cws.platform
  ,cws.age_band
  ,cws.demographic
  ,cws.customer_type
  /* Get the total sales by each week_date. */
  ,SUM(cws.sales) AS total_sales

  FROM clean_weekly_sales AS cws

  WHERE
  /* Filter week_date so it is 12 weeks before the given date and 12 weeks after. */
  (cws.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND
  cws.week_date < ('2020-06-15'::date + INTERVAL '12 weeks'))
  
  GROUP BY cws.week_date, cws.week_number, cws.calendar_year, cws.region, cws.platform, cws.age_band, cws.demographic, cws.customer_type

  ORDER BY cws.week_date ASC
)
,before_and_after_pkg_sales AS (
  SELECT
  ps.region
  /* Add the total sales we calculated previously together IF we're looking at the previous 12 weeks. This way we can get the before package sales amount. */
  ,SUM(
    CASE
      WHEN (ps.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND ps.week_date < '2020-06-15'::date)
      THEN ps.total_sales
      ELSE 0
  END) AS before_pkg_sales
  /* Add the total sales we calculated previously together IF we're looking at the after 12 weeks. This way we can get the after package sales amount. */
  ,SUM(
    CASE
      WHEN (ps.week_date >= '2020-06-15'::date AND ps.week_date < ('2020-06-15'::date + INTERVAL '12 weeks'))
      THEN ps.total_sales
      ELSE 0
  END) AS after_pkg_sales

  FROM pkg_sales AS ps

  GROUP BY ps.region
)

SELECT
pkgs.region
/* To get the value growth or reduction, we just subtract the before value from the after value. */
,pkgs.after_pkg_sales - pkgs.before_pkg_sales AS value_rate
/* To get the percentage growth or reduction, we want to take the value_rate (see above) and divide by the before package sales value. Multiply by 100 to get a percentage and round to 2 decimal places for cleanness. */
,ROUND(
  (pkgs.after_pkg_sales - pkgs.before_pkg_sales) / pkgs.before_pkg_sales * 100,
  2) AS percentage_rate

FROM before_and_after_pkg_sales AS pkgs

ORDER BY percentage_rate ASC;
```

**Table Output:**
| region        | value_rate | percentage_rate |
| ------------- | ---------- | --------------- |
| ASIA          | -53436845  | -3.26           |
| OCEANIA       | -71321100  | -3.03           |
| SOUTH AMERICA | -4584174   | -2.15           |
| CANADA        | -8174013   | -1.92           |
| USA           | -10814843  | -1.60           |
| AFRICA        | -9146811   | -0.54           |
| EUROPE        | 5152392    | 4.73            |

#### ~~ Platform Analysis ~~

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
  ,cws.calendar_year
  ,cws.region
  ,cws.platform
  ,cws.age_band
  ,cws.demographic
  ,cws.customer_type
  /* Get the total sales by each week_date. */
  ,SUM(cws.sales) AS total_sales

  FROM clean_weekly_sales AS cws

  WHERE
  /* Filter week_date so it is 12 weeks before the given date and 12 weeks after. */
  (cws.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND
  cws.week_date < ('2020-06-15'::date + INTERVAL '12 weeks'))
  
  GROUP BY cws.week_date, cws.week_number, cws.calendar_year, cws.region, cws.platform, cws.age_band, cws.demographic, cws.customer_type

  ORDER BY cws.week_date ASC
)
,before_and_after_pkg_sales AS (
  SELECT
  ps.platform
  /* Add the total sales we calculated previously together IF we're looking at the previous 12 weeks. This way we can get the before package sales amount. */
  ,SUM(
    CASE
      WHEN (ps.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND ps.week_date < '2020-06-15'::date)
      THEN ps.total_sales
      ELSE 0
  END) AS before_pkg_sales
  /* Add the total sales we calculated previously together IF we're looking at the after 12 weeks. This way we can get the after package sales amount. */
  ,SUM(
    CASE
      WHEN (ps.week_date >= '2020-06-15'::date AND ps.week_date < ('2020-06-15'::date + INTERVAL '12 weeks'))
      THEN ps.total_sales
      ELSE 0
  END) AS after_pkg_sales

  FROM pkg_sales AS ps

  GROUP BY ps.platform
)

SELECT
pkgs.platform
/* To get the value growth or reduction, we just subtract the before value from the after value. */
,pkgs.after_pkg_sales - pkgs.before_pkg_sales AS value_rate
/* To get the percentage growth or reduction, we want to take the value_rate (see above) and divide by the before package sales value. Multiply by 100 to get a percentage and round to 2 decimal places for cleanness. */
,ROUND(
  (pkgs.after_pkg_sales - pkgs.before_pkg_sales) / pkgs.before_pkg_sales * 100,
  2) AS percentage_rate

FROM before_and_after_pkg_sales AS pkgs

ORDER BY percentage_rate ASC;
```

**Table Output:**
| platform | value_rate | percentage_rate |
| -------- | ---------- | --------------- |
| Retail   | -168083834 | -2.43           |
| Shopify  | 15758440   | 7.18            |

#### ~~ Age Band Analysis ~~

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
  ,cws.calendar_year
  ,cws.region
  ,cws.platform
  ,cws.age_band
  ,cws.demographic
  ,cws.customer_type
  /* Get the total sales by each week_date. */
  ,SUM(cws.sales) AS total_sales

  FROM clean_weekly_sales AS cws

  WHERE
  /* Filter week_date so it is 12 weeks before the given date and 12 weeks after. */
  (cws.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND
  cws.week_date < ('2020-06-15'::date + INTERVAL '12 weeks'))
  
  GROUP BY cws.week_date, cws.week_number, cws.calendar_year, cws.region, cws.platform, cws.age_band, cws.demographic, cws.customer_type

  ORDER BY cws.week_date ASC
)
,before_and_after_pkg_sales AS (
  SELECT
  ps.age_band
  /* Add the total sales we calculated previously together IF we're looking at the previous 12 weeks. This way we can get the before package sales amount. */
  ,SUM(
    CASE
      WHEN (ps.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND ps.week_date < '2020-06-15'::date)
      THEN ps.total_sales
      ELSE 0
  END) AS before_pkg_sales
  /* Add the total sales we calculated previously together IF we're looking at the after 12 weeks. This way we can get the after package sales amount. */
  ,SUM(
    CASE
      WHEN (ps.week_date >= '2020-06-15'::date AND ps.week_date < ('2020-06-15'::date + INTERVAL '12 weeks'))
      THEN ps.total_sales
      ELSE 0
  END) AS after_pkg_sales

  FROM pkg_sales AS ps

  GROUP BY ps.age_band
)

SELECT
pkgs.age_band
/* To get the value growth or reduction, we just subtract the before value from the after value. */
,pkgs.after_pkg_sales - pkgs.before_pkg_sales AS value_rate
/* To get the percentage growth or reduction, we want to take the value_rate (see above) and divide by the before package sales value. Multiply by 100 to get a percentage and round to 2 decimal places for cleanness. */
,ROUND(
  (pkgs.after_pkg_sales - pkgs.before_pkg_sales) / pkgs.before_pkg_sales * 100,
  2) AS percentage_rate

FROM before_and_after_pkg_sales AS pkgs

ORDER BY percentage_rate ASC;
```

**Table Output:**
| age_band     | value_rate | percentage_rate |
| ------------ | ---------- | --------------- |
| unknown      | -92393021  | -3.34           |
| Middle Aged  | -22994292  | -1.97           |
| Retirees     | -29549521  | -1.23           |
| Young Adults | -7388560   | -0.92           |

#### ~~ Demographic Analysis ~~

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
  ,cws.calendar_year
  ,cws.region
  ,cws.platform
  ,cws.age_band
  ,cws.demographic
  ,cws.customer_type
  /* Get the total sales by each week_date. */
  ,SUM(cws.sales) AS total_sales

  FROM clean_weekly_sales AS cws

  WHERE
  /* Filter week_date so it is 12 weeks before the given date and 12 weeks after. */
  (cws.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND
  cws.week_date < ('2020-06-15'::date + INTERVAL '12 weeks'))
  
  GROUP BY cws.week_date, cws.week_number, cws.calendar_year, cws.region, cws.platform, cws.age_band, cws.demographic, cws.customer_type

  ORDER BY cws.week_date ASC
)
,before_and_after_pkg_sales AS (
  SELECT
  ps.demographic
  /* Add the total sales we calculated previously together IF we're looking at the previous 12 weeks. This way we can get the before package sales amount. */
  ,SUM(
    CASE
      WHEN (ps.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND ps.week_date < '2020-06-15'::date)
      THEN ps.total_sales
      ELSE 0
  END) AS before_pkg_sales
  /* Add the total sales we calculated previously together IF we're looking at the after 12 weeks. This way we can get the after package sales amount. */
  ,SUM(
    CASE
      WHEN (ps.week_date >= '2020-06-15'::date AND ps.week_date < ('2020-06-15'::date + INTERVAL '12 weeks'))
      THEN ps.total_sales
      ELSE 0
  END) AS after_pkg_sales

  FROM pkg_sales AS ps

  GROUP BY ps.demographic
)

SELECT
pkgs.demographic
/* To get the value growth or reduction, we just subtract the before value from the after value. */
,pkgs.after_pkg_sales - pkgs.before_pkg_sales AS value_rate
/* To get the percentage growth or reduction, we want to take the value_rate (see above) and divide by the before package sales value. Multiply by 100 to get a percentage and round to 2 decimal places for cleanness. */
,ROUND(
  (pkgs.after_pkg_sales - pkgs.before_pkg_sales) / pkgs.before_pkg_sales * 100,
  2) AS percentage_rate

FROM before_and_after_pkg_sales AS pkgs

ORDER BY percentage_rate ASC;
```

**Table Output:**
| demographic | value_rate | percentage_rate |
| ----------- | ---------- | --------------- |
| unknown     | -92393021  | -3.34           |
| Families    | -42320015  | -1.82           |
| Couples     | -17612358  | -0.87           |

#### ~~ Customer Type Analysis ~~

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
  ,cws.calendar_year
  ,cws.region
  ,cws.platform
  ,cws.age_band
  ,cws.demographic
  ,cws.customer_type
  /* Get the total sales by each week_date. */
  ,SUM(cws.sales) AS total_sales

  FROM clean_weekly_sales AS cws

  WHERE
  /* Filter week_date so it is 12 weeks before the given date and 12 weeks after. */
  (cws.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND
  cws.week_date < ('2020-06-15'::date + INTERVAL '12 weeks'))
  
  GROUP BY cws.week_date, cws.week_number, cws.calendar_year, cws.region, cws.platform, cws.age_band, cws.demographic, cws.customer_type

  ORDER BY cws.week_date ASC
)
,before_and_after_pkg_sales AS (
  SELECT
  ps.customer_type
  /* Add the total sales we calculated previously together IF we're looking at the previous 12 weeks. This way we can get the before package sales amount. */
  ,SUM(
    CASE
      WHEN (ps.week_date >= ('2020-06-15'::date - INTERVAL '12 weeks') AND ps.week_date < '2020-06-15'::date)
      THEN ps.total_sales
      ELSE 0
  END) AS before_pkg_sales
  /* Add the total sales we calculated previously together IF we're looking at the after 12 weeks. This way we can get the after package sales amount. */
  ,SUM(
    CASE
      WHEN (ps.week_date >= '2020-06-15'::date AND ps.week_date < ('2020-06-15'::date + INTERVAL '12 weeks'))
      THEN ps.total_sales
      ELSE 0
  END) AS after_pkg_sales

  FROM pkg_sales AS ps

  GROUP BY ps.customer_type
)

SELECT
pkgs.customer_type
/* To get the value growth or reduction, we just subtract the before value from the after value. */
,pkgs.after_pkg_sales - pkgs.before_pkg_sales AS value_rate
/* To get the percentage growth or reduction, we want to take the value_rate (see above) and divide by the before package sales value. Multiply by 100 to get a percentage and round to 2 decimal places for cleanness. */
,ROUND(
  (pkgs.after_pkg_sales - pkgs.before_pkg_sales) / pkgs.before_pkg_sales * 100,
  2) AS percentage_rate

FROM before_and_after_pkg_sales AS pkgs

ORDER BY percentage_rate ASC;
```

**Table Output:**
| customer_type | value_rate | percentage_rate |
| ------------- | ---------- | --------------- |
| Guest         | -77202666  | -3.00           |
| Existing      | -83872973  | -2.27           |
| New           | 8750245    | 1.01            |

#### ~~ Analysis ~~

It appears that Guests have the highest negative impact on sales at a percentage rate of -3.0%, which is also showcased by the output of age_band and demographic, where both show uknown at -3.34%. To mitigate this, perhaps Danny should have an incentive to sign up for an account when purchasing items from the store.

The data also shows that Retail has the highest negative impact on sales at a percentage rate of -2.43%. This could be due to people being impacted by the sustainable package changes more in person than when they order online but it's hard to say without knowing what sustainable package changes have been made (i.e. Did they get rid of plastic bagging? Is it a change in actual packaging of products? Etc.).

Finally, we see that Asia has the highest negative impact on sales at a percentage rate of -3.26%. This could be due to the culture in Asia. Again, it's hard to say without knowing the sustainable package changes were.

I believe we can evaulate this further if we understood more of what is meant by "sustainable package changes." For example, if the change was that plastic bagging was removed from retail stores, then we can address the issue with certain incentives like giving a discount when bringing your own bags or using paper bags instead of plastic.