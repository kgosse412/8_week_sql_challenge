# Customer Nodes Exploration
## Table of Contents

[1. How many unique nodes are there on the Data Bank system?](#1-how-many-unique-nodes-are-there-on-the-data-bank-system)

[2. What is the number of nodes per region?](#2-what-is-the-number-of-nodes-per-region)

[3. How many customers are allocated to each region?](#3-how-many-customers-are-allocated-to-each-region)

[4. How many days on average are customers reallocated to a different node?](#4-how-many-days-on-average-are-customers-reallocated-to-a-different-node)

[5. What is the median, 80th, and 95th percentile for this same reallocation days metric for each region?](#5-what-is-the-median-80th-and-95th-percentile-for-this-same-reallocation-days-metric-for-each-region)

## Questions and Answers
### 1. How many unique nodes are there on the Data Bank system?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_nodes | Contains information on the nodes in the Data Bank system |

**SQL Statement:**
	
```sql	
SELECT
/* Count the unique number of nodes in the customer_nodes table. */
COUNT(DISTINCT cn.node_id) AS "Number of Unique Nodes"

FROM data_bank.customer_nodes AS cn
```

**Table Output:**

| Number of Unique Nodes |
| ---------------------- |
| 5                      |

**Answer:**

- There are 5 unique nodes in the Data Bank system.

### 2. What is the number of nodes per region?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_nodes | Contains information on the nodes in the Data Bank system |
| regions | Contains information on the region names |

**SQL Statement:**
	
```sql	
SELECT
cn.region_id AS "Region ID"
,r.region_name AS "Region Name" -- pulled from the regions table
/* Count the unique number of nodes in the customer_nodes table. */
,COUNT(DISTINCT cn.node_id) AS "Number of Nodes"

FROM data_bank.customer_nodes AS cn
/* Connect the regions table to our customer_nodes table via the region_id.
This is so we can get the region_name from the regions table and the nodes
from the customer_nodes table. */
JOIN data_bank.regions AS r ON cn.region_id = r.region_id

/* Group our count statement by Region ID and Region Name to see
how many nodes are in each region. */
GROUP BY "Region ID", "Region Name"

/* Sort by Region ID to make the output look nicer. */
ORDER BY "Region ID"
```

**Table Output:**

| Region ID | Region Name | Number of Nodes |
| --------- | ----------- | --------------- |
| 1         | Australia   | 5               |
| 2         | America     | 5               |
| 3         | Africa      | 5               |
| 4         | Asia        | 5               |
| 5         | Europe      | 5               |

**Answer:**

- Each region has 5 nodes.

### 3. How many customers are allocated to each region?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_nodes | Contains information on the customers in the Data Bank system |
| regions | Contains information on the region names |

**SQL Statement:**
	
```sql	
SELECT
cn.region_id AS "Region ID"
,r.region_name AS "Region Name" -- pulled from the regions table
/* Count the unique number of customers in the customer_nodes table. */
,COUNT(DISTINCT cn.customer_id) AS "Number of Customers"

FROM data_bank.customer_nodes AS cn
/* Connect the regions table to our customer_nodes table via the region_id.
This is so we can get the region_name from the regions table and the customers
from the customer_nodes table. */
JOIN data_bank.regions AS r ON cn.region_id = r.region_id

/* Group our count statement by Region ID and Region Name to see
how many customers are in each region. */
GROUP BY "Region ID", "Region Name"

/* Sort by Region ID to make the output look nicer. */
ORDER BY "Region ID"
```

**Table Output:**

| Region ID | Region Name | Number of Customers |
| --------- | ----------- | ------------------- |
| 1         | Australia   | 110                 |
| 2         | America     | 105                 |
| 3         | Africa      | 102                 |
| 4         | Asia        | 95                  |
| 5         | Europe      | 88                  |

**Answer:**

- Australia has 110 customers, America has 105 customers, Africa has 102 customers, Asia has 95 customers, and Europe has 88 customers.

### 4. How many days on average are customers reallocated to a different node?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_nodes | Contains information on the start and end dates for each node allocation |

**SQL Statement:**
	
```sql	
WITH node_days AS (SELECT 
  cn.customer_id
  ,cn.node_id
  /* Add the the number of days between the start_date and the end_date per each customer
  and node. NOTE: EXTRACT requires a TIMESTAMP, but end_date and start_date are of a DATE
  type, so they need to be converted using ::timestamp. */
  ,SUM(EXTRACT(DAY FROM end_date::timestamp - start_date::timestamp)) AS date_diff_sum

  FROM data_bank.customer_nodes AS cn

  WHERE
  /* We only want to look at the data where there is a definitive end_date. If the end_date
  is 9999-12-31, then the customer is still on that node and we can exclude it from our
  calculation. */
  cn.end_date != '9999-12-31'

  GROUP BY cn.customer_id, cn.node_id
)
                   
SELECT
/* Take the average of date_diff_sum and round to the nearest whole number. */
ROUND(AVG(nd.date_diff_sum)) AS "Average Days Spent in Node"

FROM node_days AS nd
```

**Table Output:**

| Average Days Spent in Node |
| -------------------------- |
| 24                         |

**Answer:**

- Customers are reallocated to a new node roughly every 24 days.

### 5. What is the median, 80th, and 95th percentile for this same reallocation days metric for each region?
___________________________________________________________________________________________________________________________
**Overview:**

Tables Used:

| Table | Why |
| ----- | --- |
| customer_nodes | Contains information on the start and end dates for each node allocation |
| regions | Contains information on the region names |

**SQL Statement:**
	
```sql	
WITH node_days AS (SELECT 
  cn.customer_id
  ,cn.node_id
  ,cn.region_id
  ,r.region_name
  /* Add the the number of days between the start_date and the end_date per each customer
  and node. NOTE: EXTRACT requires a TIMESTAMP, but end_date and start_date are of a DATE
  type, so they need to be converted using ::timestamp. */
  ,SUM(EXTRACT(DAY FROM end_date::timestamp - start_date::timestamp)) AS date_diff_sum

  FROM data_bank.customer_nodes AS cn
  JOIN data_bank.regions AS r ON cn.region_id = r.region_id

  WHERE
  /* We only want to look at the data where there is a definitive end_date. If the end_date
  is 9999-12-31, then the customer is still on that node and we can exclude it from our
  calculation. */
  cn.end_date != '9999-12-31'

  GROUP BY cn.customer_id, cn.node_id, cn.region_id, r.region_name
)
                   
SELECT
nd.region_name AS "Region"
/* Take the average of date_diff_sum and round to the nearest whole number. */
,ROUND(AVG(nd.date_diff_sum)) AS "Average Days Spent in Node"
,PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY nd.date_diff_sum) AS "Median Days in Node"
,PERCENTILE_CONT(0.80) WITHIN GROUP (ORDER BY nd.date_diff_sum) AS "80th Percentile Days in Node"
,PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY nd.date_diff_sum) AS "95th Percentile Days in Node"

FROM node_days AS nd

GROUP BY nd.region_name

ORDER BY "Region"
```

**Table Output:**

| Region    | Average Days Spent in Node | Median Days in Node | 80th Percentile Days in Node | 95th Percentile Days in Node |
| --------- | -------------------------- | ------------------- | ---------------------------- | ---------------------------- |
| Africa    | 24                         | 22                  | 35                           | 54                           |
| America   | 24                         | 22                  | 34                           | 53.69999999999999            |
| Asia      | 24                         | 22                  | 34.60000000000002            | 52                           |
| Australia | 23                         | 21                  | 34                           | 51                           |
| Europe    | 23                         | 23                  | 34                           | 51.39999999999998            |

**Answer:**

- See table above.