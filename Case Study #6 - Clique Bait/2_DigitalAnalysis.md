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
```

**Table Output:**

**Answer:**

### 5. What is the percentage of visits which have a purchase event?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
```

**Table Output:**

**Answer:**

### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
```

**Table Output:**

**Answer:**

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