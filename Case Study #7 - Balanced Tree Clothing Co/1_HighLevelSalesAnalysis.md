# High Level Sales Analysis
## Table of Contents

[1. What was the total quantity sold for all products?](#1-what-was-the-total-quantity-sold-for-all-products)

[2. What is the total generated revenue for all products before discounts?](#2-what-is-the-total-generated-revenue-for-all-products-before-discounts)

[3. What was the total discount amount for all products?](#3-what-was-the-total-discount-amount-for-all-products)

## Questions and Answers
### 1. What was the total quantity sold for all products?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
SELECT
SUM(sales.qty)

FROM balanced_tree.sales AS sales;
```

**Table Output:**
| total_qty_sold |
| -------------- |
| 45216          |

**Answer:**

The total quantity sold for all products in 45,216 pieces.

### 2. What is the total generated revenue for all products before discounts?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
SELECT
SUM(sales.qty * sales.price) total_revenue

FROM balanced_tree.sales AS sales;
```

**Table Output:**
| total_revenue |
| ------------- |
| 1289453       |

**Answer:**

Total revenue before discounts in $1,289,453.

### 2. What was the total discount amount for all products?
___________________________________________________________________________________________________________________________
**SQL Statement:**
	
```sql
SELECT
ROUND(
  SUM(sales.qty * sales.price * sales.discount::numeric/100),
  2
) total_discount

FROM balanced_tree.sales AS sales;
```

**Table Output:**
| total_discount |
| -------------- |
| 156229.14      |

**Answer:**

The total discounts given was $156,229.14.