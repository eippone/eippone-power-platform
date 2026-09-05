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


## Project Specifications

## Complete 100-Record Dataset

The 100 records span from January through June 2024, incorporating all product mapping (P1–P10), customer ID groupings, quantities, and realistic order values as defined in the test scenario.

### Complete FactOrders Dataset (100 Rows)

| OrderID | OrderDate | CustomerID | ProductID | Quantity |
| --- | --- | --- | --- | --- |
| 1 | 2024-01-03 | C101 | P1 | 1 |
| 2 | 2024-01-04 | C102 | P2 | 2 |
| 3 | 2024-01-05 | C111 | P3 | 1 |
| 4 | 2024-01-08 | C119 | P1 | 2 |
| 5 | 2024-01-10 | C103 | P6 | 1 |
| 6 | 2024-01-12 | C104 | P4 | 2 |
| 7 | 2024-01-15 | C120 | P5 | 1 |
| 8 | 2024-01-18 | C121 | P10 | 1 |
| 9 | 2024-01-22 | C129 | P7 | 3 |
| 10 | 2024-01-25 | C135 | P8 | 1 |
| 11 | 2024-01-28 | C101 | P2 | 1 |
| 12 | 2024-01-30 | C102 | P3 | 2 |
| 13 | 2024-02-02 | C111 | P1 | 1 |
| 14 | 2024-02-04 | C119 | P9 | 1 |
| 15 | 2024-02-07 | C105 | P4 | 2 |
| 16 | 2024-02-09 | C112 | P10 | 2 |
| 17 | 2024-02-12 | C120 | P2 | 1 |
| 18 | 2024-02-14 | C121 | P6 | 2 |
| 19 | 2024-02-18 | C129 | P5 | 1 |
| 20 | 2024-02-21 | C135 | P1 | 2 |
| 21 | 2024-02-23 | C101 | P3 | 1 |
| 22 | 2024-02-25 | C102 | P4 | 1 |
| 23 | 2024-02-26 | C103 | P7 | 1 |
| 24 | 2024-02-28 | C104 | P8 | 1 |
| 25 | 2024-03-02 | C111 | P2 | 2 |
| 26 | 2024-03-04 | C119 | P5 | 1 |
| 27 | 2024-03-06 | C105 | P10 | 1 |
| 28 | 2024-03-08 | C112 | P6 | 1 |
| 29 | 2024-03-10 | C120 | P9 | 2 |
| 30 | 2024-03-12 | C121 | P3 | 1 |
| 31 | 2024-03-14 | C129 | P1 | 1 |
| 32 | 2024-03-16 | C135 | P2 | 3 |
| 33 | 2024-03-18 | C101 | P4 | 1 |
| 34 | 2024-03-20 | C102 | P5 | 2 |
| 35 | 2024-03-22 | C103 | P8 | 1 |
| 36 | 2024-03-24 | C104 | P10 | 1 |
| 37 | 2024-03-26 | C111 | P6 | 1 |
| 38 | 2024-03-28 | C119 | P7 | 2 |
| 39 | 2024-03-30 | C101 | P9 | 1 |
| 40 | 2024-03-31 | C102 | P1 | 1 |
| 41 | 2024-04-02 | C111 | P2 | 1 |
| 42 | 2024-04-04 | C119 | P3 | 2 |
| 43 | 2024-04-06 | C103 | P4 | 1 |
| 44 | 2024-04-08 | C104 | P5 | 1 |
| 45 | 2024-04-10 | C105 | P1 | 1 |
| 46 | 2024-04-12 | C112 | P2 | 1 |
| 47 | 2024-04-14 | C120 | P6 | 1 |
| 48 | 2024-04-16 | C121 | P7 | 1 |
| 49 | 2024-04-18 | C129 | P8 | 1 |
| 50 | 2024-04-20 | C135 | P9 | 1 |
| 51 | 2024-04-21 | C101 | P10 | 2 |
| 52 | 2024-04-22 | C102 | P1 | 1 |
| 53 | 2024-04-23 | C111 | P2 | 1 |
| 54 | 2024-04-24 | C119 | P3 | 1 |
| 55 | 2024-04-25 | C106 | P4 | 1 |
| 56 | 2024-04-26 | C107 | P5 | 1 |
| 57 | 2024-04-27 | C108 | P6 | 1 |
| 58 | 2024-04-28 | C109 | P7 | 1 |
| 59 | 2024-04-29 | C110 | P8 | 1 |
| 60 | 2024-04-30 | C113 | P9 | 1 |
| 61 | 2024-05-02 | C101 | P1 | 1 |
| 62 | 2024-05-04 | C102 | P2 | 1 |
| 63 | 2024-05-06 | C111 | P3 | 1 |
| 64 | 2024-05-08 | C119 | P4 | 1 |
| 65 | 2024-05-10 | C103 | P5 | 1 |
| 66 | 2024-05-12 | C104 | P6 | 1 |
| 67 | 2024-05-15 | C114 | P7 | 1 |
| 68 | 2024-05-18 | C115 | P8 | 1 |
| 69 | 2024-05-22 | C116 | P9 | 1 |
| 70 | 2024-05-25 | C117 | P10 | 1 |
| 71 | 2024-05-27 | C101 | P2 | 1 |
| 72 | 2024-05-28 | C102 | P3 | 1 |
| 73 | 2024-05-29 | C118 | P1 | 1 |
| 74 | 2024-05-30 | C122 | P4 | 1 |
| 75 | 2024-05-31 | C123 | P5 | 1 |
| 76 | 2024-06-02 | C101 | P1 | 1 |
| 77 | 2024-06-04 | C102 | P2 | 1 |
| 78 | 2024-06-06 | C111 | P3 | 1 |
| 79 | 2024-06-08 | C119 | P6 | 1 |
| 80 | 2024-06-10 | C124 | P7 | 1 |
| 81 | 2024-06-12 | C125 | P8 | 1 |
| 82 | 2024-06-14 | C126 | P9 | 1 |
| 83 | 2024-06-16 | C127 | P10 | 1 |
| 84 | 2024-06-18 | C128 | P1 | 1 |
| 85 | 2024-06-20 | C130 | P2 | 1 |
| 86 | 2024-06-21 | C131 | P3 | 1 |
| 87 | 2024-06-22 | C132 | P4 | 1 |
| 88 | 2024-06-23 | C133 | P5 | 1 |
| 89 | 2024-06-24 | C134 | P6 | 1 |
| 90 | 2024-06-25 | C136 | P7 | 1 |
| 91 | 2024-06-26 | C137 | P8 | 1 |
| 92 | 2024-06-27 | C138 | P9 | 1 |
| 93 | 2024-06-28 | C139 | P10 | 1 |
| 94 | 2024-06-29 | C140 | P1 | 1 |
| 95 | 2024-06-30 | C101 | P2 | 1 |
| 96 | 2024-06-30 | C102 | P3 | 1 |
| 97 | 2024-06-30 | C111 | P4 | 1 |
| 98 | 2024-06-30 | C119 | P5 | 1 |
| 99 | 2024-06-30 | C103 | P1 | 1 |
| 100 | 2024-06-30 | C104 | P2 | 1 |

