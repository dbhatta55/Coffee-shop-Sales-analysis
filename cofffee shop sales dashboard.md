**1. Database Setup \& Data Cleaning**



```sql

\-- Create and select the database

CREATE DATABASE Coffee\_sales;

USE Coffee\_sales;



\-- Preview the raw data

SELECT \* FROM `coffee shop sales`;

DESCRIBE `coffee shop sales`;



\-- Allow updates without safe mode restriction

SET SQL\_SAFE\_UPDATES = 0;

```



```sql

\-- Fix transaction\_date: convert from string 'DD/MM/YYYY' to proper DATE type

UPDATE `coffee shop sales`

SET transaction\_date = STR\_TO\_DATE(transaction\_date, '%d/%m/%Y');



ALTER TABLE `coffee shop sales`

MODIFY COLUMN transaction\_date DATE;

```



```sql

\-- Fix transaction\_time: convert from string to proper TIME type

UPDATE `coffee shop sales`

SET transaction\_time = STR\_TO\_DATE(transaction\_time, '%H:%i:%s');



ALTER TABLE `coffee shop sales`

MODIFY COLUMN transaction\_time TIME;

```



```sql

\-- Fix encoding issue in transaction\_id column name (BOM character)

ALTER TABLE `coffee shop sales`

CHANGE COLUMN `ï»¿transaction\_id` transaction\_id INT;

```



**2. Total Sales Analysis**



```sql

\-- Total revenue for a specific month (example: May = Month 5)

SELECT 

&#x20;   ROUND(SUM(unit\_price \* transaction\_qty)) AS Total\_Sales

FROM `coffee shop sales`

WHERE MONTH(transaction\_date) = 5;

```



```sql

\-- Month-over-month sales comparison with % change (April vs May)

SELECT 

&#x20;   MONTH(transaction\_date) AS month,

&#x20;   ROUND(SUM(unit\_price \* transaction\_qty)) AS total\_sales,

&#x20;   (SUM(unit\_price \* transaction\_qty) 

&#x20;       - LAG(SUM(unit\_price \* transaction\_qty), 1) OVER (ORDER BY MONTH(transaction\_date)))

&#x20;     / LAG(SUM(unit\_price \* transaction\_qty), 1) OVER (ORDER BY MONTH(transaction\_date)) \* 100 

&#x20;     AS mom\_increase\_percentage

FROM `coffee shop sales`

WHERE MONTH(transaction\_date) IN (4, 5)

GROUP BY MONTH(transaction\_date)

ORDER BY MONTH(transaction\_date);

```



\---



**3. Total Orders Analysis**



```sql

\-- Total number of orders for a specific month (example: April)

SELECT 

&#x20;   COUNT(transaction\_id) AS Total\_Orders

FROM `coffee shop sales`

WHERE MONTH(transaction\_date) = 4;

```



```sql

\-- Month-over-month order count comparison with % change (April vs May)

SELECT 

&#x20;   MONTH(transaction\_date) AS month,

&#x20;   ROUND(COUNT(transaction\_id)) AS total\_orders,

&#x20;   (COUNT(transaction\_id) 

&#x20;       - LAG(COUNT(transaction\_id), 1) OVER (ORDER BY MONTH(transaction\_date)))

&#x20;     / LAG(COUNT(transaction\_id), 1) OVER (ORDER BY MONTH(transaction\_date)) \* 100 

&#x20;     AS mom\_increase\_percentage

FROM `coffee shop sales`

WHERE MONTH(transaction\_date) IN (4, 5)

GROUP BY MONTH(transaction\_date)

ORDER BY MONTH(transaction\_date);

```



\---



**4. Total Quantity Sold Analysis**



```sql

\-- Total quantity sold in a specific month (example: May)

SELECT 

&#x20;   SUM(transaction\_qty) AS Total\_Quantity\_Sold

FROM `coffee shop sales`

WHERE MONTH(transaction\_date) = 5;

```



```sql

\-- Month-over-month quantity comparison with % change (April vs May)

SELECT 

&#x20;   MONTH(transaction\_date) AS month,

&#x20;   ROUND(SUM(transaction\_qty)) AS total\_quantity\_sold,

&#x20;   (SUM(transaction\_qty) 

&#x20;       - LAG(SUM(transaction\_qty), 1) OVER (ORDER BY MONTH(transaction\_date)))

&#x20;     / LAG(SUM(transaction\_qty), 1) OVER (ORDER BY MONTH(transaction\_date)) \* 100 

&#x20;     AS mom\_increase\_percentage

FROM `coffee shop sales`

WHERE MONTH(transaction\_date) IN (4, 5)

GROUP BY MONTH(transaction\_date)

ORDER BY MONTH(transaction\_date);

 **5. KPI Summary for a Specific Date**



```sql

\-- All three KPIs (sales, orders, quantity) formatted for a single date

SELECT 

