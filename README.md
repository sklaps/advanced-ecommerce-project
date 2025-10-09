# EcomPulse — Advanced eCommerce Analytics (SK)

> Turning raw event logs into business decisions — SQL + Excel charts + RFM segmentation.  
> Portfolio case study: data cleaning, funnel & hourly analysis, top-product & revenue insights, customer segmentation.

---

## TL;DR
- **What I did:** Cleaned event logs, built funnel and hourly analyses, identified top products & brand revenue, and created an RFM segmentation — all using SQL (MySQL) and Excel visualizations.  
- **Why it matters:** Shows which brands drive revenue vs. volume, when to run promotions, and which customers to target for retention.  
- **Result highlights:** Apple leads revenue (~₹211,843), Samsung leads unit sales (364 purchases); 55 VIP customers (RFM = 10).  
- **Skills shown:** SQL (data cleaning, window functions, CTEs), Excel charting, business storytelling.

---

## Quick project facts
- **Source table:** `new_project.small_file`  
- **Total events (sample):** **93,912**  
- **Distinct users:** **19,226**  
- **Distinct products:** **19,908**  
- **Event types:** `view` (91,268), `purchase` (1,535), `cart` (1,109)  
- **Time window:** `2019-10-01 00:00:00` → `2019-10-01 04:22:05` (peak activity at 03:00 UTC)
---
## 🧭 Project Overview

The goal of this project was to understand how customers interact with an eCommerce platform — from viewing products to purchasing — and discover actionable business patterns.

Using real event logs, I explored:
- 🧹 **Data Cleaning:** Fixing null values, removing missing brands, validating event types.
- 🔍 **Exploratory Analysis:** Which brands and products perform best, how users behave by hour, and how many drop off before purchasing.
- 💰 **Revenue Insights:** Identifying top-earning brands and products.
- 👥 **Customer Segmentation:** Creating an RFM model (Recency, Frequency, Monetary) to find VIPs and at-risk customers.

This work replicates what a junior data analyst would perform in a real-world business environment.

---

## ❓ Business Questions Answered

1. **Event Distribution:** How are different user actions (`view`, `cart`, `purchase`) distributed?  
2. **Conversion Funnel:** What percent of users move from viewing → adding to cart → purchasing?  
3. **Top Products:** Which products are purchased most often?  
4. **Top Brands by Average Price:** Which brands have the highest average selling price?  
5. **Top Brands by Revenue:** Which brands drive the most total revenue?  
6. **Hourly Activity:** During which hours do purchases peak?  
7. **Customer Segmentation (RFM):** Who are the VIP, loyal, and inactive customers?

Each question was solved using SQL and visualized in Excel for clearer business communication.
---

## 🧹 Data & Cleaning

### 📦 Dataset Details
- **Source:** Kaggle — eCommerce Behavior Data (October 2019 sample)
- **Size (after filtering):** 93,912 rows
- **File Used:** `small_file.csv`
- **Database Table:** `new_project.small_file`

### 🧾 Columns Used
| Column | Description |
|--------|--------------|
| event_time | Timestamp of the user action |
| event_type | `view`, `cart`, or `purchase` |
| product_id | Unique product identifier |
| category_code | Product category |
| brand | Brand name |
| price | Product price |
| user_id | Unique customer ID |
| user_session | Unique session ID |

---

### 🧼 Cleaning Summary

| Check | Finding | Action |
|--------|----------|--------|
| Missing `event_type` | 0 | ✅ No issue |
| Missing `product_id` | 0 | ✅ No issue |
| Missing `brand` | 13,509 | 🔄 Kept as “Unknown” |
| Missing `price` | 113 | 🔄 Dropped (negligible) |
| Unknown brands | 0 | ✅ No issue |

---

### 🧮 Basic Data Health

| Metric | Result |
|---------|---------|
| Total rows | 93,912 |
| Distinct users | 19,226 |
| Distinct products | 19,908 |
| Event types | View: 91,268 • Cart: 1,109 • Purchase: 1,535 |
| Date range | 2019-10-01 00:00 → 2019-10-01 04:22 |
| Peak activity hour | 03:00 UTC |

---

💡 *Why this matters:*  
A clean dataset ensures correct insights. Dropping only 113 rows (0.12%) preserved integrity, while unknown brands were labeled as “Unknown” to maintain completeness.
---

