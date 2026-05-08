# ☕ Coffee Shop Sales Analysis

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Data Analytics](https://img.shields.io/badge/Data%20Analytics-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)

A complete end-to-end sales analysis of a coffee shop chain using **MySQL** for data cleaning & transformation and **Power BI** for interactive dashboards. The project covers revenue trends, product performance, store comparisons, and time-based patterns across Jan–Mar 2023.

---

## 📌 Project Overview

This project analyzes transactional point-of-sale data from three coffee shop locations to answer key business questions:

- How are total sales, orders, and quantity trending month over month?
- Which products and categories drive the most revenue?
- How do the three store locations compare in performance?
- When are peak sales hours — weekdays vs weekends?

---

## 🛠 Tools Used

| Tool | Purpose |
|------|---------|
| MySQL | Data cleaning, formatting, and aggregation |
| Power BI | Interactive dashboard and visualizations |
| Excel / CSV | Raw data source |

---

## 📂 Dataset

- **Source:** Coffee Shop POS transactional data
- **Period:** January 2023 – March 2023
- **Stores:** Hell's Kitchen · Astoria · Lower Manhattan
- **Key Fields:** `transaction_id`, `transaction_date`, `transaction_time`, `store_location`, `product_detail`, `product_category`, `unit_price`, `transaction_qty`

---

## 📊 Dashboard Preview

### January 2023
![January Dashboard](Screenshot%202026-05-08%20130739.jpg)

### February 2023
![February Dashboard](Screenshot%202026-05-08%20130810.jpg)

### March 2023
![March Dashboard](Screenshot%202026-05-08%20131002.jpg)

### Dashboard Tooltip View
![Tooltip View](Screenshot%202026-05-08%20131042.jpg)

---

## 🗄 SQL Queries

### 🔧 1. Database Setup & Data Cleaning

```sql
-- Create and select the database
CREATE DATABASE Coffee_sales;
USE Coffee_sales;

-- Preview the raw data
SELECT * FROM `coffee shop sales`;
DESCRIBE `coffee shop sales`;

-- Allow updates without safe mode restriction
SET SQL_SAFE_UPDATES = 0;
```

```sql
-- Fix transaction_date: convert from string 'DD/MM/YYYY' to proper DATE type
UPDATE `coffee shop sales`
SET transaction_date = STR_TO_DATE(transaction_date, '%d/%m/%Y');

ALTER TABLE `coffee shop sales`
MODIFY COLUMN transaction_date DATE;
```

```sql
-- Fix transaction_time: convert from string to proper TIME type
UPDATE `coffee shop sales`
SET transaction_time = STR_TO_DATE(transaction_time, '%H:%i:%s');

ALTER TABLE `coffee shop sales`
MODIFY COLUMN transaction_time TIME;
```

```sql
-- Fix encoding issue in transaction_id column name (BOM character)
ALTER TABLE `coffee shop sales`
CHANGE COLUMN `ï»¿transaction_id` transaction_id INT;
```

---

### 💰 2. Total Sales Analysis

```sql
-- Total revenue for a specific month (example: May = Month 5)
SELECT 
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales
FROM `coffee shop sales`
WHERE MONTH(transaction_date) = 5;
```

```sql
-- Month-over-month sales comparison with % change (April vs May)
SELECT 
    MONTH(transaction_date) AS month,
    ROUND(SUM(unit_price * transaction_qty)) AS total_sales,
    (SUM(unit_price * transaction_qty) 
        - LAG(SUM(unit_price * transaction_qty), 1) OVER (ORDER BY MONTH(transaction_date)))
      / LAG(SUM(unit_price * transaction_qty), 1) OVER (ORDER BY MONTH(transaction_date)) * 100 
      AS mom_increase_percentage
FROM `coffee shop sales`
WHERE MONTH(transaction_date) IN (4, 5)
GROUP BY MONTH(transaction_date)
ORDER BY MONTH(transaction_date);
```

---

### 🧾 3. Total Orders Analysis

```sql
-- Total number of orders for a specific month (example: April)
SELECT 
    COUNT(transaction_id) AS Total_Orders
FROM `coffee shop sales`
WHERE MONTH(transaction_date) = 4;
```

```sql
-- Month-over-month order count comparison with % change (April vs May)
SELECT 
    MONTH(transaction_date) AS month,
    ROUND(COUNT(transaction_id)) AS total_orders,
    (COUNT(transaction_id) 
        - LAG(COUNT(transaction_id), 1) OVER (ORDER BY MONTH(transaction_date)))
      / LAG(COUNT(transaction_id), 1) OVER (ORDER BY MONTH(transaction_date)) * 100 
      AS mom_increase_percentage
FROM `coffee shop sales`
WHERE MONTH(transaction_date) IN (4, 5)
GROUP BY MONTH(transaction_date)
ORDER BY MONTH(transaction_date);
```

---

### 📦 4. Total Quantity Sold Analysis

```sql
-- Total quantity sold in a specific month (example: May)
SELECT 
    SUM(transaction_qty) AS Total_Quantity_Sold
FROM `coffee shop sales`
WHERE MONTH(transaction_date) = 5;
```

```sql
-- Month-over-month quantity comparison with % change (April vs May)
SELECT 
    MONTH(transaction_date) AS month,
    ROUND(SUM(transaction_qty)) AS total_quantity_sold,
    (SUM(transaction_qty) 
        - LAG(SUM(transaction_qty), 1) OVER (ORDER BY MONTH(transaction_date)))
      / LAG(SUM(transaction_qty), 1) OVER (ORDER BY MONTH(transaction_date)) * 100 
      AS mom_increase_percentage
FROM `coffee shop sales`
WHERE MONTH(transaction_date) IN (4, 5)
GROUP BY MONTH(transaction_date)
ORDER BY MONTH(transaction_date);
```

---

### 📅 5. KPI Summary for a Specific Date

```sql
-- All three KPIs (sales, orders, quantity) formatted for a single date
SELECT 
    CONCAT(ROUND(SUM(unit_price * transaction_qty) / 1000, 1), 'K') AS total_sales,
    CONCAT(ROUND(COUNT(transaction_id) / 1000, 1), 'K')             AS total_orders,
    CONCAT(ROUND(SUM(transaction_qty) / 1000, 1), 'K')              AS total_quantity_sold
FROM `coffee shop sales`
WHERE transaction_date = '2023-05-18';
```

---

### 📆 6. Weekday vs Weekend Sales

```sql
-- Compare total revenue on weekdays vs weekends (example: May)
SELECT 
    CASE 
        WHEN DAYOFWEEK(transaction_date) IN (1, 7) THEN 'Weekends'
        ELSE 'Weekdays'
    END AS day_type,
    ROUND(SUM(unit_price * transaction_qty), 2) AS total_sales
FROM `coffee shop sales`
WHERE MONTH(transaction_date) = 5
GROUP BY 
    CASE 
        WHEN DAYOFWEEK(transaction_date) IN (1, 7) THEN 'Weekends'
        ELSE 'Weekdays'
    END;
```

---

### 🏪 7. Sales by Store Location

```sql
-- Revenue breakdown by store for a specific month (example: May)
SELECT 
    store_location,
    CONCAT(ROUND(SUM(unit_price * transaction_qty) / 1000, 2), 'K') AS Total_Sales
FROM `coffee shop sales`
WHERE MONTH(transaction_date) = 5
GROUP BY store_location
ORDER BY SUM(unit_price * transaction_qty) DESC;
```

---

### 🛍 8. Top 10 Products by Revenue

```sql
-- Rank top 10 product types by total sales for a specific month (example: May)
SELECT 
    product_type,
    ROUND(SUM(unit_price * transaction_qty), 1) AS Total_Sales
FROM `coffee shop sales`
WHERE MONTH(transaction_date) = 5
GROUP BY product_type
ORDER BY SUM(unit_price * transaction_qty) DESC
LIMIT 10;
```

---

### ⏰ 9. Sales by Specific Day & Hour (Tooltip / Drill-Through)

```sql
-- KPI snapshot for a specific weekday and hour — powers the heatmap tooltip
-- Example: Tuesday at 8 AM in May
SELECT 
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales,
    SUM(transaction_qty)                     AS Total_Quantity,
    COUNT(*)                                 AS Total_Orders
FROM `coffee shop sales`
WHERE 
    DAYOFWEEK(transaction_date) = 3   -- 1=Sun, 2=Mon, 3=Tue, 4=Wed, 5=Thu, 6=Fri, 7=Sat
    AND HOUR(transaction_time) = 8    -- Hour in 24-hr format (8 = 8 AM)
    AND MONTH(transaction_date) = 5;  -- Month number (5 = May)
```

---

### 📈 10. Average Daily Sales

```sql
-- Calculate average daily revenue for a specific month (example: May)
-- Uses a subquery to first get each day's total, then averages them
SELECT 
    AVG(total_sales) AS average_sales
FROM (
    SELECT 
        SUM(unit_price * transaction_qty) AS total_sales
    FROM `coffee shop sales`
    WHERE MONTH(transaction_date) = 5
    GROUP BY transaction_date
) AS internal_query;
```

---

### 📅 11. Sales by Day of Week

```sql
-- Total revenue broken down by day name for a specific month (example: May)
-- Useful for identifying which weekdays perform best
SELECT 
    CASE 
        WHEN DAYOFWEEK(transaction_date) = 2 THEN 'Monday'
        WHEN DAYOFWEEK(transaction_date) = 3 THEN 'Tuesday'
        WHEN DAYOFWEEK(transaction_date) = 4 THEN 'Wednesday'
        WHEN DAYOFWEEK(transaction_date) = 5 THEN 'Thursday'
        WHEN DAYOFWEEK(transaction_date) = 6 THEN 'Friday'
        WHEN DAYOFWEEK(transaction_date) = 7 THEN 'Saturday'
        ELSE 'Sunday'
    END AS Day_of_Week,
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales
FROM `coffee shop sales`
WHERE MONTH(transaction_date) = 5
GROUP BY 
    CASE 
        WHEN DAYOFWEEK(transaction_date) = 2 THEN 'Monday'
        WHEN DAYOFWEEK(transaction_date) = 3 THEN 'Tuesday'
        WHEN DAYOFWEEK(transaction_date) = 4 THEN 'Wednesday'
        WHEN DAYOFWEEK(transaction_date) = 5 THEN 'Thursday'
        WHEN DAYOFWEEK(transaction_date) = 6 THEN 'Friday'
        WHEN DAYOFWEEK(transaction_date) = 7 THEN 'Saturday'
        ELSE 'Sunday'
    END;
```

---

## 💡 Key Insights

- **March 2023** was the strongest month with **$99K** in revenue — a **+29.8%** jump vs February
- **Barista Espresso** is the top-selling product across all months
- **Hell's Kitchen** consistently leads in store-level revenue
- **Morning hours (8–10 AM)** show the highest sales concentration in the heatmap
- **Weekdays** account for ~71–75% of total monthly revenue

---

## 🚀 How to Run

1. Clone this repository
2. Import the raw CSV into MySQL and run the scripts in `/sql/` in order
3. Open `Coffee_Shop_Sales.pbix` in Power BI Desktop
4. Refresh the data source connection if needed

---

## 📁 Repository Structure

```
📦 coffee-shop-sales
 ┣ 📂 sql
 ┃ ┗ 📄 coffee_shop_analysis.sql
 ┣ 📂 screenshots
 ┃ ┣ 🖼 jan_2023.jpg
 ┃ ┣ 🖼 feb_2023.jpg
 ┃ ┗ 🖼 mar_2023.jpg
 ┣ 📄 Coffee_Shop_Sales.pbix
 ┗ 📄 README.md
```

---

## 🙋 Author

Made with ☕ — feel free to fork, star, or raise issues!