&#x20;   CONCAT(ROUND(SUM(unit\_price \* transaction\_qty) / 1000, 1), 'K') AS total\_sales,

&#x20;   CONCAT(ROUND(COUNT(transaction\_id) / 1000, 1), 'K')             AS total\_orders,

&#x20;   CONCAT(ROUND(SUM(transaction\_qty) / 1000, 1), 'K')              AS total\_quantity\_sold

FROM `coffee shop sales`

WHERE transaction\_date = '2023-05-18';

```



\---



&#x20;**6. Weekday vs Weekend Sales**



```sql

\-- Compare total revenue on weekdays vs weekends (example: May)

SELECT 

&#x20;   CASE 

&#x20;       WHEN DAYOFWEEK(transaction\_date) IN (1, 7) THEN 'Weekends'

&#x20;       ELSE 'Weekdays'

&#x20;   END AS day\_type,

&#x20;   ROUND(SUM(unit\_price \* transaction\_qty), 2) AS total\_sales

FROM `coffee shop sales`

WHERE MONTH(transaction\_date) = 5

GROUP BY 

&#x20;   CASE 

&#x20;       WHEN DAYOFWEEK(transaction\_date) IN (1, 7) THEN 'Weekends'

&#x20;       ELSE 'Weekdays'

&#x20;   END;

```



\---



&#x20;**7. Sales by Store Location**



```sql

\-- Revenue breakdown by store for a specific month (example: May)

SELECT 

&#x20;   store\_location,

&#x20;   CONCAT(ROUND(SUM(unit\_price \* transaction\_qty) / 1000, 2), 'K') AS Total\_Sales

FROM `coffee shop sales`

WHERE MONTH(transaction\_date) = 5

GROUP BY store\_location

ORDER BY SUM(unit\_price \* transaction\_qty) DESC;

```



\---



**8. Top 10 Products by Revenue**



```sql

\-- Rank top 10 product types by total sales for a specific month (example: May)

SELECT 

&#x20;   product\_type,

&#x20;   ROUND(SUM(unit\_price \* transaction\_qty), 1) AS Total\_Sales

FROM `coffee shop sales`

WHERE MONTH(transaction\_date) = 5

GROUP BY product\_type

ORDER BY SUM(unit\_price \* transaction\_qty) DESC

LIMIT 10;

```



\---

**9. Sales by Specific Day \& Hour (Tooltip / Drill-Through)**



```sql

\-- KPI snapshot for a specific weekday and hour — powers the heatmap tooltip

\-- Example: Tuesday at 8 AM in May

SELECT 

&#x20;   ROUND(SUM(unit\_price \* transaction\_qty)) AS Total\_Sales,

&#x20;   SUM(transaction\_qty)                     AS Total\_Quantity,

&#x20;   COUNT(\*)                                 AS Total\_Orders

FROM `coffee shop sales`

WHERE 

&#x20;   DAYOFWEEK(transaction\_date) = 3   -- 1=Sun, 2=Mon, 3=Tue, 4=Wed, 5=Thu, 6=Fri, 7=Sat

&#x20;   AND HOUR(transaction\_time) = 8    -- Hour in 24-hr format (8 = 8 AM)

&#x20;   AND MONTH(transaction\_date) = 5;  -- Month number (5 = May)

```



\---



**10. Average Daily Sales**



```sql

\-- Calculate average daily revenue for a specific month (example: May)

\-- Uses a subquery to first get each day's total, then averages them

SELECT 

&#x20;   AVG(total\_sales) AS average\_sales

FROM (

&#x20;   SELECT 

&#x20;       SUM(unit\_price \* transaction\_qty) AS total\_sales

&#x20;   FROM `coffee shop sales`

&#x20;   WHERE MONTH(transaction\_date) = 5

&#x20;   GROUP BY transaction\_date

) AS internal\_query;

```



\---



**11. Sales by Day of Week**



```sql

\-- Total revenue broken down by day name for a specific month (example: May)

\-- Useful for identifying which weekdays perform best

SELECT 

&#x20;   CASE 

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 2 THEN 'Monday'

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 3 THEN 'Tuesday'

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 4 THEN 'Wednesday'

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 5 THEN 'Thursday'

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 6 THEN 'Friday'

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 7 THEN 'Saturday'

&#x20;       ELSE 'Sunday'

&#x20;   END AS Day\_of\_Week,

&#x20;   ROUND(SUM(unit\_price \* transaction\_qty)) AS Total\_Sales

FROM `coffee shop sales`

WHERE MONTH(transaction\_date) = 5

GROUP BY 

&#x20;   CASE 

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 2 THEN 'Monday'

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 3 THEN 'Tuesday'

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 4 THEN 'Wednesday'

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 5 THEN 'Thursday'

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 6 THEN 'Friday'

&#x20;       WHEN DAYOFWEEK(transaction\_date) = 7 THEN 'Saturday'

&#x20;       ELSE 'Sunday'

&#x20;   END;



