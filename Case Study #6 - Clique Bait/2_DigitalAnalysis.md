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
```

**Table Output:**

**Answer:**

### 3. What is the unique number of visits by all users per month?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
```

**Table Output:**

**Answer:**

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