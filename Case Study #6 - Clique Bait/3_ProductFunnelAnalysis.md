# Product Funnel Analysis
## Table of Contents

[1. Which product had the most views, cart adds and purchases?](#1-which-product-had-the-most-views-cart-adds-and-purchases)

[2. Which product was most likely to be abandoned?](#2-which-product-was-most-likely-to-be-abandoned)

[3. Which product had the highest view to purchase percentage?](#3-which-product-had-the-highest-view-to-purchase-percentage)

[4. What is the average conversion rate from view to cart add?](#4-what-is-the-average-conversion-rate-from-view-to-cart-add)

[5. What is the average conversion rate from cart add to purchase?](#5-what-is-the-average-conversion-rate-from-cart-add-to-purchase)

## Pre-Requisite Work
Using a single SQL query - create a new output table which has the following details:

--> How many times was each product viewed?

--> How many times was each product added to cart?

--> How many times was each product added to a cart but not purchased (abandoned)?

--> How many times was each product purchased?

Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

Use your 2 new output tables - answer the following questions.

### ~~ Table 1 ~~
**SQL Statement:**
	
```sql
/* Determine the number of times a Page View event has happened for each page_name. */
WITH page_view_events AS (
  SELECT
  ph.page_name
  ,SUM(
    CASE
  	  WHEN ei.event_name = 'Page View' THEN 1
      ELSE 0
    END
  ) AS page_views
  
FROM clique_bait.events AS e
JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id

WHERE
ph.product_category IS NOT NULL

GROUP BY ph.page_name
),
/* Determine the number of times an Add to Cart event happened for each page_name. */
add_to_cart_events AS (
  SELECT
  ph.page_name
  ,SUM(
    CASE
  	  WHEN ei.event_name = 'Add to Cart' THEN 1
      ELSE 0
    END
  ) AS add_to_cart
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id

  WHERE
  ph.product_category IS NOT NULL

  GROUP BY ph.page_name
),
/* Determine the total number of purchase events and the IDs associated to those purchase events. */
purchase_events AS (
  SELECT
  e.visit_id
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  
  WHERE
  ei.event_name = 'Purchase'
),
/* COUNT the visit_id as purchases but only when the visit_id is in the list of the purchase_events CTE AND when the event_name is Add to Cart. */
purchases AS (
  SELECT
  ph.page_name
  ,COUNT(e.visit_id) AS purchases

  FROM clique_bait.events AS e
  JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type

  WHERE
  /* This ensures we're only looking at the IDs that made purchases. */
  e.visit_id IN (SELECT
    		     visit_id
  			     FROM purchase_events) AND
  /* This ensures we're only looking at events where something was added to the cart. */
  ei.event_name = 'Add to Cart'
               
  GROUP BY ph.page_name
),
/* Use the add_to_cart_events CTE to get the add_to_cart amount, and subtract the purchases from the purchases CTE from it to get how many times something was added to the cart but not purchased. */
abandoned_events AS (
  SELECT
  ace.page_name
  ,ace.add_to_cart - p.purchases AS abandoned
  
  FROM add_to_cart_events AS ace
  JOIN purchases AS p ON p.page_name = ace.page_name
)

SELECT
pve.page_name
,pve.page_views
,ace.add_to_cart
,ae.abandoned
,p.purchases

FROM page_view_events AS pve
JOIN add_to_cart_events AS ace ON ace.page_name = pve.page_name
JOIN abandoned_events AS ae ON ae.page_name = pve.page_name
JOIN purchases AS p ON p.page_name = pve.page_name

ORDER BY pve.page_name ASC;
```

**Table Output:**
| page_name      | page_views | add_to_cart | abandoned | purchases |
| -------------- | ---------- | ----------- | --------- | --------- |
| Abalone        | 1525       | 932         | 233       | 699       |
| Black Truffle  | 1469       | 924         | 217       | 707       |
| Crab           | 1564       | 949         | 230       | 719       |
| Kingfish       | 1559       | 920         | 213       | 707       |
| Lobster        | 1547       | 968         | 214       | 754       |
| Oyster         | 1568       | 943         | 217       | 726       |
| Russian Caviar | 1563       | 946         | 249       | 697       |
| Salmon         | 1559       | 938         | 227       | 711       |
| Tuna           | 1515       | 931         | 234       | 697       |

### ~~ Table 2 ~~
**SQL Statement:**
	
```sql
/* Determine the number of times a Page View event has happened for each page_name. */
WITH page_view_events AS (
  SELECT
  ph.product_category
  ,SUM(
    CASE
  	  WHEN ei.event_name = 'Page View' THEN 1
      ELSE 0
    END
  ) AS page_views
  
FROM clique_bait.events AS e
JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id

WHERE
ph.product_category IS NOT NULL

GROUP BY ph.product_category
),
/* Determine the number of times an Add to Cart event happened for each page_name. */
add_to_cart_events AS (
  SELECT
  ph.product_category
  ,SUM(
    CASE
  	  WHEN ei.event_name = 'Add to Cart' THEN 1
      ELSE 0
    END
  ) AS add_to_cart
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id

  WHERE
  ph.product_category IS NOT NULL

  GROUP BY ph.product_category
),
/* Determine the total number of purchase events and the IDs associated to those purchase events. */
purchase_events AS (
  SELECT
  e.visit_id
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  
  WHERE
  ei.event_name = 'Purchase'
),
/* COUNT the visit_id as purchases but only when the visit_id is in the list of the purchase_events CTE AND when the event_name is Add to Cart. */
purchases AS (
  SELECT
  ph.product_category
  ,COUNT(e.visit_id) AS purchases

  FROM clique_bait.events AS e
  JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type

  WHERE
  /* This ensures we're only looking at the IDs that made purchases. */
  e.visit_id IN (SELECT
    		     visit_id
  			     FROM purchase_events) AND
  /* This ensures we're only looking at events where something was added to the cart. */
  ei.event_name = 'Add to Cart'
               
  GROUP BY ph.product_category
),
/* Use the add_to_cart_events CTE to get the add_to_cart amount, and subtract the purchases from the purchases CTE from it to get how many times something was added to the cart but not purchased. */
abandoned_events AS (
  SELECT
  ace.product_category
  ,ace.add_to_cart - p.purchases AS abandoned
  
  FROM add_to_cart_events AS ace
  JOIN purchases AS p ON p.product_category = ace.product_category
)

SELECT
pve.product_category
,pve.page_views
,ace.add_to_cart
,ae.abandoned
,p.purchases

FROM page_view_events AS pve
JOIN add_to_cart_events AS ace ON ace.product_category = pve.product_category
JOIN abandoned_events AS ae ON ae.product_category = pve.product_category
JOIN purchases AS p ON p.product_category = pve.product_category

ORDER BY pve.product_category ASC;
```

**Table Output:**
| product_category | page_views | add_to_cart | abandoned | purchases |
| ---------------- | ---------- | ----------- | --------- | --------- |
| Fish             | 4633       | 2789        | 674       | 2115      |
| Luxury           | 3032       | 1870        | 466       | 1404      |
| Shellfish        | 6204       | 3792        | 894       | 2898      |

## Questions and Answers
### 1. Which product had the most views, cart adds and purchases?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
/* Determine the number of times a Page View event has happened for each page_name. */
WITH page_view_events AS (
  SELECT
  ph.page_name
  ,SUM(
    CASE
  	  WHEN ei.event_name = 'Page View' THEN 1
      ELSE 0
    END
  ) AS page_views
  
FROM clique_bait.events AS e
JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id

WHERE
ph.product_category IS NOT NULL

GROUP BY ph.page_name
),
/* Determine the number of times an Add to Cart event happened for each page_name. */
add_to_cart_events AS (
  SELECT
  ph.page_name
  ,SUM(
    CASE
  	  WHEN ei.event_name = 'Add to Cart' THEN 1
      ELSE 0
    END
  ) AS add_to_cart
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id

  WHERE
  ph.product_category IS NOT NULL

  GROUP BY ph.page_name
),
/* Determine the total number of purchase events and the IDs associated to those purchase events. */
purchase_events AS (
  SELECT
  e.visit_id
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  
  WHERE
  ei.event_name = 'Purchase'
),
/* COUNT the visit_id as purchases but only when the visit_id is in the list of the purchase_events CTE AND when the event_name is Add to Cart. */
purchases AS (
  SELECT
  ph.page_name
  ,COUNT(e.visit_id) AS purchases

  FROM clique_bait.events AS e
  JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type

  WHERE
  /* This ensures we're only looking at the IDs that made purchases. */
  e.visit_id IN (SELECT
    		     visit_id
  			     FROM purchase_events) AND
  /* This ensures we're only looking at events where something was added to the cart. */
  ei.event_name = 'Add to Cart'
               
  GROUP BY ph.page_name
),
/* Use the add_to_cart_events CTE to get the add_to_cart amount, and subtract the purchases from the purchases CTE from it to get how many times something was added to the cart but not purchased. */
abandoned_events AS (
  SELECT
  ace.page_name
  ,ace.add_to_cart - p.purchases AS abandoned
  
  FROM add_to_cart_events AS ace
  JOIN purchases AS p ON p.page_name = ace.page_name
)
,product_numbers AS (
  SELECT
  pve.page_name
  ,pve.page_views
  ,ace.add_to_cart
  ,ae.abandoned
  ,p.purchases

  FROM page_view_events AS pve
  JOIN add_to_cart_events AS ace ON ace.page_name = pve.page_name
  JOIN abandoned_events AS ae ON ae.page_name = pve.page_name
  JOIN purchases AS p ON p.page_name = pve.page_name

  ORDER BY pve.page_name
)

SELECT
(SELECT pn.page_name
 FROM product_numbers AS pn
 ORDER BY pn.page_views DESC
 LIMIT 1
) AS most_page_views
,(SELECT pn.page_name
 FROM product_numbers AS pn
 ORDER BY pn.add_to_cart DESC
 LIMIT 1
) AS most_added_to_cart
,(SELECT pn.page_name
 FROM product_numbers AS pn
 ORDER BY pn.purchases DESC
 LIMIT 1
) AS most_purchased;
```

**Table Output:**
| most_page_views | most_added_to_cart | most_purchased |
| --------------- | ------------------ | -------------- |
| Oyster          | Lobster            | Lobster        |

**Answer:**

Oyster had the most page views, and Lobster was added to carts and purchased the most.

### 2. Which product was most likely to be abandoned?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
/* Determine the number of times a Page View event has happened for each page_name. */
WITH page_view_events AS (
  SELECT
  ph.page_name
  ,SUM(
    CASE
  	  WHEN ei.event_name = 'Page View' THEN 1
      ELSE 0
    END
  ) AS page_views
  
FROM clique_bait.events AS e
JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id

WHERE
ph.product_category IS NOT NULL

GROUP BY ph.page_name
),
/* Determine the number of times an Add to Cart event happened for each page_name. */
add_to_cart_events AS (
  SELECT
  ph.page_name
  ,SUM(
    CASE
  	  WHEN ei.event_name = 'Add to Cart' THEN 1
      ELSE 0
    END
  ) AS add_to_cart
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id

  WHERE
  ph.product_category IS NOT NULL

  GROUP BY ph.page_name
),
/* Determine the total number of purchase events and the IDs associated to those purchase events. */
purchase_events AS (
  SELECT
  e.visit_id
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  
  WHERE
  ei.event_name = 'Purchase'
),
/* COUNT the visit_id as purchases but only when the visit_id is in the list of the purchase_events CTE AND when the event_name is Add to Cart. */
purchases AS (
  SELECT
  ph.page_name
  ,COUNT(e.visit_id) AS purchases

  FROM clique_bait.events AS e
  JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type

  WHERE
  /* This ensures we're only looking at the IDs that made purchases. */
  e.visit_id IN (SELECT
    		     visit_id
  			     FROM purchase_events) AND
  /* This ensures we're only looking at events where something was added to the cart. */
  ei.event_name = 'Add to Cart'
               
  GROUP BY ph.page_name
),
/* Use the add_to_cart_events CTE to get the add_to_cart amount, and subtract the purchases from the purchases CTE from it to get how many times something was added to the cart but not purchased. */
abandoned_events AS (
  SELECT
  ace.page_name
  ,ace.add_to_cart - p.purchases AS abandoned
  
  FROM add_to_cart_events AS ace
  JOIN purchases AS p ON p.page_name = ace.page_name
)
,product_numbers AS (
  SELECT
  pve.page_name
  ,pve.page_views
  ,ace.add_to_cart
  ,ae.abandoned
  ,p.purchases

  FROM page_view_events AS pve
  JOIN add_to_cart_events AS ace ON ace.page_name = pve.page_name
  JOIN abandoned_events AS ae ON ae.page_name = pve.page_name
  JOIN purchases AS p ON p.page_name = pve.page_name

  ORDER BY pve.page_name
)

SELECT
(SELECT pn.page_name
 FROM product_numbers AS pn
 ORDER BY pn.abandoned DESC
 LIMIT 1
) most_abandoned;
```

**Table Output:**
| most_abandoned |
| -------------- |
| Russian Caviar |

**Answer:**

Russian Caviar was most likely to be abandoned.

### 3. Which product had the highest view to purchase percentage?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
/* Determine the number of times a Page View event has happened for each page_name. */
WITH page_view_events AS (
  SELECT
  ph.page_name
  ,SUM(
    CASE
  	  WHEN ei.event_name = 'Page View' THEN 1
      ELSE 0
    END
  ) AS page_views
  
FROM clique_bait.events AS e
JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id

WHERE
ph.product_category IS NOT NULL

GROUP BY ph.page_name
),
/* Determine the number of times an Add to Cart event happened for each page_name. */
add_to_cart_events AS (
  SELECT
  ph.page_name
  ,SUM(
    CASE
  	  WHEN ei.event_name = 'Add to Cart' THEN 1
      ELSE 0
    END
  ) AS add_to_cart
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id

  WHERE
  ph.product_category IS NOT NULL

  GROUP BY ph.page_name
),
/* Determine the total number of purchase events and the IDs associated to those purchase events. */
purchase_events AS (
  SELECT
  e.visit_id
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  
  WHERE
  ei.event_name = 'Purchase'
),
/* COUNT the visit_id as purchases but only when the visit_id is in the list of the purchase_events CTE AND when the event_name is Add to Cart. */
purchases AS (
  SELECT
  ph.page_name
  ,COUNT(e.visit_id) AS purchases

  FROM clique_bait.events AS e
  JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type

  WHERE
  /* This ensures we're only looking at the IDs that made purchases. */
  e.visit_id IN (SELECT
    		     visit_id
  			     FROM purchase_events) AND
  /* This ensures we're only looking at events where something was added to the cart. */
  ei.event_name = 'Add to Cart'
               
  GROUP BY ph.page_name
),
/* Use the add_to_cart_events CTE to get the add_to_cart amount, and subtract the purchases from the purchases CTE from it to get how many times something was added to the cart but not purchased. */
abandoned_events AS (
  SELECT
  ace.page_name
  ,ace.add_to_cart - p.purchases AS abandoned
  
  FROM add_to_cart_events AS ace
  JOIN purchases AS p ON p.page_name = ace.page_name
)
,product_numbers AS (
  SELECT
  pve.page_name
  ,pve.page_views
  ,ace.add_to_cart
  ,ae.abandoned
  ,p.purchases

  FROM page_view_events AS pve
  JOIN add_to_cart_events AS ace ON ace.page_name = pve.page_name
  JOIN abandoned_events AS ae ON ae.page_name = pve.page_name
  JOIN purchases AS p ON p.page_name = pve.page_name

  ORDER BY pve.page_name
)

SELECT
pn.page_name
,ROUND(
  pn.purchases::numeric / pn.page_views * 100,
  2
) AS percentage_of_page_views_to_purchases

FROM product_numbers AS pn

ORDER BY percentage_of_page_views_to_purchases DESC

LIMIT 1;
```

**Table Output:**
| page_name | percentage_of_page_views_to_purchases |
| --------- | ------------------------------------- |
| Lobster   | 48.74                                 |

**Answer:**

Lobster has the most page views to purchases percentage at a rate of 48.74%.

### 4. What is the average conversion rate from view to cart add?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
```

**Table Output:**

**Answer:**

### 5. What is the average conversion rate from cart add to purchase?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
```

**Table Output:**

**Answer:**