# advanced-ecommerce-project
Data analytics project using SQL + Excel on eCommerce behavior


# EcomPulse — Advanced eCommerce Analytics (SK)

Short pitch: Advanced analysis of product and customer behaviour from event logs (views, carts, purchases). Reproducible SQL + Excel charts + RFM segmentation for business action.

# EcomPulse — Advanced eCommerce Analytics (SK)

Short pitch: Advanced analysis of product and customer behaviour from event logs (views, carts, purchases). Reproducible SQL + Excel charts + RFM segmentation for business action.

Quick facts (dataset)

Source table: new_project.small_file

Total events: 93,912

Distinct users: 19,226

Distinct products: 19,908

Event types: view (91,268), purchase (1,535), cart (1,109)

Time window in sample: 2019-10-01 00:00:00 → 2019-10-01 04:22:05 (hourly peak at 03:00 UTC)

Project structure (recommended)

ecommerce-advanced-project/
├── README.md
├── sql/
│   ├── 00_data_cleaning.sql
│   ├── 01_analysis_queries.sql
│   └── 02_rfm.sql
├── charts/                    # upload chart PNGs here
│   ├── top_products.png
│   ├── avg_price_by_brand.png
│   ├── revenue_by_brand.png
│   ├── hourly_activity.png
│   └── rfm_distribution.png
├── data_sample/               # optional small sample CSV
└── docs/
    └── EcomPulse_Report_SK.pdf
**What I cleaned**

-- 1) Make brand blanks explicit
UPDATE new_project.small_file
SET brand = 'Unknown'
WHERE TRIM(brand) = '';

-- 2) Replace price 0 with NULL (so averages ignore them)
UPDATE new_project.small_file
SET price = NULL
WHERE price = 0;

SELECT
  COUNT(*) AS total_rows,
  COUNT(DISTINCT user_id) AS distinct_users,
  COUNT(DISTINCT product_id) AS distinct_products
FROM new_project.small_file;
(result: 93912 | 19226 | 19908)

Analyses & exact SQL

Below are the main analyses we ran — each section contains the SQL I used and a short explanation. Save these queries in sql/01_analysis_queries.sql.

1) Funnel & conversion rates

SELECT
  SUM(CASE WHEN event_type = 'view' THEN 1 ELSE 0 END) AS total_views,
  SUM(CASE WHEN event_type = 'cart' THEN 1 ELSE 0 END) AS total_carts,
  SUM(CASE WHEN event_type = 'purchase' THEN 1 ELSE 0 END) AS total_purchases,
  ROUND(SUM(CASE WHEN event_type = 'cart' THEN 1 ELSE 0 END) /
        SUM(CASE WHEN event_type = 'view' THEN 1 ELSE 0 END) * 100, 2) AS view_to_cart_percent,
  ROUND(SUM(CASE WHEN event_type = 'purchase' THEN 1 ELSE 0 END) /
        SUM(CASE WHEN event_type = 'cart' THEN 1 ELSE 0 END) * 100, 2) AS cart_to_purchase_percent,
  ROUND(SUM(CASE WHEN event_type = 'purchase' THEN 1 ELSE 0 END) /
        SUM(CASE WHEN event_type = 'view' THEN 1 ELSE 0 END) * 100, 2) AS overall_conversion_percent
FROM new_project.small_file;

Result you got:
views=91,268; carts=1,109; purchases=1,535
view→cart = 1.22% ; cart→purchase = 138.41% (see note) ; overall = 1.68%

Note about cart_to_purchase > 100%: purchases exceed cart events. Possible causes:

Some purchases recorded without prior cart event (direct buy, buy-now flow).

Cart events missing in logs.
To check purchases without carts per session:

-- sessions where a purchase occurred but no cart recorded
SELECT user_session, COUNT(DISTINCT event_type) AS types_present,
       SUM(event_type='cart') AS cart_count, SUM(event_type='purchase') AS purchase_count
FROM new_project.small_file
GROUP BY user_session
HAVING purchase_count > 0 AND cart_count = 0
LIMIT 20;

2) Hourly activity (to plan ops & marketing)

   SELECT 
  HOUR(STR_TO_DATE(REPLACE(event_time, ' UTC', ''), '%Y-%m-%d %H:%i:%s')) AS hour_of_day,
  COUNT(*) AS total_events
FROM new_project.small_file
GROUP BY hour_of_day
ORDER BY hour_of_day;

Your results: hour 0: 1,083 | 1: 121 | 2: 22,886 | 3: 49,409 (peak) | 4: 20,413

Chart file: charts/hourly_activity.png

Insight: Peak activity between 02:00–04:00 UTC, highest at 03:00 — consider timezone effects and schedule campaigns/scale servers accordingly.

