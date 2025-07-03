# SQL Scripts – WBS Magist Case Study

This folder contains structured SQL queries developed for the Magist marketplace case study during the WBS Coding School Data Science program.

Each script addresses a specific business question related to sales performance, logistics, product categories, and revenue trends using the Magist database.

---

## 📁 Files & Purpose

| File Name                        | Purpose |
|----------------------------------|---------|
| `tech_categories_table.sql`      | Creates a temporary table for tech-related product categories |
| `total_revenue.sql`              | Calculates the total revenue from all products sold |
| `average_order_price.sql`        | Computes the average revenue per order |
| `average_item_price.sql`         | Calculates the average price per item sold |
| `monthly_revenue.sql`            | Summarizes monthly revenue trends |
| `tech_product_sales.sql`         | Counts tech products sold and compares their share of total sales |
| `delivery_performance.sql`       | Measures how early or late deliveries are compared to estimated dates |
| `freight_cost_analysis.sql`      | Analyzes the average freight cost per order |
| `category_sales_distribution.sql`| Breaks down total items sold by product category |
| `tech_delivery_vs_nontech.sql`   | Compares freight cost and delivery times between tech and non-tech products |

---

## 🧰 Usage

Each `.sql` file can be run independently in a PostgreSQL environment with access to the Magist database. Some scripts rely on the temporary `tech_categories` table—run `tech_categories_table.sql` first if needed.

---

## 📌 Notes

- All scripts assume clean, properly formatted data
- Dates are compared using PostgreSQL’s `DATE_PART()` and `DATE_TRUNC()` functions
- Files are modular and can be adapted to other e-commerce datasets

---

## 📄 tech_categories_table.sql
**Purpose:** Creates a temporary table listing all tech-related categories for focused analysis.

```sql
CREATE TEMP TABLE tech_categories AS
SELECT 'computers_accessories' AS category_name UNION
SELECT 'electronics' UNION
SELECT 'telephony';
```

---

## 📄 total_revenue.sql
**Purpose:** Calculates the total revenue from all products sold on the Magist platform.

```sql
SELECT SUM(price) AS total_revenue FROM order_items;
```

---

## 📄 average_order_price.sql
**Purpose:** Computes the average revenue per order.

```sql
SELECT AVG(order_total) AS average_order_price
FROM (
  SELECT order_id, SUM(price) AS order_total
  FROM order_items
  GROUP BY order_id
) AS subquery;
```

---

## 📄 average_item_price.sql
**Purpose:** Calculates the average price per item across all sales.

```sql
SELECT AVG(price) AS average_item_price FROM order_items;
```

---

## 📄 monthly_revenue.sql
**Purpose:** Summarizes revenue per month to evaluate growth trends.

```sql
SELECT DATE_TRUNC('month', order_purchase_timestamp) AS month,
SUM(price) AS revenue
FROM order_items
JOIN orders ON order_items.order_id = orders.order_id
GROUP BY month
ORDER BY month;
```

---

## 📄 tech_product_sales.sql
**Purpose:** Counts total tech product items sold and calculates their percentage of overall sales.

```sql
SELECT COUNT(*) AS tech_items_sold FROM order_items
JOIN products ON order_items.product_id = products.product_id
WHERE products.product_category_name IN (
  SELECT category_name FROM tech_categories
);
```

---

## 📄 delivery_performance.sql
**Purpose:** Calculates the average number of days delivery was earlier or later than estimated.

```sql
SELECT AVG(DATE_PART('day', order_estimated_delivery_date - order_delivered_customer_date)) AS avg_days_early
FROM orders
WHERE order_status = 'delivered';
```

---

## 📄 freight_cost_analysis.sql
**Purpose:** Computes average freight cost per order and analyzes cost variation.

```sql
SELECT AVG(freight_value) AS average_freight FROM order_items;
```

---

## 📄 category_sales_distribution.sql
**Purpose:** Breaks down total items sold per category and calculates percentages.

```sql
SELECT p.product_category_name, COUNT(*) AS items_sold
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
GROUP BY p.product_category_name
ORDER BY items_sold DESC;
```

---

## 📄 tech_delivery_vs_nontech.sql
**Purpose:** Compares delivery times and freight cost between tech and non-tech categories.

```sql
SELECT
  CASE WHEN p.product_category_name IN (SELECT category_name FROM tech_categories)
       THEN 'tech' ELSE 'non-tech' END AS category_group,
  AVG(freight_value) AS avg_freight,
  AVG(DATE_PART('day', order_delivered_customer_date - order_purchase_timestamp)) AS avg_delivery_days
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN orders o ON oi.order_id = o.order_id
GROUP BY category_group;
```
**Author:** Amir Talebi  
**Project:** WBS Data-Driven Business Analysis 
---
