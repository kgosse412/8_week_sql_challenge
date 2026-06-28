# Campaigns Analysis
## Table of Contents

## Pre-Requisite Work
Generate a table that has 1 single row for every unique visit_id record and has the following columns:

--> user_id

--> visit_id

--> visit_start_time: the earliest event_time for each visit

--> page_views: count of page views for each visit

--> cart_adds: count of product cart add events for each visit

--> purchase: 1/0 flag if a purchase event exists for each visit

--> campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date

--> impression: count of ad impressions for each visit

--> click: count of ad clicks for each visit

--> (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.

### ~~ Table ~~
**SQL Statement:**
	
```sql
/* Determine the total number of purchase events and the IDs associated to those purchase events. */
WITH purchase_events AS (
  SELECT
  e.visit_id
  
  FROM clique_bait.events AS e
  JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
  
  WHERE
  ei.event_name = 'Purchase'
)

SELECT
u.user_id
,e.visit_id
,MIN(e.event_time) AS visit_start_time
,SUM(
    CASE
  	  WHEN ei.event_name = 'Page View' THEN 1
      ELSE 0
    END
  ) AS page_views
,SUM(
    CASE
  	  WHEN ei.event_name = 'Add to Cart' THEN 1
      ELSE 0
    END
) AS cart_adds
,MAX(
  CASE
  	WHEN e.visit_id = pe.visit_id THEN 1
    ELSE 0
  END
) AS purchases
,ci.campaign_name
,SUM(
  CASE
    WHEN ei.event_name = 'Ad Impression' THEN 1
    ELSE 0
  END
) AS impressions
,SUM(
  CASE
    WHEN ei.event_name = 'Ad Click' THEN 1
    ELSE 0
  END
) AS click
,STRING_AGG(ph.page_name, ', ' ORDER BY e.sequence_number ASC) 
 FILTER (WHERE ph.product_category IS NOT NULL AND ei.event_name = 'Add to Cart')

FROM clique_bait.events AS e
JOIN clique_bait.users AS u ON u.cookie_id = e.cookie_id
JOIN clique_bait.event_identifier AS ei ON ei.event_type = e.event_type
JOIN clique_bait.page_hierarchy AS ph ON ph.page_id = e.page_id
LEFT JOIN purchase_events AS pe ON pe.visit_id = e.visit_id
LEFT JOIN clique_bait.campaign_identifier AS ci ON e.event_time BETWEEN ci.start_date AND ci.end_date

WHERE
/* Limit the table so it's not as large. */
u.user_id <= 5

GROUP BY u.user_id, e.visit_id, ci.campaign_name

ORDER BY u.user_id ASC, visit_start_time ASC;
```

**Table Output:**
| user_id | visit_id | visit_start_time           | page_views | cart_adds | purchases | campaign_name                     | impressions | click | string_agg                                                                            |
| ------- | -------- | -------------------------- | ---------- | --------- | --------- | --------------------------------- | ----------- | ----- | ------------------------------------------------------------------------------------- |
| 1       | 0fc437   | 2020-02-04 17:49:49.602976 | 10         | 6         | 1         | Half Off - Treat Your Shellf(ish) | 1           | 1     | Tuna, Russian Caviar, Black Truffle, Abalone, Crab, Oyster                            |
| 1       | ccf365   | 2020-02-04 19:16:09.182546 | 7          | 3         | 1         | Half Off - Treat Your Shellf(ish) | 0           | 0     | Lobster, Crab, Oyster                                                                 |
| 1       | 0826dc   | 2020-02-26 05:58:37.918618 | 1          | 0         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     |                                                                                       |
| 1       | 02a5d5   | 2020-02-26 16:57:26.260871 | 4          | 0         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     |                                                                                       |
| 1       | f7c798   | 2020-03-15 02:23:26.312543 | 9          | 3         | 1         | Half Off - Treat Your Shellf(ish) | 0           | 0     | Russian Caviar, Crab, Oyster                                                          |
| 1       | 30b94d   | 2020-03-15 13:12:54.023936 | 9          | 7         | 1         | Half Off - Treat Your Shellf(ish) | 1           | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Abalone, Lobster, Crab                        |
| 1       | 41355d   | 2020-03-25 00:11:17.860655 | 6          | 1         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     | Lobster                                                                               |
| 1       | eaffde   | 2020-03-25 20:06:32.342989 | 10         | 8         | 1         | Half Off - Treat Your Shellf(ish) | 1           | 1     | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster           |
| 2       | 3b5871   | 2020-01-18 10:16:32.158475 | 9          | 6         | 1         | 25% Off - Living The Lux Life     | 1           | 1     | Salmon, Kingfish, Russian Caviar, Black Truffle, Lobster, Oyster                      |
| 2       | c5c0ee   | 2020-01-18 10:35:22.765382 | 1          | 0         | 0         | 25% Off - Living The Lux Life     | 0           | 0     |                                                                                       |
| 2       | e26a84   | 2020-01-18 16:06:40.90728  | 6          | 2         | 1         | 25% Off - Living The Lux Life     | 0           | 0     | Salmon, Oyster                                                                        |
| 2       | d58cbd   | 2020-01-18 23:40:54.761906 | 8          | 4         | 0         | 25% Off - Living The Lux Life     | 0           | 0     | Kingfish, Tuna, Abalone, Crab                                                         |
| 2       | 910d9a   | 2020-02-01 10:40:46.875968 | 8          | 1         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     | Abalone                                                                               |
| 2       | 1f1198   | 2020-02-01 21:51:55.078775 | 1          | 0         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     |                                                                                       |
| 2       | 49d73d   | 2020-02-16 06:21:27.138532 | 11         | 9         | 1         | Half Off - Treat Your Shellf(ish) | 1           | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster |
| 2       | 0635fb   | 2020-02-16 06:42:42.73573  | 9          | 4         | 1         | Half Off - Treat Your Shellf(ish) | 0           | 0     | Salmon, Kingfish, Abalone, Crab                                                       |
| 3       | 9a2f24   | 2020-02-21 03:19:10.032455 | 6          | 2         | 1         | Half Off - Treat Your Shellf(ish) | 0           | 0     | Kingfish, Black Truffle                                                               |
| 3       | 25502e   | 2020-02-21 11:26:15.353389 | 1          | 0         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     |                                                                                       |
| 3       | bf200a   | 2020-03-11 04:10:26.708385 | 7          | 2         | 1         | Half Off - Treat Your Shellf(ish) | 0           | 0     | Salmon, Crab                                                                          |
| 3       | eb13cd   | 2020-03-11 21:36:37.222763 | 1          | 0         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     |                                                                                       |
| 3       | 80e2fe   | 2020-04-08 04:08:00.231658 | 10         | 5         | 1         |                                   | 0           | 0     | Salmon, Tuna, Russian Caviar, Abalone, Oyster                                         |
| 3       | dda9ae   | 2020-04-08 18:24:44.8597   | 10         | 8         | 1         |                                   | 1           | 1     | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster           |
| 3       | 791afc   | 2020-04-29 00:37:16.741118 | 8          | 2         | 1         |                                   | 0           | 0     | Salmon, Oyster                                                                        |
| 3       | 8902ad   | 2020-04-29 22:56:53.062046 | 10         | 6         | 0         |                                   | 1           | 1     | Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster                         |
| 3       | 7e89a0   | 2020-05-28 10:57:51.749847 | 9          | 6         | 0         |                                   | 1           | 1     | Salmon, Tuna, Russian Caviar, Black Truffle, Lobster, Crab                            |
| 3       | 76ee84   | 2020-05-28 20:11:54.997406 | 7          | 3         | 1         |                                   | 0           | 0     | Salmon, Lobster, Crab                                                                 |
| 4       | 7caba5   | 2020-02-22 17:49:37.646174 | 5          | 2         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     | Tuna, Lobster                                                                         |
| 4       | 4c0ce3   | 2020-02-22 19:42:45.498271 | 1          | 0         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     |                                                                                       |
| 4       | b90e25   | 2020-03-19 11:01:58.182947 | 9          | 4         | 1         | Half Off - Treat Your Shellf(ish) | 1           | 1     | Tuna, Black Truffle, Lobster, Crab                                                    |
| 4       | 07a950   | 2020-03-19 17:56:24.610445 | 6          | 0         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     |                                                                                       |
| 5       | f61ed7   | 2020-02-01 06:30:39.766168 | 8          | 2         | 1         | Half Off - Treat Your Shellf(ish) | 0           | 0     | Lobster, Crab                                                                         |
| 5       | b45feb   | 2020-02-01 07:47:45.025247 | 1          | 0         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     |                                                                                       |
| 5       | 580bf6   | 2020-02-11 04:05:42.307991 | 8          | 6         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster                         |
| 5       | 05c52a   | 2020-02-11 12:30:33.479052 | 9          | 8         | 0         | Half Off - Treat Your Shellf(ish) | 1           | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab         |
| 5       | fa70cb   | 2020-02-26 11:12:20.361638 | 1          | 0         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     |                                                                                       |
| 5       | 4bffe1   | 2020-02-26 16:03:10.377881 | 8          | 1         | 0         | Half Off - Treat Your Shellf(ish) | 0           | 0     | Russian Caviar                                                                              |

## Insights Found
___________________________________________________________________________________________________________________________