3) Top 10 purchased products

   SELECT product_id, brand, COUNT(*) AS total_purchases
FROM new_project.small_file
WHERE event_type = 'purchase'
GROUP BY product_id, brand
ORDER BY total_purchases DESC
LIMIT 10;


Top 10 you found (summary):
Product 1004856 (samsung) — 57 purchases; 1004767 (samsung) — 53; 1002544 (apple) — 45; … top-10 shows Samsung heavy presence.


Chart file: charts/top_products.png
Insight (caption for chart): Samsung dominates 6/10 top sellers; Apple strong in revenue.

4) Average price by brand (price positioning)

   SELECT brand, ROUND(AVG(price), 2) AS avg_price
FROM new_project.small_file
WHERE event_type = 'purchase' AND brand IS NOT NULL
GROUP BY brand
ORDER BY avg_price DESC
LIMIT 10;

Top avg prices: mercury 2499.68, pulser 1369.38, apple 773.15, ...

Chart file: charts/avg_price_by_brand.png
Insight: Mercury and Pulser are premium; Apple mid-high; several brands cluster mid-range.


5) Revenue by brand (total revenue)

   SELECT brand,
       ROUND(SUM(price), 2) AS total_revenue,
       COUNT(*) AS total_purchases
FROM new_project.small_file
WHERE event_type = 'purchase' AND brand IS NOT NULL
GROUP BY brand
ORDER BY total_revenue DESC
LIMIT 10;

Top revenue: apple (211,842.88 from 274 purchases), samsung (99,079.31, 364 purchases), then Unknown, xiaomi, huawei...

Chart file: charts/revenue_by_brand.png
Insight: Apple leads revenue (premium → high AOV), Samsung leads units sold (volume strategy).

6) RFM segmentation (Recency / Frequency / Monetary)

Preview query used (recency in hours because dataset is short):

WITH purchases AS (
  SELECT
    user_id,
    COUNT(*) AS frequency,
    ROUND(SUM(price),2) AS monetary,
    MAX(STR_TO_DATE(REPLACE(event_time, ' UTC', ''), '%Y-%m-%d %H:%i:%s')) AS last_purchase_ts
  FROM new_project.small_file
  WHERE event_type = 'purchase'
  GROUP BY user_id
),
max_time AS (
  SELECT MAX(STR_TO_DATE(REPLACE(event_time, ' UTC', ''), '%Y-%m-%d %H:%i:%s')) AS max_ts
  FROM new_project.small_file
)
SELECT
  p.user_id,
  p.frequency,
  p.monetary,
  TIMESTAMPDIFF(HOUR, p.last_purchase_ts, m.max_ts) AS recency_hours,
  (5 - NTILE(4) OVER (ORDER BY TIMESTAMPDIFF(HOUR, p.last_purchase_ts, m.max_ts) ASC)) AS recency_score,
  NTILE(4) OVER (ORDER BY p.frequency DESC) AS frequency_score,
  NTILE(4) OVER (ORDER BY p.monetary DESC) AS monetary_score,
  CONCAT(
    (5 - NTILE(4) OVER (ORDER BY TIMESTAMPDIFF(HOUR, p.last_purchase_ts, m.max_ts) ASC)),
    NTILE(4) OVER (ORDER BY p.frequency DESC),
    NTILE(4) OVER (ORDER BY p.monetary DESC)
  ) AS rfm_code,
  ((5 - NTILE(4) OVER (ORDER BY TIMESTAMPDIFF(HOUR, p.last_purchase_ts, m.max_ts) ASC))
   + NTILE(4) OVER (ORDER BY p.frequency DESC)
   + NTILE(4) OVER (ORDER BY p.monetary DESC)) AS rfm_score
FROM purchases p
CROSS JOIN max_time m
ORDER BY rfm_score DESC, monetary DESC
LIMIT 200;

Distribution (your run):

rfm_score | user_count
10        | 55
9         | 305
8         | 287
7         | 278
6         | 242
5         | 41
4         | 22
3         | 13

Chart file: charts/rfm_distribution.png
Insight: Most users fall into scores 7–9 (active/loyal); 55 VIPs (score 10) — target for loyalty programs. ~76 users have scores ≤5 → win-back campaigns.

 How to create the charts (Excel — step-by-step)

For every analysis above, build a chart in Excel:

1  Paste the SQL result table (headers + rows) into a new Excel sheet.

2 Select the data range. Insert → Column Chart (Clustered) or Bar Chart for brand revenue.

3 Add a clear title, axis labels, and data labels.

4 Save image: Right-click chart → Save as Picture… → Save as charts/<filename>.png (use the filenames above).

5 Upload those PNGs to your repo charts/ folder.

