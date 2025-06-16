
#  Supply Chain Order Analytics | MySQL

###  Analyze order performance, cancellation trends, and supply chain KPIs using  MySQL.

---

### Table of Contents

* [Project Overview](#project-overview)
* [Dataset Description](#dataset-description)
* [Database Schema](#database-schema)
* [Data Cleaning ](#data-cleaning)
* [Key Metrics ](#key-metrics)
* [Business Questions](#business-questions)
* [SQL Views and Stored Procedures](#sql-views-and-stored-procedures)
* [Business Insights](#business-insights)
* [Recommendations](#conclusion-and-recommendations)
*  [Future Improvements](#future-improvements).
  
---

##  Project Overview

This project simulates a real-world e-commerce supply chain analytics system. Using historical order and cancellation data, I developed a MySQL-based data solution to help business 

stakeholders understand:

* Order performance trends
* Cancellation rates (CAL) and root causes
* Product and region-level failure points
* Vendor and customer behavior insights

---

##  Dataset Description

> Source: https://www.kaggle.com/datasets/annelee1/supply-chain-cel-dataset

* canceled_test.csv: Contains information on canceled orders.

 Columns: ORDER_NO, DATE, LINE, CUSTOMER_NO, ITEM, NC_ORDER, NC_SHIP

* sales_test.csv: Contains information on successfully fulfilled orders.

 Columns: ORDER_NO, DATE, LINE, CUSTOMER_NO, ITEM, NS_ORDER, NS_SHIP

---

## Database Schema

**Tables created**:

* `orders`: All delivered orders
  
* `cancellations`: Canceled orders 

Optional future expansions:

* `customers`, `products`, `vendors`, `inventory`

---


## Data cleaning

1. Changed the date format for both the tables

```sql

  ALTER TABLE orders ADD COLUMN proper_date DATE;
   
  SET SQL_SAFE_UPDATES = 0;

  UPDATE orders
  
  SET proper_date = STR_TO_DATE(TRIM(SUBSTRING(`DATE`, LOCATE(',', `DATE`) + 1)), '%M %d, %Y');

  ALTER TABLE orders DROP COLUMN `DATE`;

  ALTER TABLE orders CHANGE proper_date `DATE` DATE;
  
  ```
  
2 Finding Duplicates

```sql

WITH DUPLICATE AS (
    SELECT  *, 
    ROW_NUMBER() OVER (
            PARTITION BY ORDER_NO, LINE, DATE, CUSTOMER_NO 
            ORDER BY ORDER_NO DESC
        ) AS rn
    FROM 
        cancellations
)

SELECT  rn FROM  DUPLICATE WHERE   rn>1;

```

![Screenshot 2025-06-16 111652](https://github.com/user-attachments/assets/7c879fe8-6358-4be3-b46b-c1cd9c8f5dc6)

Results : No duplicates where found in both the tables.


## Key Metrics

* Monthly Order Volume helps track trends over time and identify seasonal patterns in customer demand.

* Cancellation Rate provides insight into the overall health of operations and can indicate issues with fulfillment or customer satisfaction.

* Most Canceled Categories highlight potential problems with specific products or vendors, helping to target improvements.

* Best-Selling Products are useful for demand forecasting, optimizing inventory, and guiding marketing and sales strategies.



---

##  

* `JOIN`, `GROUP BY`, `CASE`, `DATE_FORMAT`, `DENSE_RANK`
* Aggregations & KPI queries
* SQL **Views** for reusable logic:

  * `monthly_order_volume`
  * `monthly_cancellation_rate`
* **Stored Procedure** for generating monthly KPI summaries
---

##  Tools Used

* MySQL Workbench
* CSV file import
* (Optional: Power BI / Tableau)

---



###  ** Business Questions**

---

### 1. **How is the cancellation rate trending over time?**

**Business Understanding**: Trends in cancellation rates are crucial for understanding if operational or logistical issues are improving or worsening over time.

**SQL Query**:

```sql
SELECT 
    DATE_FORMAT(o.Date, '%Y-%m') AS order_month,
    COUNT(c.ORDER_NO) AS canceled_orders,
    COUNT(o.ORDER_NO) AS total_orders,
    ROUND(COUNT(c.ORDER_NO) * 100.0 / COUNT(o.ORDER_NO), 2) AS cal_percent
FROM orders o
LEFT JOIN cancellations c ON o.ORDER_NO = c.ORDER_NO
GROUP BY order_month
ORDER BY order_month;
```


![Screenshot 2025-06-16 112634](https://github.com/user-attachments/assets/d2e36201-80ea-436b-9770-30300365f64f)


**Business Insight**: The **cancellation rate** has increased, it may indicate operational issues, stock shortages, or delays.

---


### 2. **Which products have the highest cancellation rates?**

**Business understanding**: Identifying products with high cancellation rates helps with product sourcing and understanding demand mismatch.

**SQL Query**:

```sql
SELECT 
    o.ITEM,
    COUNT(c.ORDER_NO) AS canceled_orders,
    COUNT(o.ORDER_NO) AS total_orders,
    ROUND(COUNT(c.ORDER_NO) * 100.0 / COUNT(o.ORDER_NO), 2) AS cal_percent
FROM orders o
LEFT JOIN cancellations c ON o.ORDER_NO = c.ORDER_NO AND o.ITEM = c.ITEM
GROUP BY o.ITEM
ORDER BY cal_percent desc
LIMIT 10;

```


![Screenshot 2025-06-16 113942](https://github.com/user-attachments/assets/7893a327-9b5d-46cc-890d-93d6506dd8a5)

**Business Insight**: These items are highly reliable, with cancellation rates under 1.1%, indicating strong inventory availability, demand consistency, or effective logistics.

The item 10098739, while having the highest volume among these, still maintains only 0.42% cancellations — showing it's both popular and dependable.

Low cancellation items are ideal for:

Promotions and bundling (less customer dissatisfaction risk),

Core catalog features, 

Inventory investment focus.

---

### 3. **Are cancellations more frequent in specific days ?**

**Business understanding**: Important for identifying operational or behavioral patterns.

```sql
SELECT 
    DAYNAME(o.DATE) AS order_day,
    COUNT(c.ORDER_NO) AS canceled_orders,
    COUNT(o.ORDER_NO) AS total_orders,
    ROUND(COUNT(c.ORDER_NO) * 100.0 / COUNT(o.ORDER_NO), 2) AS cal_percent
FROM orders o
LEFT JOIN cancellations c 
    ON o.ORDER_NO = c.ORDER_NO
GROUP BY order_day
ORDER BY cal_percent DESC;
```


![Screenshot 2025-06-16 114533](https://github.com/user-attachments/assets/6fdbe6f1-f3db-4498-a01f-b483345e3e52)

**Business Insight**: Tuesday should be closely reviewed — the high cancellation rate suggests process inefficiencies, possibly in order processing or stock updating from weekend lags.

End-of-week orders (Thursdays & Fridays) are more reliable in fulfillment, indicating a good window for promotions or bulk orders.

Consider investigating operational, inventory, or delivery issues affecting early-week performance

---

### 4. **Which customers have the highest cancellation rate?**

**Business Understanding**: Understanding high-risk customers can help prevent future cancellations by offering promotions, improving customer service, or adjusting inventory.

```sql
SELECT 
    o.CUSTOMER_NO,
    COUNT(c.ORDER_NO) AS canceled_orders,
    COUNT(o.ORDER_NO) AS total_orders,
    ROUND(COUNT(c.ORDER_NO) * 100.0 / COUNT(o.ORDER_NO), 2) AS cal_percent
FROM orders o
LEFT JOIN cancellations c 
    ON o.ORDER_NO = c.ORDER_NO
GROUP BY o.CUSTOMER_NO
HAVING cal_percent > 10
ORDER BY cal_percent DESC
limit 10;

```


![Screenshot 2025-06-16 115127](https://github.com/user-attachments/assets/57c9c331-a7ab-43d5-927f-72a528aaa1d5)

**Business Insight**: These  subset of customers accounts for a significant share of total cancellations. These may be frequent buyers with changing needs or customers facing delivery issues. Understanding this group can help reduce operational costs and improve customer support.



---

5. Top selliing products

**Business Understanding** : Helps with inventory planning, promotion focus, and identifying popular products.

```sql

  SELECT 
    ITEM,
    SUM(NS_SHIP) AS total_units_sold
FROM orders
GROUP BY ITEM
ORDER BY total_units_sold DESC
LIMIT 10;

```

![Screenshot 2025-06-16 120215](https://github.com/user-attachments/assets/4168993c-351b-4c1a-b2b6-89591cd928d9)

**Business Insights** : Item 10098739 is the clear leader, selling over 27K units — likely a flagship product. It should be closely monitored for inventory and customer satisfaction.

Many of these items (e.g., 1041106, 1040765, 1041105) appear to have consistent demand, signaling potential for bulk purchasing or long-term supplier contracts.

Items in the bottom half of this list (e.g., 1041108, 1038323) could be evaluated for cross-selling or inventory prioritization.

---

##  **SQL Views and Stored Procedures**

### 🧾 1. SQL View: Monthly Order Volume

```sql
CREATE VIEW monthly_order_volume AS
SELECT 
    DATE_FORMAT(DATE, '%Y-%m') AS order_month,
    COUNT(DISTINCT ORDER_NO) AS total_orders
FROM orders
GROUP BY order_month
ORDER BY order_month;
```

> Use this view to report month-by-month trends in total order activity.
```
SELECT * FROM sales.monthly_order_volume;

```

![Screenshot 2025-06-16 120611](https://github.com/user-attachments/assets/dc4e2380-700b-4e9d-9a4e-74a01b41b3fa)

---

### 2. SQL View: Monthly Cancellation Rate (CAL)

```sql
CREATE VIEW monthly_cancellation_rate AS
SELECT 
    DATE_FORMAT(o.DATE, '%Y-%m') AS order_month,
    COUNT(c.ORDER_NO) AS canceled_orders,
    COUNT(o.ORDER_NO) AS total_orders,
    ROUND(COUNT(c.ORDER_NO) * 100.0 / COUNT(o.ORDER_NO), 2) AS cal_percent
FROM orders o
LEFT JOIN cancellations c ON o.ORDER_NO = c.ORDER_NO
GROUP BY order_month
ORDER BY order_month;

```

---

### ⚙️ 3. Stored Procedure: Get KPI Summary for a Specific Month

```sql
DELIMITER //

CREATE PROCEDURE kpi_summary_by_month(IN input_month VARCHAR(7))
BEGIN
    DECLARE total_orders INT DEFAULT 0;
    DECLARE total_cancellations INT DEFAULT 0;

    -- Calculate total orders for the month
    SELECT COUNT(DISTINCT ORDER_NO)
    INTO total_orders
    FROM orders
    WHERE DATE_FORMAT(DATE, '%Y-%m') = input_month;

    -- Calculate total cancellations for the month
    SELECT COUNT(DISTINCT c.ORDER_NO)
    INTO total_cancellations
    FROM cancellations c
    JOIN orders o ON c.ORDER_NO = o.ORDER_NO
    WHERE DATE_FORMAT(o.DATE, '%Y-%m') = input_month;

    -- Final output
    SELECT 
        input_month AS month,
        total_orders,
        total_cancellations,
        ROUND(IFNULL(total_cancellations * 100.0 / NULLIF(total_orders, 0), 0), 2) AS cancellation_rate;
END //kpi_summary_by_monthkpi_summary_by_month

DELIMITER ;

call sales.kpi_summary_by_month('2017-01');

```

![Screenshot 2025-06-16 121644](https://github.com/user-attachments/assets/4e2a8dc2-6e1e-456b-a359-e0934e407233)

---


##  Business Insights 

1 Top Canceled Products

Products like 10135358 and 1041106 have the highest cancellation rates, exceeding 50%. This suggests issues related to inventory stockouts, misaligned customer expectations, or unreliable 

vendors.

2 Customer Behavior

A small group of customers contributes disproportionately to total cancellations. These may require targeted support, better communication, or customer education to reduce churn and failed 

transactions.

3 Day-wise Cancellation Trend

Cancellations are more frequent during weekends (especially Friday and Saturday), potentially due to limited logistics support or customer unavailability during deliveries.

4 Top Selling Products

Products with the highest sales volumes tend to have lower cancellation rates, showing effective alignment between supply and demand and a well-functioning fulfillment process for core items.




---


### 5.  Recommendation 

*  Flag vendors linked to high cancellation rates for review
  
*  Focus on reliable brands with high order volume and low cancellations

*  Automate weekly KPI monitoring using views and stored procedures

---

## Future Improvements

* Normalize schema with separate `products`, `vendors`, `regions` tables

* Build live dashboards (Power BI / Tableau) from MySQL views

* Set up alerts for high cancellation spikes using triggers or Python scripts

* Use historical data to build predictive models (CAL forecasting)

---

## Author

**Yashi Agrawal**
www.linkedin.com/in/yashi-agrawal-| agrawalyashi774@gmail.com


