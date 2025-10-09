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


