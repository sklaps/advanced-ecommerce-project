# EcomPulse — Advanced eCommerce Analytics (SK)

**Short pitch:** Advanced analysis of product and customer behaviour from event logs (views, carts, purchases). Reproducible SQL + Excel charts + RFM segmentation for business action.

---

## Quick facts (dataset)
- **Source table:** `new_project.small_file`  
- **Total events:** **93,912**  
- **Distinct users:** **19,226**  
- **Distinct products:** **19,908**  
- **Event types:** `view` (91,268), `purchase` (1,535), `cart` (1,109)  
- **Time window in sample:** `2019-10-01 00:00:00` → `2019-10-01 04:22:05` (hourly peak at **03:00** UTC)

---

## Visuals & Key findings

### Top 10 Purchased Products
**Query:** top products by purchase count.  
![Top Products](charts/top_products.png)

**Insights**
- Samsung holds 6 of the top 10 items — strong volume leader.  
- Apple items appear frequently among top revenue-producing products.  
- Business action: prioritize Samsung inventory and Apple premium offers.

---

### Average Price by Brand
**Query:** average purchase price per brand.  
![Avg Price by Brand](charts/avg_price_by_brand.png)

**Insights**
- Mercury and Pulser are premium-priced; Apple sits in mid-high range.  
- Use price segmentation to plan premium bundles vs. mass campaigns.

---

### Revenue by Brand
**Query:** brand-level total revenue (sum of purchase prices).  
![Revenue by Brand](charts/revenue_by_brand.png)

**Insights**
- Apple leads revenue despite fewer units → high average order value (AOV).  
- Samsung leads unit sales → volume strategy.  
- Consider distinct promotions for high-AOV vs. high-volume brands.

---

### Hourly Activity (Operations Insight)
**Query:** events per hour.  
![Hourly Activity](charts/hourly_activity.png)

**Insights**
- Peak activity 02:00–04:00 UTC (peak at 03:00). Could be timezone-driven — schedule campaigns and scale infra accordingly.

---

### RFM Customer Segmentation
**Query:** Recency / Frequency / Monetary segmentation (RFM).  
![RFM Distribution](charts/rfm_distribution.png)

**Insights**
- Majority of customers have RFM scores between 7–9 (active/loyal).  
- 55 VIPs (score 10) — ideal for loyalty outreach.  
- ~76 low-score users → win-back campaigns recommended.

---

## SQL files
- `queries.sql` — all analysis queries used in this project (funnel, top products, revenue, RFM, etc.)

---

## How to reproduce locally
1. Load data into `new_project.small_file`.  
2. Run `sql/00_data_cleaning.sql` (brand blanks → 'Unknown', price zeros → NULL).  
3. Run `sql/01_analysis_queries.sql` to produce CSVs for charts.  
4. Paste CSV results into Excel, create charts, export PNG images into `charts/`.

---

## Actionable recommendations
1. **Homepage & inventory:** emphasize Samsung for conversions; highlight Apple for margin.  
2. **Campaign timing:** run flash sales around 02:00–04:00 UTC (test with small A/B groups first).  
3. **Customer programs:** invite VIPs (RFM ≥10) to a loyalty program; send 10% win-back coupons to low-score users.

---

## Files in this repository
- `README.md` — this file  
- `queries.sql` — SQL queries used  
- `charts/` — chart PNG files (top_products.png, avg_price_by_brand.png, revenue_by_brand.png, hourly_activity.png, rfm_distribution.png)  
- `docs/EcomPulse_Report_SK.pdf` — polished report (coming soon)

---

**Author:** SK — Data Analyst  