## 📊 Analysis & SQL Queries

---

### 🧮 Q1. How are user events distributed? (Views, Carts, Purchases)

```sql
SELECT event_type, COUNT(*) AS total_events
FROM new_project.small_file
GROUP BY event_type
ORDER BY total_events DESC;



Insights

Most actions are view events — typical for eCommerce browsing.

Only ~1.6% of views lead to purchases → potential for funnel improvement.

Marketing teams can target high-view, low-purchase categories to increase conversion.

**Q2. What percent of users move from viewing → cart → purchasing?**********

SELECT 
    (SELECT COUNT(DISTINCT user_id) FROM new_project.small_file WHERE event_type='view') AS total_views,
    (SELECT COUNT(DISTINCT user_id) FROM new_project.small_file WHERE event_type='cart') AS total_carts,
    (SELECT COUNT(DISTINCT user_id) FROM new_project.small_file WHERE event_type='purchase') AS total_purchases;



💡 Insights

View-to-cart conversion: ~1.2%

Cart-to-purchase conversion: ~138% (repeat buyers or multiple items).

Funnel shows where users drop off — ideal area for UX or pricing optimization.

Q3. Which are the top 10 most purchased products?

SELECT product_id, COUNT(*) AS purchase_count
FROM new_project.small_file
WHERE event_type = 'purchase'
GROUP BY product_id
ORDER BY purchase_count DESC
LIMIT 10;


💡 Insights

Samsung and Apple dominate product-level sales.

Product IDs like 1004856 & 1004767 are top sellers.

These can be featured in ad campaigns or bundles.

Q4. Which brands have the highest average selling price?

SELECT brand, ROUND(AVG(price), 2) AS avg_price
FROM new_project.small_file
WHERE event_type = 'purchase' AND brand IS NOT NULL
GROUP BY brand
ORDER BY avg_price DESC
LIMIT 10;



Insights

Premium brands like Mercury and Apple show high average prices.

Indicates their luxury positioning in the market.

Helps pricing teams understand product tier gaps.

Q5. Which brands drive the most total revenue?

SELECT brand, 
       ROUND(SUM(price), 2) AS total_revenue, 
       COUNT(*) AS total_purchases
FROM new_project.small_file
WHERE event_type = 'purchase' AND brand IS NOT NULL
GROUP BY brand
ORDER BY total_revenue DESC
LIMIT 10;


💡 Insights

Apple generates the most total revenue (₹211,843), while Samsung leads in total purchases.

Apple’s fewer but high-value transactions show strong brand power.

Suggests upselling opportunities for mid-tier brands.


## 🧠 RFM Segmentation — Understanding Customer Value

### 🧩 What is RFM?
RFM (Recency, Frequency, Monetary) is a marketing analytics technique used to measure customer engagement and loyalty:
- **Recency** → How recently the customer made a purchase  
- **Frequency** → How often they purchase  
- **Monetary** → How much money they spend  

It helps identify **VIPs**, **loyal customers**, and **churn risks**.

---

### 🧮 SQL Query Used

```sql
WITH purchases AS (
    SELECT user_id,
           COUNT(*) AS frequency,
           SUM(price) AS monetary,
           TIMESTAMPDIFF(HOUR, MAX(event_time), (SELECT MAX(event_time) FROM new_project.small_file)) AS recency_hours
    FROM new_project.small_file
    WHERE event_type = 'purchase'
    GROUP BY user_id
)
SELECT *,
       NTILE(4) OVER (ORDER BY recency_hours ASC) AS recency_score,
       NTILE(4) OVER (ORDER BY frequency DESC) AS frequency_score,
       NTILE(4) OVER (ORDER BY monetary DESC) AS monetary_score,
       CONCAT(
           NTILE(4) OVER (ORDER BY recency_hours ASC),
           NTILE(4) OVER (ORDER BY frequency DESC),
           NTILE(4) OVER (ORDER BY monetary DESC)
       ) AS rfm_code,
       (
           NTILE(4) OVER (ORDER BY recency_hours ASC) +
           NTILE(4) OVER (ORDER BY frequency DESC) +
           NTILE(4) OVER (ORDER BY monetary DESC)
       ) AS rfm_score
FROM purchases
LIMIT 100;


