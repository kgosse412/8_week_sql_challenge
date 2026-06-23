# Data Exploration
## Table of Contents

[1. What day of the week is used for each week_date value?](#1-what-day-of-the-week-is-used-for-each-week_date-value)

[2. What range of week numbers are missing from the dataset?](#2-what-range-of-week-numbers-are-missing-from-the-dataset)

[3. How many total transactions were there for each year in the dataset?](#3-how-many-total-transactions-were-there-for-each-year-in-the-dataset)

[4. What is the total sales for each region for each month?](#4-what-is-the-total-sales-for-each-region-for-each-month)

[5. What is the total count of transactions for each platform?](#5-what-is-the-total-count-of-transactions-for-each-platform)

[6. What is the percentage of sales for Retail vs Shopify for each month?](#6-what-is-the-percentage-of-sales-for-retail-vs-shopify-for-each-month)

[7. What is the percentage of sales by demographic for each year in the dataset?](#7-what-is-the-percentage-of-sales-by-demographic-for-each-year-in-the-dataset)

[8. Which age_band and demographic values contribute the most to Retail sales?](#8-which-age_band-and-demographic-values-contribute-the-most-to-retail-sales)

[9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?](#9-can-we-use-the-avg_transaction-column-to-find-the-average-transaction-size-for-each-year-for-retail-vs-shopify-if-not---how-would-you-calculate-it-instead)

## Questions and Answers
### 1. What day of the week is used for each week_date value?
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

SELECT
DISTINCT TO_CHAR(cws.week_date, 'Day') AS day_of_week
FROM clean_weekly_sales AS cws;	
```

**Table Output:**
| day_of_week |
| ----------- |
| Monday      |

**Answer:**

Monday is used as the first day of the week for each week_date value because EXTRACT('week' FROM <date>) uses ISO 8601. This is also confirmed by converting the week number to a day.

### 2. What range of week numbers are missing from the dataset?
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
),
/* Generate all the week numbers in a year (1 - 52). This will be used to compare the data to the actual week_number in the clean_weekly_sales table. */
generate_weeks AS (
  SELECT
  GENERATE_SERIES(1, 52) AS generated_week
)

SELECT
ROW_NUMBER() OVER (ORDER BY gw.generated_week ASC) AS id
,gw.generated_week AS missing_weeks

FROM generate_weeks AS gw
/* This pulls all the weeks from clean_weekly_sales and ties them to a provided generated_week from generate_weeks table. */
LEFT JOIN clean_weekly_sales AS cws ON cws.week_number = gw.generated_week

WHERE
/* We only care about the numbers where there isn't a week_number in the clean_weekly_sales table. */
cws.week_number IS NULL;
```

**Table Output:**
| id  | missing_weeks |
| --- | ------------- |
| 1   | 1             |
| 2   | 2             |
| 3   | 3             |
| 4   | 4             |
| 5   | 5             |
| 6   | 6             |
| 7   | 7             |
| 8   | 8             |
| 9   | 9             |
| 10  | 10            |
| 11  | 11            |
| 12  | 12            |
| 13  | 37            |
| 14  | 38            |
| 15  | 39            |
| 16  | 40            |
| 17  | 41            |
| 18  | 42            |
| 19  | 43            |
| 20  | 44            |
| 21  | 45            |
| 22  | 46            |
| 23  | 47            |
| 24  | 48            |
| 25  | 49            |
| 26  | 50            |
| 27  | 51            |
| 28  | 52            |

**Answer:**

There are 28 missing weeks in the clean_weekly_sales dataset.

### 3. How many total transactions were there for each year in the dataset?
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

SELECT
cws.calendar_year
,SUM(cws.transactions) AS total_transactions

FROM clean_weekly_sales AS cws

GROUP BY cws.calendar_year

ORDER BY cws.calendar_year ASC	
```

**Table Output:**
| calendar_year | total_transactions |
| ------------- | ------------------ |
| 2018          | 346406460          |
| 2019          | 365639285          |
| 2020          | 375813651          |

**Answer:**

See table for answer.

### 4. What is the total sales for each region for each month?
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

SELECT
cws.month_number
,cws.region
,SUM(cws.sales) AS total_sales

FROM clean_weekly_sales AS cws

GROUP BY cws.month_number, cws.region

ORDER BY cws.month_number ASC, cws.region ASC	
```

**Table Output:**
| month_number | region        | total_sales |
| ------------ | ------------- | ----------- |
| 3            | AFRICA        | 567767480   |
| 3            | ASIA          | 529770793   |
| 3            | CANADA        | 144634329   |
| 3            | EUROPE        | 35337093    |
| 3            | OCEANIA       | 783282888   |
| 3            | SOUTH AMERICA | 71023109    |
| 3            | USA           | 225353043   |
| 4            | AFRICA        | 1911783504  |
| 4            | ASIA          | 1804628707  |
| 4            | CANADA        | 484552594   |
| 4            | EUROPE        | 127334255   |
| 4            | OCEANIA       | 2599767620  |
| 4            | SOUTH AMERICA | 238451531   |
| 4            | USA           | 759786323   |
| 5            | AFRICA        | 1647244738  |
| 5            | ASIA          | 1526285399  |
| 5            | CANADA        | 412378365   |
| 5            | EUROPE        | 109338389   |
| 5            | OCEANIA       | 2215657304  |
| 5            | SOUTH AMERICA | 201391809   |
| 5            | USA           | 655967121   |
| 6            | AFRICA        | 1767559760  |
| 6            | ASIA          | 1619482889  |
| 6            | CANADA        | 443846698   |
| 6            | EUROPE        | 122813826   |
| 6            | OCEANIA       | 2371884744  |
| 6            | SOUTH AMERICA | 218247455   |
| 6            | USA           | 703878990   |
| 7            | AFRICA        | 1960219710  |
| 7            | ASIA          | 1768844756  |
| 7            | CANADA        | 477134947   |
| 7            | EUROPE        | 136757466   |
| 7            | OCEANIA       | 2563459400  |
| 7            | SOUTH AMERICA | 235582776   |
| 7            | USA           | 760331754   |
| 8            | AFRICA        | 1809596890  |
| 8            | ASIA          | 1663320609  |
| 8            | CANADA        | 447073019   |
| 8            | EUROPE        | 122102995   |
| 8            | OCEANIA       | 2432313652  |
| 8            | SOUTH AMERICA | 221166052   |
| 8            | USA           | 712002790   |
| 9            | AFRICA        | 276320987   |
| 9            | ASIA          | 252836807   |
| 9            | CANADA        | 69067959    |
| 9            | EUROPE        | 18877433    |
| 9            | OCEANIA       | 372465518   |
| 9            | SOUTH AMERICA | 34175583    |
| 9            | USA           | 110532368   |

**Answer:**

See table for answer.

### 5. What is the total count of transactions for each platform?
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

SELECT
cws.platform
,SUM(cws.transactions) AS total_transactions

FROM clean_weekly_sales AS cws

GROUP BY cws.platform

ORDER BY cws.platform ASC
```

**Table Output:**
| platform | total_transactions |
| -------- | ------------------ |
| Retail   | 1081934227         |
| Shopify  | 5925169            |

**Answer:**

Retail has 1,081,934,227 total transactions and Shopify has 5,925,169.

### 6. What is the percentage of sales for Retail vs Shopify for each month?
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
),
sales_sum AS (
  SELECT
  cws.platform
  ,cws.month_number
  ,cws.calendar_year
  ,SUM(cws.sales) AS total_sales
  
  FROM clean_weekly_sales AS cws
  
  GROUP BY cws.platform, cws.month_number, cws.calendar_year
  
  ORDER BY cws.calendar_year ASC, cws.month_number ASC, cws.platform ASC
)

SELECT
ss.calendar_year
,ss.month_number
/* 1. Get the sales of the designated platform
** 2. Divide by the sum of the total_sales of the two platforms for that month and year
** 3. Multiply by 100 to turn it into a percentage
** 4. Round to the nearest 2 decimal places
*/
,ROUND(
  MAX(CASE
    WHEN ss.platform = 'Retail' THEN ss.total_sales
    ELSE 0
  END) / SUM(ss.total_sales) * 100, 2) AS retail_percentage
,ROUND(
  MAX(CASE
    WHEN ss.platform = 'Shopify' THEN ss.total_sales
    ELSE 0
  END) / SUM(ss.total_sales) * 100, 2) AS shopify_percentage

FROM sales_sum AS ss

GROUP BY ss.calendar_year, ss.month_number

ORDER BY ss.calendar_year ASC, ss.month_number ASC;
```

**Table Output:**
| calendar_year | month_number | retail_percentage | shopify_percentage |
| ------------- | ------------ | ----------------- | ------------------ |
| 2018          | 3            | 97.92             | 2.08               |
| 2018          | 4            | 97.93             | 2.07               |
| 2018          | 5            | 97.73             | 2.27               |
| 2018          | 6            | 97.76             | 2.24               |
| 2018          | 7            | 97.75             | 2.25               |
| 2018          | 8            | 97.71             | 2.29               |
| 2018          | 9            | 97.68             | 2.32               |
| 2019          | 3            | 97.71             | 2.29               |
| 2019          | 4            | 97.80             | 2.20               |
| 2019          | 5            | 97.52             | 2.48               |
| 2019          | 6            | 97.42             | 2.58               |
| 2019          | 7            | 97.35             | 2.65               |
| 2019          | 8            | 97.21             | 2.79               |
| 2019          | 9            | 97.09             | 2.91               |
| 2020          | 3            | 97.30             | 2.70               |
| 2020          | 4            | 96.96             | 3.04               |
| 2020          | 5            | 96.71             | 3.29               |
| 2020          | 6            | 96.80             | 3.20               |
| 2020          | 7            | 96.67             | 3.33               |
| 2020          | 8            | 96.51             | 3.49               |

**Answer:**

See table for answer.

### 7. What is the percentage of sales by demographic for each year in the dataset?
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
),
sales_sum AS (
  SELECT
  cws.demographic
  ,cws.calendar_year
  ,SUM(cws.sales) AS total_sales
  
  FROM clean_weekly_sales AS cws
  
  GROUP BY cws.demographic, cws.calendar_year
  
  ORDER BY cws.calendar_year ASC, cws.demographic ASC
)

SELECT
ss.calendar_year
/* 1. Get the sales of the designated demographic
** 2. Divide by the sum of the total_sales of the three demographics by year
** 3. Multiply by 100 to turn it into a percentage
** 4. Round to the nearest 2 decimal places
*/
,ROUND(
  MAX(CASE
    WHEN ss.demographic = 'Couples' THEN ss.total_sales
    ELSE 0
  END) / SUM(ss.total_sales) * 100, 2) AS couples_percentage
,ROUND(
  MAX(CASE
    WHEN ss.demographic = 'Families' THEN ss.total_sales
    ELSE 0
  END) / SUM(ss.total_sales) * 100, 2) AS families_percentage
,ROUND(
  MAX(CASE
    WHEN ss.demographic = 'unknown' THEN ss.total_sales
    ELSE 0
  END) / SUM(ss.total_sales) * 100, 2) AS unknown_percentage

FROM sales_sum AS ss

GROUP BY ss.calendar_year

ORDER BY ss.calendar_year ASC;	
```

**Table Output:**
| calendar_year | couples_percentage | families_percentage | unknown_percentage |
| ------------- | ------------------ | ------------------- | ------------------ |
| 2018          | 26.38              | 31.99               | 41.63              |
| 2019          | 27.28              | 32.47               | 40.25              |
| 2020          | 28.72              | 32.73               | 38.55              |

**Answer:**

See table for answer.

### 8. Which age_band and demographic values contribute the most to Retail sales?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql	
```

**Table Output:**

**Answer:**

### 9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql	
```

**Table Output:**

**Answer:**