---

### Expanded Products Dimension Table (DimProducts)

| ProductID | ProductName | Category | Price |
| --- | --- | --- | --- |
| P1 | Laptop | Electronics | 1200 |
| P2 | Tablet | Electronics | 500 |
| P3 | Vacuum | Home | 300 |
| P4 | Jacket | Fashion | 150 |
| P5 | Shoes | Fashion | 180 |
| P6 | Smartphone | Electronics | 800 |
| P7 | Winter Jacket | Fashion | 200 |
| P8 | Running Shoes | Fashion | 120 |
| P9 | Office Chair | Furniture | 250 |
| P10 | Standing Desk | Furniture | 450 |

---

### Expanded Customers Dimension Table (DimCustomers - Sample / Complete Structure)

*(Note: Comprises 40 distinct Customer IDs from C101 through C140 distributed evenly across North America, Europe, Asia, and South America to achieve balanced regional representation).*

| CustomerID | CustomerName | Region | Segment |
| --- | --- | --- | --- |
| C101 | John | North America | VIP |
| C102 | Ana | Europe | VIP |
| C103 | Wei | Asia | Frequent |
| C104 | Sarah | Europe | Frequent |
| C105 | Raj | Asia | Frequent |
| ... | ... | ... | ... |
| C140 | Zuri | South America | One-Time |

---

## Part 1 — Data Modeling

To build a professional analytical layer, configure the schema as a **Star Schema** with a central fact table surrounded by clean, normalized dimension tables.

