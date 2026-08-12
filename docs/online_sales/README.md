# EIPPONE Simulation Dynamics: International Online Sales Performance & Executive Intelligence Platform

### Project Overview

The International Online Sales Performance dashboard is an executive-level analytical solution designed to evaluate six months of global   
retail operations from January through June 2024. Spanning 100 structured orders across 40 distinct customers and 10 product lines,   
the project establishes a robust star schema data model and advanced DAX measures to track profitability, customer segmentation,   
and geographic distribution. By integrating an executive scorecard with interactive trend lines, regional maps, and category breakdowns,   
the dashboard empowers leadership to monitor key performance indicators and actionable insights at a glance.

### Business Problems & Solutions

* **Business Problem:** Management lacked visibility into temporal revenue shifts and seasonal demand spikes
* across the first half of the year, risking inventory stockouts and misallocated marketing spend during peak buying windows.
* **Solution:** Implemented Section 2 and Section 3 of the dashboard featuring monthly trend lines and regional distributions,
* which isolate Q2 order acceleration (60 orders vs. 40 in Q1) and pinpoint North America as the leading revenue driver ($22,930),
* enabling proactive inventory scaling and targeted regional campaigns.


* **Business Problem:** The organization needed clarity on which product lines and customer loyalty tiers actually drove gross profitability
  versus low-value transactional volume.
* **Solution:** Developed rigorous DAX-driven categorizations in Sections 4 and 5 to separate high-ticket electronics (anchored by Laptops generating
* over 53% of category revenue) from routine apparel purchases, while mapping lifetime value disparities that show VIP buyers generating $23.12K
* and accounting for 38% of total orders to guide precise up-sell and cross-sell retention strategies.


### Data Origination & Generation Specification

* **Business Framework:** Built upon a controlled, business-realistic international online retail simulation spanning January through June 2024 (approx. 180 days) to avoid arbitrary random noise.
* **Volume Constraints:** Engineered around a deterministic total of 100 orders distributed chronologically across six months, featuring seasonal spikes peaking in Q2 (March–June).
* **Customer Behavioral Distribution:** Comprises 40 distinct customer profiles (C101–C140) categorized into behavioral purchasing tiers:
* **VIP Customers:** 4 accounts driving 35 total orders (8–10 purchases each).
* **Frequent Customers:** 8 accounts driving 34 total orders (4–6 purchases each).
* **Regular Customers:** 15 accounts driving 30 total orders (2 purchases each).
* **One-Time Customer:** 1 account (C140) driving 1 order.


* **Regional Representation:** Balanced geographic distribution mapping customers evenly across North America, Europe, Asia, and South America.
* **Product Assortment & Pricing:** Standardized across 10 distinct product categories (P1–P10) ranging from high-value premium assets (e.g., Laptops at $1,200) to lower-cost transaction items (e.g., Jackets at $150).
* **Quantity & Transaction Spread:** Order quantities follow a realistic frequency model where single-unit purchases account for 52% of rows, scaling up to multi-unit bundle orders that drive high-value revenue thresholds exceeding $500.
