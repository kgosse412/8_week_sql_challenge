# Digital Analysis
## Table of Contents

[1. How many users are there?](#1-how-many-users-are-there)

[2. How many cookies does each user have on average?](#2-how-many-cookies-does-each-user-have-on-average)

[3. What is the unique number of visits by all users per month?](#3-what-is-the-unique-number-of-visits-by-all-users-per-month)

[4. What is the number of events for each event type?](#4-what-is-the-number-of-events-for-each-event-type)

[5. What is the percentage of visits which have a purchase event?](#5-what-is-the-percentage-of-visits-which-have-a-purchase-event)

[6. What is the percentage of visits which view the checkout page but do not have a purchase event?](#6-what-is-the-percentage-of-visits-which-view-the-checkout-page-but-do-not-have-a-purchase-event)

[7. What are the top 3 pages by number of views?](#7-what-are-the-top-3-pages-by-number-of-views)

[8. What is the number of views and cart adds for each product category?](#8-what-is-the-number-of-views-and-cart-adds-for-each-product-category)

[9. What are the top 3 products by purchases?](#9-what-are-the-top-3-products-by-purchases)

## Questions and Answers
### 1. How many users are there?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
SELECT
COUNT(DISTINCT usrs.user_id) AS number_of_users

FROM clique_bait.users AS usrs;
```

**Table Output:**
| number_of_users |
| --------------- |
| 500             |

**Answer:**

There are a total of 500 distinct users.

### 2. How many cookies does each user have on average?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
WITH count_of_cookies AS (
  SELECT
  DISTINCT usrs.user_id AS users
  ,COUNT(cookie_id) AS cookie_cnt

  FROM clique_bait.users AS usrs

  GROUP BY users

  ORDER BY users ASC
)
  
SELECT
/* Round the average cookie_cnt to 0 because you can't have part of a cookie. */
ROUND(AVG(cookie_cnt), 0) AS avg_cookie_cnt
  
FROM count_of_cookies;
```

**Table Output:**
| avg_cookie_cnt |
| -------------- |
| 4              |

**Answer:**

Each person has an average of 4 cookies (if you round to 2 decimal places, you get 3.56).

### 3. What is the unique number of visits by all users per month?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
SELECT
EXTRACT('year' FROM event_time) AS year_number
,EXTRACT('month' FROM event_time) AS month_number
,COUNT(DISTINCT visit_id) AS cnt_of_visits
  
FROM clique_bait.events

GROUP BY year_number, month_number

ORDER BY year_number ASC, month_number ASC;
```

**Table Output:**
| year_number | month_number | cnt_of_visits |
| ----------- | ------------ | ------------- |
| 2020        | 1            | 876           |
| 2020        | 2            | 1488          |
| 2020        | 3            | 916           |
| 2020        | 4            | 248           |
| 2020        | 5            | 36            |

**Answer:**

See table for answer.

### 4. What is the number of events for each event type?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
SELECT
event_type
,COUNT(visit_id) AS num_of_events
  
FROM clique_bait.events

GROUP BY event_type

ORDER BY event_type ASC;
```

**Table Output:**
| event_type | num_of_events |
| ---------- | ------------- |
| 1          | 20928         |
| 2          | 8451          |
| 3          | 1777          |
| 4          | 876           |
| 5          | 702           |

**Answer:**

See table for answer.

### 5. What is the percentage of visits which have a purchase event?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
/* First, we need to find how many unique visits are Purchases. To do this, we COUNT the DISTINCT number of visit_id when an event_name is Purchase. This means we also need to join the event_identifier table to the events table so we know which one is the Purchase type. */
WITH purchase_events AS (
  SELECT
  COUNT(DISTINCT e.visit_id) AS num_of_purchase_events
  
  FROM clique_bait.events AS e
  LEFT JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type

  WHERE
  ei.event_name = 'Purchase'
)

/* Secondly, we need to figure out what percentage is for the number of purchase events is over the distinct number of events. To do this, we do a subquery where we select the value from num_of_purchase_events from the purchase_events CTE, then divide by the total distinct count of the visit_id. Multiply that by 100 to get the percentage and round to 2 decimal points for cleanness. */
SELECT
ROUND(
  (SELECT pe.num_of_purchase_events
  FROM purchase_events AS pe)::numeric / 
  COUNT(DISTINCT e.visit_id) * 100,
  2) AS purchase_percentage
  
FROM clique_bait.events AS e;
```

**Table Output:**
| purchase_percentage |
| ------------------- |
| 49.86               |

**Answer:**

The percentage of visits that have a purchase event is 49.86%.

### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
___________________________________________________________________________________________________________________________
I actuall solved this two different ways. The second way is more efficient than the first but both will get you the same numbers.

#### ~~ First Solution ~~
**SQL Statement:**
	
```sql
/* First, we figure out the number of purchase_events there are. */
WITH purchase_events AS (
  SELECT
  COUNT(DISTINCT e.visit_id) AS num_of_purchase_events
  
  FROM clique_bait.events AS e
  LEFT JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type

  WHERE
  ei.event_name = 'Purchase'
)

/* Then we figure out how many checkout_events there are. */
SELECT
/* Will only look at checkout_events because of the WHERE clause. */
COUNT(DISTINCT e.visit_id) AS checkout_events
/* Pull in the purchase_events from the purchase_events table. */
,(SELECT pe.num_of_purchase_events
  FROM purchase_events AS pe) AS purchase_events
,COUNT(DISTINCT e.visit_id) - 
(SELECT pe.num_of_purchase_events
  FROM purchase_events AS pe) AS checkout_only_events
,ROUND(
  (
    COUNT(DISTINCT e.visit_id) - 
    (SELECT pe.num_of_purchase_events
 	 FROM purchase_events AS pe)
  )::numeric /
  COUNT(DISTINCT e.visit_id) * 100,
2) AS checkout_percentage
  
FROM clique_bait.events AS e
LEFT JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id

WHERE
ph.page_name = 'Checkout'
```

**Table Output:**
| checkout_events | purchase_events | checkout_only_events | checkout_percentage |
| --------------- | --------------- | -------------------- | ------------------- |
| 2103            | 1777            | 326                  | 15.50               |

#### ~~ Second Solution ~~
**SQL Statement:**
	
```sql
/* First, join the relevant tables and get the relevant info from each table. */
WITH expanded_events AS (
  SELECT
  e.visit_id
  ,e.page_id
  ,ph.page_name
  ,e.event_type
  ,ei.event_name
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id
)
/* Second, we want to calculate how many times the checkout page is viewed and how many times a purchase was made. */
,event_calc AS (
  SELECT
  SUM(CASE
      WHEN ee.page_name = 'Checkout' THEN 1
      ELSE 0
  END) AS checkout_events
  ,SUM(CASE
      WHEN ee.event_name = 'Purchase' THEN 1
      ELSE 0
  END) AS purchase_events

  FROM expanded_events AS ee
)

/* Finally, we want to use those values calcuated in event_calc to figure out two things;
1. How many checkout only events were there
2. What percentage of the checkout events were checkout only
*/
SELECT
ec.*
,ec.checkout_events - ec.purchase_events AS checkout_only_events
,ROUND(
  (ec.checkout_events - ec.purchase_events)::numeric / ec.checkout_events * 100,
2) AS checkout_only_events_percentage

FROM event_calc AS ec
```

**Table Output:**
| checkout_events | purchase_events | checkout_only_events | checkout_only_events_percentage |
| --------------- | --------------- | -------------------- | ------------------------------- |
| 2103            | 1777            | 326                  | 15.50                           |

**Answer:**

Either solution gives a percentage of 15.50% where there are only 326 checkout only events.

### 7. What are the top 3 pages by number of views?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
```

**Table Output:**

**Answer:**

### 8. What is the number of views and cart adds for each product category?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
```

**Table Output:**

**Answer:**

### 9. What are the top 3 products by purchases?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
```

**Table Output:**

**Answer:**