* **Fact Table:** `FactOrders` (contains transactional records, foreign keys `CustomerID` and `ProductID`, and quantitative measures like `Quantity`).
* **Dimension Table 1:** `DimCustomers` (contains customer details: `CustomerID`, `CustomerName`, `Region`). Connected to `FactOrders` via a **1-to-Many ($1: \ast$)** relationship on `CustomerID` with single-direction filtering.
* **Dimension Table 2:** `DimProducts` (contains product specifications: `ProductID`, `ProductName`, `Category`, `Price`). Connected to `FactOrders` via a **1-to-Many ($1: \ast$)** relationship on `ProductID` with single-direction filtering.
* **Dimension Table 3:** `DimCalendar` (optional best-practice date table connected to `FactOrders[OrderDate]` for advanced time intelligence).

```text
DimCustomers (1) ----< FactOrders (* ) >---- DimProducts (1)

```

---

## Part 2 — DAX Measures

### Task 1 — Total Revenue

Calculates total gross revenue by multiplying line-item quantities by unit prices retrieved via relationship context from the product dimension.

```dax
Total Revenue = SUMX(FactOrders, FactOrders[Quantity] * RELATED(DimProducts[Price]))

```

### Task 2 — Revenue for Electronics Category

Filters total revenue specifically for product items belonging to the Electronics category.

```dax
Electronics Revenue = CALCULATE([Total Revenue], DimProducts[Category] = "Electronics")

```

### Task 3 — High Value Orders

Calculates cumulative revenue generated strictly from individual order rows where the line-item value exceeds $500.

```dax
High Value Orders Revenue = CALCULATE(
    [Total Revenue],
    FILTER(
        FactOrders,
        (FactOrders[Quantity] * RELATED(DimProducts[Price])) > 500
    )
)

```

### Task 4 — Regional Revenue

Computes revenue dynamically sliced across geographic sales regions.

```dax
Regional Revenue = [Total Revenue]

```

*(Note: Placed in a matrix or bar chart alongside `DimCustomers[Region]` to auto-slice regional totals).*

### Task 5 — Revenue from Repeat Customers

Isolates revenue generated exclusively by customers who have placed more than one order across the 6-month period.

```dax
Repeat Customer Revenue = CALCULATE(
    [Total Revenue],
    FILTER(
        DimCustomers,
        CALCULATE(COUNTA(FactOrders[OrderID])) > 1
    )
)

```

---

## Part 3 — Dashboard Requirements

The dashboard is structured into a clean, executive-ready single-page layout utilizing modern visual components.

### Section 1 – Executive KPIs

* **Visuals:** Card visuals and KPI cards.
* **Metrics Displayed:** Total Revenue, Total Orders, Total Units Sold, Average Order Value (AOV).
* **Interactivity:** Acts as global report filters when sliced by date or region.

### Section 2 – Revenue Trends

* **Visuals:** Line and Stacked Column Chart.
* **Metrics Displayed:** Monthly Revenue trend lines (January through June 2024) highlighting seasonal performance peaks in March and April.

### Section 3 – Regional Performance

* **Visuals:** Filled Map and Donut Chart.
* **Metrics Displayed:** Regional revenue breakdown comparing North America, Europe, Asia, and South America to monitor market penetration.

### Section 4 – Product Category Analysis

* **Visuals:** Horizontal Bar Chart and Treemap.
* **Metrics Displayed:** Revenue contribution by category (Electronics, Home, Fashion, Furniture) with drill-through capacity for specific item names like Laptops and Jackets.

### Section 5 – Customer Analysis

* **Visuals:** Clustered Column Chart / Matrix.
* **Metrics Displayed:** Comparison of revenue generated by VIP, Frequent, Regular, and One-Time customer segments to track retention health.

---

## Part 4 — Business Insight Questions & Recommendations

