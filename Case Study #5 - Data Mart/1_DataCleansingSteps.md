# Data Cleansing Steps
## Table of Contents

[Data Cleansing Query](data-cleansings-query)

## Data Cleansing Query
**SQL Statement:**
	
```sql
/* Fix the date so it has a full 4 digit year for later conversion and is also a date formart. */
WITH date_fix AS (
  SELECT
  ws.*
  /* Fix the year to be a full 4 digits before converting to date format. */
  ,TO_DATE(LEFT(ws.week_date, LENGTH(ws.week_date) - 2) || '20' || RIGHT(ws.week_date, 2), 'DD-MM-YYYY') AS wk_date
  
  
  FROM data_mart.weekly_sales AS ws
),
/* Fix the rest of the data per instructions */
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

SELECT *
FROM clean_weekly_sales
LIMIT 10;
```

**Table Output:**
| week_date  | week_number | month_number | calendar_year | region | platform | segment | age_band     | demographic | customer_type | transactions | sales    | avg_transaction |
| ---------- | ----------- | ------------ | ------------- | ------ | -------- | ------- | ------------ | ----------- | ------------- | ------------ | -------- | --------------- |
| 2020-08-31 | 36          | 8            | 2020          | ASIA   | Retail   | C3      | Retirees     | Couples     | New           | 120631       | 3656163  | 30.31           |
| 2020-08-31 | 36          | 8            | 2020          | ASIA   | Retail   | F1      | Young Adults | Families    | New           | 31574        | 996575   | 31.56           |
| 2020-08-31 | 36          | 8            | 2020          | USA    | Retail   | unknown | unknown      | unknown     | Guest         | 529151       | 16509610 | 31.20           |
| 2020-08-31 | 36          | 8            | 2020          | EUROPE | Retail   | C1      | Young Adults | Couples     | New           | 4517         | 141942   | 31.42           |
| 2020-08-31 | 36          | 8            | 2020          | AFRICA | Retail   | C2      | Middle Aged  | Couples     | New           | 58046        | 1758388  | 30.29           |
| 2020-08-31 | 36          | 8            | 2020          | CANADA | Shopify  | F2      | Middle Aged  | Families    | Existing      | 1336         | 243878   | 182.54          |
| 2020-08-31 | 36          | 8            | 2020          | AFRICA | Shopify  | F3      | Retirees     | Families    | Existing      | 2514         | 519502   | 206.64          |
| 2020-08-31 | 36          | 8            | 2020          | ASIA   | Shopify  | F1      | Young Adults | Families    | Existing      | 2158         | 371417   | 172.11          |
| 2020-08-31 | 36          | 8            | 2020          | AFRICA | Shopify  | F2      | Middle Aged  | Families    | New           | 318          | 49557    | 155.84          |
| 2020-08-31 | 36          | 8            | 2020          | AFRICA | Retail   | C3      | Retirees     | Couples     | New           | 111032       | 3888162  | 35.02           |
