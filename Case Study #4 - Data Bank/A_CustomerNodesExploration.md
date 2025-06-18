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

**SQL Statement:**
	
```sql	

```

**Table Output:**

**Answer:**

-

### 5. What is the median, 80th, and 95th percentile for this same reallocation days metric for each region?
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