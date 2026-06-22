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
```

**Table Output:**

**Answer:**

### 5. What is the total count of transactions for each platform?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql	
```

**Table Output:**

**Answer:**

### 6. What is the percentage of sales for Retail vs Shopify for each month?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql	
```

**Table Output:**

**Answer:**

### 7. What is the percentage of sales by demographic for each year in the dataset?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql	
```

**Table Output:**

**Answer:**

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