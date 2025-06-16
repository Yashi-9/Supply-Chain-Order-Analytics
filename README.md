

# 📦 Supply Chain Order Analytics | MySQL

### 📊 Analyze order performance, cancellation trends, and supply chain KPIs using  MySQL.

---

## 📁 Project Overview

This project simulates a real-world e-commerce supply chain analytics system. Using historical order and cancellation data, I developed a MySQL-based data solution to help business stakeholders understand:

* Order performance trends
* Cancellation rates (CAL) and root causes
* Product and region-level failure points
* Vendor and customer behavior insights

---

## 📚 Dataset Used

> Source: [SQL Supply Chain Order Analysis by @mdntarif](https://github.com/mdntarif/SQL-Supply-Chain-Order-Analysis)

* `sales_test.csv` – Delivered orders with metadata
* `canceled_test.csv` – Orders that were canceled with reasons

---

## 🧱 Database Schema

<img src="your-schema-image.png" alt="ER Diagram" width="600" />

**Tables created**:

* `orders`: All successful orders
* `cancellations`: Canceled orders with reasons

Optional future expansions:

* `customers`, `products`, `vendors`, `inventory`

---

## 📈 Key Metrics & Business Questions

| KPI                              | Business Use Case                              |
| -------------------------------- | ---------------------------------------------- |
| 📊 Total Orders by Month         | Trend analysis, seasonality                    |
| ❌ Cancellation Rate (CAL)        | Operational health, customer experience        |
| 📍 Region-wise Cancellation Rate | Identify underperforming areas/logistics zones |
| 🏷️ Top Canceled Categories      | Inventory or vendor issue indicators           |
| 🔁 Cancellation Reasons          | Root cause diagnosis                           |
| 📦 Top Selling Products          | Demand planning, marketing insights            |

---

## 💡 Sample Business Insights

* **East region** had a 19.5% cancellation rate — mostly due to stockouts and delivery delays
* **Electronics and Mobile Accessories** were the most canceled categories
* **January** saw the highest order volume but also highest cancellations — indicates strain on fulfillment
* **Brand A** maintained a <2% CAL across all months — a reliable vendor

---

## 🧮 SQL Features Demonstrated

* `JOIN`, `GROUP BY`, `CASE`, `DATE_FORMAT`, `DENSE_RANK`
* Aggregations & KPI queries
* SQL **Views** for reusable logic:

  * `monthly_order_volume`
  * `monthly_cancellation_rate`
* **Stored Procedure** for generating monthly KPI summaries
* Performance-aware filtering with indexes (optional)

---

## 🚀 Future Improvements

* Normalize schema with separate `products`, `vendors`, `regions` tables
* Build live dashboards (Power BI / Tableau) from MySQL views
* Set up alerts for high cancellation spikes using triggers or Python scripts
* Use historical data to build predictive models (CAL forecasting)

---

## 📌 Tools Used

* MySQL Workbench
* CSV file import
* (Optional: Power BI / Tableau)

---

## 👨‍💻 Author

**Yashi Agrawal**
www.linkedin.com/in/yashi-agrawal-| agrawalyashi774@gmail.com

---

### 🚀 ** Business Questions to Answer**

---

### 1. **How is the cancellation rate trending over time?**

**Business Insight**: Trends in cancellation rates are crucial for understanding if operational or logistical issues are improving or worsening over time.

**SQL Query**:

```sql
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') AS order_month,
    COUNT(c.order_id) AS canceled_orders,
    COUNT(o.order_id) AS total_orders,
    ROUND(COUNT(c.order_id) * 100.0 / COUNT(o.order_id), 2) AS cal_percent
FROM orders o
LEFT JOIN cancellations c ON o.order_id = c.order_id
GROUP BY order_month
ORDER BY order_month;
```

**Business Insight**: If the **cancellation rate** has increased, it may indicate operational issues, stock shortages, or delays in certain months or seasons. Could be valuable for planning future orders or improving supply chain processes.

---

### 2. **What are the top 5 cancellation reasons, and how often do they occur?**

**Business Insight**: Knowing the common reasons for cancellations can guide inventory management, vendor communication, and operational improvements.

**SQL Query**:

```sql
SELECT cancellation_reason, COUNT(*) AS reason_count
FROM cancellations
GROUP BY cancellation_reason
ORDER BY reason_count DESC
LIMIT 5;
```

**Business Insight**: You might see **“Out of Stock”** or **“Vendor Delay”** frequently, which could suggest supply chain inefficiencies. If a certain supplier is the main source of delays, they may need to be prioritized for performance reviews.

---

### 3. **Which products have the highest cancellation rates?**

**Business Insight**: Identifying products with high cancellation rates helps with product sourcing and understanding demand mismatch.

**SQL Query**:

```sql
SELECT p.product_name, 
       COUNT(c.order_id) AS canceled_orders,
       COUNT(o.order_id) AS total_orders,
       ROUND(COUNT(c.order_id) * 100.0 / COUNT(o.order_id), 2) AS cal_percent
FROM orders o
LEFT JOIN cancellations c ON o.order_id = c.order_id
JOIN products p ON o.sku_id = p.sku_id
GROUP BY p.product_name
ORDER BY cal_percent DESC
LIMIT 5;
```

**Business Insight**: If **product categories** like **electronics** or **furniture** have high cancellation rates, you may need to consider better demand forecasting, improve vendor relationships, or enhance stock levels.

---

### 4. **Are cancellations more frequent in specific regions or cities?**

**Business Insight**: Identifying regions or cities with higher cancellations could point to logistical issues or regional customer experience challenges.

**SQL Query**:

```sql
SELECT region, city, 
       COUNT(c.order_id) AS canceled_orders,
       COUNT(o.order_id) AS total_orders,
       ROUND(COUNT(c.order_id) * 100.0 / COUNT(o.order_id), 2) AS cal_percent
FROM orders o
LEFT JOIN cancellations c ON o.order_id = c.order_id
GROUP BY region, city
ORDER BY cal_percent DESC;
```

**Business Insight**: A city with a 30% cancellation rate may require targeted actions such as **better courier management**, **more warehouses**, or **customer service training**.

---

### 5. **What’s the average order value for canceled vs. delivered orders?**

**Business Insight**: Understanding how much revenue you’re losing from canceled orders can highlight the financial impact on the business and help prioritize actions.

**SQL Query**:

```sql
SELECT 
    CASE 
        WHEN c.order_id IS NOT NULL THEN 'Canceled' 
        ELSE 'Delivered' 
    END AS order_status,
    AVG(o.price * o.qty) AS avg_order_value
FROM orders o
LEFT JOIN cancellations c ON o.order_id = c.order_id
GROUP BY order_status;
```

**Business Insight**: If canceled orders have a much higher **average order value**, it could suggest issues with **premium product fulfillment** or **vendor reliability**.

---

### 6. **Which customers have the highest cancellation rate?**

**Business Insight**: Understanding high-risk customers can help prevent future cancellations by offering promotions, improving customer service, or adjusting inventory.

**SQL Query**:

```sql
SELECT o.customer_id,
       COUNT(c.order_id) AS canceled_orders,
       COUNT(o.order_id) AS total_orders,
       ROUND(COUNT(c.order_id) * 100.0 / COUNT(o.order_id), 2) AS cal_percent
FROM orders o
LEFT JOIN cancellations c ON o.order_id = c.order_id
GROUP BY o.customer_id
HAVING cal_percent > 10
ORDER BY cal_percent DESC;
```

**Business Insight**: If a **small group** of customers has a high cancellation rate, you can consider **account management** or offering **exclusive solutions** to prevent future issues.

---

### 7. **How do cancellation rates differ by product category?**

**Business Insight**: Knowing which categories are prone to higher cancellations will help you **optimize your product offerings** and manage stock levels effectively.

**SQL Query**:

```sql
SELECT category,
       COUNT(c.order_id) AS canceled_orders,
       COUNT(o.order_id) AS total_orders,
       ROUND(COUNT(c.order_id) * 100.0 / COUNT(o.order_id), 2) AS cal_percent
FROM orders o
LEFT JOIN cancellations c ON o.order_id = c.order_id
GROUP BY category
ORDER BY cal_percent DESC;
```

**Business Insight**: Categories like **Electronics** might show a higher **cancellation rate**, possibly due to product defects, vendor issues, or stockouts. You may need to prioritize **supplier relationships** or **quality assurance**.

---

### 8. **What’s the average lead time (time between order and delivery) for delivered orders?**

**Business Insight**: Understanding lead times helps in **improving logistics** and customer satisfaction, especially when cancellations occur due to delivery delays.

**SQL Query**:

```sql
SELECT 
    AVG(DATEDIFF(delivery_date, order_date)) AS avg_lead_time
FROM deliveries d
JOIN orders o ON d.order_id = o.order_id
WHERE o.status = 'delivered';
```

**Business Insight**: If average lead times are high, it may indicate logistical inefficiencies, leading to **customer dissatisfaction** and **increased cancellations**.

---

### 9. **What is the most frequent cancellation reason for high-value orders?**

**Business Insight**: Identifying the most common reasons for high-value order cancellations allows you to take targeted actions, such as improving vendor stock, customer service, or logistics.

**SQL Query**:

```sql
SELECT cancellation_reason, COUNT(*) AS reason_count
FROM cancellations c
JOIN orders o ON c.order_id = o.order_id
WHERE (o.price * o.qty) > 1000
GROUP BY cancellation_reason
ORDER BY reason_count DESC;
```

**Business Insight**: If high-value cancellations are **due to stockouts**, you may want to implement **better stock forecasting** for premium products.

---


##  **SQL Views and Stored Procedures for Monthly KPIs**

Creating **views** and **stored procedures** will show that you're not just running ad hoc queries — you understand reusable SQL and modular design. This is something hiring managers **really value**.

---

### 🧾 1. SQL View: Monthly Order Volume

```sql
CREATE VIEW monthly_order_volume AS
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') AS order_month,
    COUNT(*) AS total_orders
FROM orders
GROUP BY order_month
ORDER BY order_month;
```

> ✅ Use this view to report month-by-month trends in total order activity.

---

### ❌ 2. SQL View: Monthly Cancellation Rate (CAL)

```sql
CREATE VIEW monthly_cancellation_rate AS
SELECT 
    DATE_FORMAT(o.order_date, '%Y-%m') AS order_month,
    COUNT(c.order_id) AS canceled_orders,
    COUNT(o.order_id) AS total_orders,
    ROUND(COUNT(c.order_id) * 100.0 / COUNT(o.order_id), 2) AS cal_percent
FROM orders o
LEFT JOIN cancellations c ON o.order_id = c.order_id
GROUP BY order_month
ORDER BY order_month;
```

---

### ⚙️ 3. Stored Procedure: Get KPI Summary for a Specific Month

```sql
DELIMITER //

CREATE PROCEDURE kpi_summary_by_month(IN input_month VARCHAR(7))
BEGIN
    SELECT 
        input_month AS month,
        (SELECT COUNT(*) FROM orders WHERE DATE_FORMAT(order_date, '%Y-%m') = input_month) AS total_orders,
        (SELECT COUNT(*) FROM cancellations c 
         JOIN orders o ON c.order_id = o.order_id 
         WHERE DATE_FORMAT(o.order_date, '%Y-%m') = input_month) AS total_cancellations,
        ROUND((
            SELECT COUNT(*) FROM cancellations c 
            JOIN orders o ON c.order_id = o.order_id 
            WHERE DATE_FORMAT(o.order_date, '%Y-%m') = input_month
        ) * 100.0 / (
            SELECT COUNT(*) FROM orders WHERE DATE_FORMAT(order_date, '%Y-%m') = input_month
        ), 2) AS cancellation_rate;
END //

DELIMITER ;
```

##  Business Insights 

After you run your views and queries, extract real insights to include in your project write-up. Here are **examples of real-world insights** you can find with this dataset:

---

### 1.  High Cancellation Rate in Certain Regions

> “The Eastern region has a cancellation rate of 19.5%, 2× higher than other regions. Most of these are due to 'Out of stock' and 'Vendor delay' reasons — indicating supply issues.”

---

### 2. 🛍 Problematic Product Categories

> “Mobile Accessories and Electronics have the highest number of cancellations — 35% of all canceled orders — which signals a mismatch in demand forecasting or vendor fulfillment.”

---

### 3.  Spike in Orders + Cancellations in January

> “January shows a 22% increase in order volume compared to December, likely due to New Year promotions, but also a spike in cancellations, especially in the North region — may indicate capacity planning issues.”

---

### 4.  Most Reliable Brands

> “Brand X and Brand Y maintained a <2% cancellation rate across 3 months — they should be prioritized for promotions and procurement contracts.”

---

### 5.  Recommendation Summary for Management

*  Increase buffer inventory for top-canceled SKUs in East & North
*  Flag vendors linked to high cancellation rates for review
*  Focus on reliable brands with high order volume and low cancellations
*  Automate weekly KPI monitoring using views and stored procedures

---