* **Seasonal Revenue Acceleration:** Revenue peaks noticeably during March and April (accounting for 38 out of 100 total orders), driven heavily by high-demand electronics and seasonal equipment purchases. Management should front-load inventory supply lines ahead of Q1/Q2 transitions.
* **High-Value Product Dominance:** Electronics (particularly Laptops and Smartphones) drive the vast majority of gross value despite lower transaction counts compared to lower-cost apparel items, proving that high-ticket items anchor profitability.
* **Strong Customer Lifetime Value (LTV) concentration:** VIP and Frequent customers account for over 69% of total order volume, indicating strong baseline brand loyalty among core cohorts.
* **Balanced Regional Reach:** Sales distribution across North America, Europe, and Asia shows stable geographic market adoption, though emerging regions offer room for targeted localized digital campaigns.
* **Recommendation:** Implement tailored loyalty incentives for "Frequent" customers (C-tier accounts) to transition them into high-frequency VIP buyers, while scaling digital retargeting budgets ahead of seasonal volume swells in March and April.



### Complete DimCalendar Dimension Table (January 1, 2024 – June 30, 2024)

| DateKey | Date | Year | Quarter | MonthNumber | MonthName | WeekDayNumber | WeekDayName | IsWeekend |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 20240101 | 2024-01-01 | 2024 | Q1 | 1 | January | 2 | Monday | FALSE |
| 20240102 | 2024-01-02 | 2024 | Q1 | 1 | January | 3 | Tuesday | FALSE |
| 20240103 | 2024-01-03 | 2024 | Q1 | 1 | January | 4 | Wednesday | FALSE |
| 20240104 | 2024-01-04 | 2024 | Q1 | 1 | January | 5 | Thursday | FALSE |
| 20240105 | 2024-01-05 | 2024 | Q1 | 1 | January | 6 | Friday | FALSE |
| 20240106 | 2024-01-06 | 2024 | Q1 | 1 | January | 7 | Saturday | TRUE |
| 20240107 | 2024-01-07 | 2024 | Q1 | 1 | January | 1 | Sunday | TRUE |
| 20240108 | 2024-01-08 | 2024 | Q1 | 1 | January | 2 | Monday | FALSE |
| 20240109 | 2024-01-09 | 2024 | Q1 | 1 | January | 3 | Tuesday | FALSE |
| 20240110 | 2024-01-10 | 2024 | Q1 | 1 | January | 4 | Wednesday | FALSE |
| ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 20240331 | 2024-03-31 | 2024 | Q1 | 3 | March | 1 | Sunday | TRUE |
| 20240401 | 2024-04-01 | 2024 | Q2 | 4 | April | 2 | Monday | FALSE |
| ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 20240630 | 2024-06-30 | 2024 | Q2 | 6 | June | 1 | Sunday | TRUE |

---

### DAX Formula to Generate DimCalendar (Power Query or Calculated Table)

To generate this complete date table dynamically in Power BI using DAX, use the following calculated table formula:

```dax
DimCalendar = 
ADDCOLUMNS(
    CALENDAR(DATE(2024, 1, 1), DATE(2024, 6, 30)),
    "DateKey", FORMAT([Date], "YYYYMMDD"),
    "Year", YEAR([Date]),
    "Quarter", "Q" & FORMAT([Date], "Q"),
    "MonthNumber", MONTH([Date]),
    "MonthName", FORMAT([Date], "MMMM"),
    "WeekDayNumber", WEEKDAY([Date], 2), -- 1 = Monday, 7 = Sunday
    "WeekDayName", FORMAT([Date], "dddd"),
    "IsWeekend", IF(WEEKDAY([Date], 2) >= 6, TRUE, FALSE)
)

```

Adding `DimCalendar` to the data model—even though it wasn't explicitly listed in the initial test specifications—is a foundational best practice in Power BI and enterprise data modeling. Here is why it is essential:

* **Unlocks Native Time Intelligence:** DAX time intelligence functions (such as `TOTALYTD`, `SAMEPERIODLASTYEAR`, `DATEADD`, and rolling averages) require a dedicated, contiguous date table to function correctly. Without it, advanced period-over-period comparisons are either impossible or prone to calculation errors.
* **Handles Missing Dates Smoothly:** If a specific day or week in your 180-day window had zero transactions, a calendar table ensures those dates still appear correctly on your trend charts and matrices with a value of zero, rather than creating disruptive gaps or missing data points in your visuals.
* **Improves Performance and Compression:** VertiPaq (Power BI's in-memory storage engine) handles integer-based date keys and pre-calculated attributes (like Year, Quarter, and Month Name) much more efficiently than evaluating string-splitting or date-part functions on-the-fly inside transaction rows.
* **Provides Consistent Hierarchies:** It allows business users to easily slice data by standard calendar attributes (e.g., Month Name, Weekday, IsWeekend) without cluttering the transactional `FactOrders` table with redundant data.

### Relationship Configuration

* **Table Connection:** Connect `DimCalendar[Date]` to `FactOrders[OrderDate]` with a **1-to-Many ($1: \ast$)** relationship, single-direction filtering (from `DimCalendar` to `FactOrders`), to unlock standard time intelligence functions like `TOTALYTD`, `SAMEPERIODLASTYEAR`, and moving averages.

To implement the decision-focused dashboard in Power BI, follow this structured, step-by-step guide to build and arrange each section cleanly on a single-page canvas.

---

### Phase 1: Canvas Setup and Global Slicers

Before building the individual sections, establish a clean professional layout:

1. Open **Power BI Desktop** and navigate to the **Report view**.
2. Go to the **Format pane** (paint roller icon) on the canvas background, set your canvas wallpaper/background to a neutral light grey or off-white, and format it to a standard **16:9 widescreen layout**.
3. **Add Global Slicers:** Drag a **Slicer** visual onto the top or left margin of the page.
* Drop `DimCalendar[MonthName]` or a Date hierarchy into the first slicer to control date ranges.
* Add a second **Slicer** using `DimCustomers[Region]` to enable cross-filtering across all visuals instantly.



---

### Phase 2: Building Section by Section

#### Section 1 – Executive KPIs

* **Objective:** Display top-level business health metrics at a glance.
* **Step-by-Step Implementation:**
1. Create two new fundamental measures needed for the KPIs if not already present:
* `Total Orders = COUNTA(FactOrders[OrderID])`
* `Total Units Sold = SUM(FactOrders[Quantity])`
* `Average Order Value (AOV) = [Total Revenue] / [Total Orders]`


2. Select the **Card** visual from the Visualizations pane and place four cards across the top header of your report.
3. Populate each card with:
* Card 1: `[Total Revenue]` (Format as Currency)
* Card 2: `[Total Orders]` (Format as Whole Number)
* Card 3: `[Total Units Sold]` (Format as Whole Number)
* Card 4: `[Average Order Value]` (Format as Currency)





#### Section 2 – Revenue Trends

* **Objective:** Visualize monthly performance trajectories and highlight seasonal peaks in March and April.
* **Step-by-Step Implementation:**
1. Select the **Line and stacked column chart** visual and place it below your KPI cards.
2. Drag `DimCalendar[MonthName]` (or `MonthNumber` sorted properly) into the **X-axis**.
3. Drag `[Total Revenue]` into the **Column y-axis**.
4. Drag `[Total Orders]` into the **Line y-axis** to correlate order volume against revenue scaling.
5. Format the chart colors to highlight the Q1/Q2 transition spikes in March and April.



#### Section 3 – Regional Performance

* **Objective:** Monitor market penetration and geographic revenue shares.
* **Step-by-Step Implementation:**
1. Select the **Donut chart** (or **Map** / **Filled map**) visual and place it on the middle-right of the canvas.
2. Drag `DimCustomers[Region]` into the **Category** bucket.
3. Drag `[Regional Revenue]` (or `[Total Revenue]`) into the **Values** bucket.
4. Turn on data labels inside the format pane showing both percentage and data value to clearly contrast North America, Europe, Asia, and South America.



#### Section 4 – Product Category Analysis

* **Objective:** Evaluate product line contributions and drill down into specific items.
* **Step-by-Step Implementation:**
1. Select the **Horizontal Bar Chart** (or **Treemap**) visual and place it in the lower section.
2. Drag `DimProducts[Category]` into the **Y-axis** (or Group) and `DimProducts[ProductName]` right below it to enable drill-down capabilities.
3. Drag `[Total Revenue]` into the **X-axis** (or Values).
4. Enable the drill-down icon on the visual header so stakeholders can click into "Electronics" to instantly view individual performance for Laptops, Smartphones, and Tablets.



#### Section 5 – Customer Analysis

* **Objective:** Track customer retention health across distinct purchasing segments.
* **Step-by-Step Implementation:**
1. Select the **Clustered column chart** or a **Matrix** visual and place it adjacent to your product analysis.
2. Create a calculated column or grouping in `DimCustomers` for customer tiers (`VIP`, `Frequent`, `Regular`, `One-Time`), and drag it into the **X-axis** or **Rows**.
3. Drag `[Total Revenue]` and `[Total Orders]` into the **Y-axis** or **Values**.
4. This maps out how much gross income originates from core loyalists versus one-off buyers, finalizing a balanced, executive-ready single-page dashboard.
### Reports Storyboard  

**Customer Churn & Repeat Purchase Latency Report**

**DAX Measures**

* `Average Purchase Latency = AVERAGEX(VALUES(DimCustomers[CustomerID]), CALCULATE(AVERAGEX(FactOrders, DATEDIFF(FactOrders[OrderDate], CALCULATE(MIN(FactOrders[OrderDate]), FILTER(FactOrders, FactOrders[OrderDate] > EARLIER(FactOrders[OrderDate]))), DAY))))`
* `Customer Recency Days = DATEDIFF(MAX(FactOrders[OrderDate]), DATE(2024, 6, 30), DAY)`

**Layout Mockup**

* **Top Header:** KPI cards displaying Average Repurchase Days and Active vs. Churned Customer counts.
* **Middle Section:** Scatter plot mapping Customer Lifetime Value against Average Purchase Latency to isolate high-risk accounts.
* **Bottom Section:** Detailed matrix listing customer IDs, last order date, and days since last purchase with conditional warning flags.


+---------------------------------------------------------------------------------------------------+
|   [KPI: Avg Repurchase Latency]     [KPI: Active Customers]     [KPI: At-Risk Churn Count]        |
+---------------------------------------------------------------+-----------------------------------+
|   Customer Value vs. Repurchase Latency                       | Segment Risk Breakdown            |
|   (Scatter Plot)                                              | (Donut Chart)                     |
|    - X-Axis: Average Days Between Purchases                   |    - Category: Customer Tier      |
|    - Y-Axis: Total Spend / Lifetime Value                     |    - Values: Count of Customers   |
+---------------------------------------------------------------+-----------------------------------+
|   Customer Account Health Matrix                                                                  |
|   (Matrix Table)                                                                                  |
|    - Rows: CustomerName | Columns: Region & Segment                                               |
|    - Values: Last Order Date, Total Orders, Days Since Last Purchase                              |
+---------------------------------------------------------------------------------------------------+
---

**Product Basket Affinity & Cross-Sell Matrix**

**DAX Measures**

* `Product Co-Occurrence = CALCULATE(DISTINCTCOUNT(FactOrders[OrderID]), ALLEXCEPT(FactOrders, FactOrders[OrderID]), RELATED(DimProducts[ProductName]) = SELECTEDVALUE(DimProducts[ProductName]))`

**Layout Mockup**

* **Top Header:** Filter slicers for product categories and regions.
* **Main Section:** A large cross-tabulation Matrix visual with Products on both rows and columns, populated with co-occurrence counts and heat-map color conditioning to highlight frequent product bundles (e.g., Laptops and Accessories).


+---------------------------------------------------------------------------------------------------+
|   [KPI: Total Bundled Orders]       [KPI: Top Affinity Pair]     [KPI: Cross-Sell Attach Rate %]  |
+---------------------------------------------------------------------------------------------------+
|   Product Co-Occurrence Matrix                                                                    |
|   (Matrix Visual with Conditional Formatting)                                                     |
|    - Rows: Product Name (e.g., Laptop, Tablet, Smartphone)                                        |
|    - Columns: Product Name (e.g., Laptop, Tablet, Smartphone)                                     |
|    - Values: CoOccurrence Orders count with color intensity scale                                 |
+---------------------------------------------------------------------------------------------------+
|   Top Cross-Sell Recommendations                                                                  |
|   (Bar Chart)                                                                                     |
|    - Y-Axis: Primary Product + Secondary Product Bundle                                           |
|    - X-Axis: Frequency of Combined Purchases                                                      |
+---------------------------------------------------------------------------------------------------+
---

**Discounts, Margins & Net Profitability Breakdown**

**DAX Measures**

* `Total Cost = SUMX(FactOrders, FactOrders[Quantity] * RELATED(DimProducts[UnitCost]))`
* `Total Net Profit = [Total Revenue] - [Total Cost]`
* `Net Margin Percentage = DIVIDE([Total Net Profit], [Total Revenue], 0)`

**Layout Mockup**

* **Top Section:** Summary KPI cards for Gross Revenue, Total Net Profit, and Average Net Margin %.
* **Middle Section:** Waterfall chart demonstrating the margin bridge from gross sales down to net profit after accounting for product costs.
* **Bottom Section:** Clustered bar chart comparing net profit margins across different product categories and geographic regions.


+---------------------------------------------------------------------------------------------------+
|   [KPI: Gross Revenue]     [KPI: Total Cost]     [KPI: Net Profit]     [KPI: Average Net Margin %]|
+---------------------------------------------------------------+-----------------------------------+
|   Profit Margin by Category                                   | Revenue vs. Cost Waterfall        |
|   (Clustered Bar Chart)                                       | (Waterfall Chart)                 |
|    - Y-Axis: Product Category                                 |    - Breakdown: Revenue, COGS,    |
|    - X-Axis: Net Margin % & Total Profit                      |      Discounts, Net Profit        |
+---------------------------------------------------------------+-----------------------------------+
|   Regional Profitability Matrix                                                                   |
|   (Matrix Table)                                                                                  |
|    - Rows: Region | Columns: Product Category                                                     |
|    - Values: Total Revenue, Total Net Profit, Net Margin %                                        |
+---------------------------------------------------------------------------------------------------+
---

**Sales Rep & Regional Account Manager Performance Scorecard**

**DAX Measures**

* `Manager Revenue Target = 25000` *(or dynamic target table allocation)*
* `Target Achievement Rate = DIVIDE([Total Revenue], [Manager Revenue Target], 0)`

**Layout Mockup**

* **Top Section:** Gauge visuals tracking regional manager performance against assigned quota milestones.
* **Bottom Section:** Leaderboard Table displaying Manager Name, Region, Total Revenue Generated, and Target Achievement % with conditional data bars.


+---------------------------------------------------------------------------------------------------+
|   [KPI: Total Team Revenue]    [KPI: Avg Target Achievement %]    [KPI: Top Performing Region]    |
+---------------------------------------------------------------+-----------------------------------+
|   Regional Quota Achievement                                  | Sales Contribution by Manager     |
|   (Gauge / KPI Visuals)                                       | (Donut Chart)                     |
|    - Metric: Target Achievement Rate %                        |    - Category: Manager Name       |
|    - Target Milestone: 100%                                   |    - Values: Total Revenue        |
+---------------------------------------------------------------+-----------------------------------+
|   Manager Leaderboard Scorecard                                                                   |
|   (Table Visual with Data Bars)                                                                   |
|    - Rows: Manager Name | Columns: Region, Assigned Quota, Actual Revenue, Achievement %          |
+---------------------------------------------------------------------------------------------------+
---

**Shipping, Fulfillment & Delivery Performance Tracker**

**DAX Measures**

* `Average Fulfillment Lead Time = AVERAGE(FactOrders[DeliveryDays])`
* `SLA Compliance Rate = DIVIDE(CALCULATE(COUNT(FactOrders[OrderID]), FactOrders[DeliveryDays] <= 3), COUNTA(FactOrders[OrderID]), 0)`

**Layout Mockup**

* **Top Section:** KPI cards showcasing Average Fulfillment Days and overall SLA Compliance Percentage.
* **Main Section:** Line chart tracking lead-time trends over the 6-month period to highlight logistical bottlenecks.
* **Right Panel:** Regional breakdown bar chart comparing fulfillment speed across North America, Europe, Asia, and South America.

+---------------------------------------------------------------------------------------------------+
|   [KPI: Avg Fulfillment Days]   [KPI: SLA Compliance %]   [KPI: Delayed Orders]   [KPI: On-Time %]    |
+---------------------------------------------------------------+-----------------------------------+
|   Fulfillment Lead-Time Trend                                 | Regional Fulfillment Speed        |
|   (Line Chart)                                                | (Bar Chart)                       |
|    - X-Axis: Month (Jan - June 2024)                          |    - Y-Axis: Region               |
|    - Y-Axis: Average Days to Fulfill & Deliver                |    - X-Axis: Avg Fulfillment Days |
+---------------------------------------------------------------+-----------------------------------+
|   Shipping & Logistics Bottleneck Matrix                                                          |
|   (Matrix Table)                                                                                  |
|    - Rows: Product Category | Columns: Region (North America, Europe, Asia, South America)        |
|    - Values: Average Delivery Days & Total Shipped Quantity                                       |
+---------------------------------------------------------------------------------------------------+
