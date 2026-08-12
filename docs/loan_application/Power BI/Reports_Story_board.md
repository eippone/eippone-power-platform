Here are the complete **DAX measure formulas** and **layout mockups** for all **5 suggested report pages** to complete the  Loan Analytics suite.

---

# Page 1: Credit Risk & Applicant Profile

### DAX Measures

```dax
// 1. Average Requested Loan Amount
Avg Requested Amount = 
AVERAGE(cr_application[Amount])

// 2. Average Stated Income
Avg Stated Income = 
AVERAGE(cr_application[StatedIncome])

// 3. Debt-to-Income (DTI) Ratio %
Avg Debt to Income Ratio = 
AVERAGEX(
    FILTER(cr_application, NOT(ISBLANK(cr_application[StatedIncome])) && cr_application[StatedIncome] > 0),
    DIVIDE(cr_application[MonthlyDebt], cr_application[StatedIncome] / 12, 0)
)

// 4. Income Tier Category (Calculated Column)
Income Tier = 
SWITCH(
    TRUE(),
    cr_application[StatedIncome] < 50000, "< $50k",
    cr_application[StatedIncome] <= 100000, "$50k - $100k",
    cr_application[StatedIncome] <= 150000, "$100k - $150k",
    "> $150k"
)

```

### Layout Mockup

```text
+---------------------------------------------------------------------------------------------------+
|  [KPI: Avg Requested Amount]  [KPI: Avg Stated Income]  [KPI: Avg DTI Ratio]  [KPI: Avg Credit Score] |
+---------------------------------------------------------------+-----------------------------------+
|  Loan Volume & Approval Rate by Income Tier                   | Demographic Breakdown             |
|  (Clustered Column & Line Chart)                              | (Donut Chart)                     |
|   - X-Axis: Income Tier (<$50k, $50k-$100k, etc.)             |   - Slice: Employment Status      |
|   - Columns: Application Count                                |     (Employed, Self-Employed, etc.)|
|   - Line: Approval Rate %                                     |   - Metric: Volume & Approval %   |
+---------------------------------------------------------------+-----------------------------------+
|  Applicant Risk Matrix                                                                            |
|  (Scatter Plot: X = Stated Income, Y = Requested Amount, Size = Approval Rate %, Legend = Status) |
+---------------------------------------------------------------------------------------------------+

```

---

# Page 2: Operational Bottlenecks & Review Velocity

### DAX Measures

```dax
// 1. Average Time to Decision (Days)
Avg Decision Time (Days) = 
AVERAGEX(
    FILTER(cr_application, NOT(ISBLANK(cr_application[DecisionDate]))),
    DATEDIFF(cr_application[SubmissionDate], cr_application[DecisionDate], DAY)
)

// 2. Under Review Backlog Count
Under Review Backlog = 
CALCULATE(
    COUNT(cr_application[ApplicationID]),
    cr_application[Status] = "Under Review"
)

// 3. SLA Compliance % (Threshold: <= 5 Days)
SLA Compliance % = 
DIVIDE(
    CALCULATE(
        COUNT(cr_application[ApplicationID]),
        FILTER(
            cr_application,
            NOT(ISBLANK(cr_application[DecisionDate])) &&
            DATEDIFF(cr_application[SubmissionDate], cr_application[DecisionDate], DAY) <= 5
        )
    ),
    CALCULATE(
        COUNT(cr_application[ApplicationID]),
        NOT(ISBLANK(cr_application[DecisionDate]))
    ),
    0
)

// 4. Application Age Bucket (Calculated Column)
Application Age Group = 
VAR DaysPending = DATEDIFF(cr_application[SubmissionDate], TODAY(), DAY)
RETURN
SWITCH(
    TRUE(),
    ISBLANK(cr_application[DecisionDate]) && DaysPending <= 3, "0 - 3 Days",
    ISBLANK(cr_application[DecisionDate]) && DaysPending <= 7, "4 - 7 Days",
    ISBLANK(cr_application[DecisionDate]) && DaysPending <= 14, "8 - 14 Days",
    ISBLANK(cr_application[DecisionDate]), "15+ Days (Overdue)",
    "Decisioned"
)

```

### Layout Mockup

```text
+---------------------------------------------------------------------------------------------------+
|  [KPI: Avg Decision Time (Days)]  [KPI: SLA Compliance %]  [KPI: Under Review Backlog] [KPI: Overdue] |
+---------------------------------------------------------------+-----------------------------------+
|  Stage Duration Funnel                                        | Underwriter Workload              |
|  (Funnel Visual)                                              | (Stacked Bar Chart)               |
|   - Submitted -> Document Verified -> Under Review -> Decision|   - Y-Axis: Underwriter Name      |
|   - Value: Average Days Spent per Stage                       |   - Bars: Under Review vs Decisioned  |
+---------------------------------------------------------------+-----------------------------------+
|  Aging Backlog Matrix                                                                             |
|  (Matrix Table)                                                                                   |
|   - Rows: Loan Type | Columns: Application Age Group (0-3 days, 4-7 days, 8-14 days, 15+ days)      |
|   - Values: Count of Applications & Sum of Loan Amount                                            |
+---------------------------------------------------------------------------------------------------+

```

---

# Page 3: High-Value Loan & Concentration Portfolio

### DAX Measures

```dax
// Dynamic Dynamic High-Value Threshold (Uses Configuration Table or Parameter)
High Value Threshold = 150000 

// 1. High-Value Application Count
High Value App Count = 
CALCULATE(
    COUNT(cr_application[ApplicationID]),
    cr_application[Amount] >= [High Value Threshold]
)

// 2. Total High-Value Exposure ($)
High Value Total Exposure = 
CALCULATE(
    SUM(cr_application[Amount]),
    cr_application[Amount] >= [High Value Threshold]
)

// 3. Pending High-Value Exposure ($)
High Value Pending Exposure = 
CALCULATE(
    SUM(cr_application[Amount]),
    cr_application[Amount] >= [High Value Threshold],
    cr_application[Status] = "Under Review"
)

// 4. High-Value Approval Rate %
High Value Approval Rate % = 
DIVIDE(
    CALCULATE(COUNT(cr_application[ApplicationID]), cr_application[Amount] >= [High Value Threshold], cr_application[Status] = "Approved"),
    CALCULATE(COUNT(cr_application[ApplicationID]), cr_application[Amount] >= [High Value Threshold], cr_application[Status] IN {"Approved", "Rejected"}),
    0
)

```

### Layout Mockup

```text
+---------------------------------------------------------------------------------------------------+
|  [KPI: Total HV Exposure ($)] [KPI: Pending HV Exposure ($)] [KPI: HV App Count] [KPI: HV Approval %]|
+---------------------------------------------------------------+-----------------------------------+
|  High-Value Exposure Concentration                            | High-Value Approval Trend Line    |
|  (Treemap Visual)                                             | (Line Chart)                      |
|   - Grouping: Loan Type & Loan Term                           |   - X-Axis: Month                 |
|   - Size: Sum of Requested Loan Amount                        |   - Lines: Approved Amount ($) vs |
|   - Color: Status                                             |            Rejected Amount ($)    |
+---------------------------------------------------------------+-----------------------------------+
|  Top Pending High-Value Applications Review Queue                                                 |
|  (Table Visual with Conditional Formatting on High Confidence/Missing Docs)                       |
|   - Columns: App ID | Applicant Name | Requested Amount | Income | Confidence Score | Assigned To |
+---------------------------------------------------------------------------------------------------+

```

---

# Page 4: AI Extraction & Governance

### DAX Measures

```dax
// 1. Average OCR Confidence Score %
Avg Extraction Confidence = 
AVERAGE(cr_loandocument[ConfidenceScore])

// 2. Straight-Through Processing (STP) Rate % (Processed without manual override)
STP Rate % = 
DIVIDE(
    CALCULATE(
        COUNT(cr_loandocument[DocumentID]),
        cr_loandocument[ExtractionStatus] = "Processed",
        cr_loandocument[ConfidenceScore] >= 0.85
    ),
    COUNT(cr_loandocument[DocumentID]),
    0
)

// 3. Missing / Failed Document Count
Failed Document Verification Count = 
CALCULATE(
    COUNT(cr_loandocument[DocumentID]),
    cr_loandocument[ExtractionStatus] IN {"Failed", "Requires Manual Review"}
)

```

### Layout Mockup

```text
+---------------------------------------------------------------------------------------------------+
|  [KPI: Avg OCR Confidence Score] [KPI: STP Rate %] [KPI: Failed Verifications] [KPI: Total Processed]|
+---------------------------------------------------------------+-----------------------------------+
|  Confidence Score Distribution by Document Type               | Extraction Exception Categories   |
|  (Column Chart)                                               | (Donut Chart)                     |
|   - X-Axis: Document Type (ID, Tax Return, Income Proof, etc.)|   - Slices: Low Confidence,       |
|   - Y-Axis: Average Confidence Score %                        |     Missing Required Field,       |
|   - Threshold Line at 85% Target                              |     Unreadable Document/Scan      |
+---------------------------------------------------------------+-----------------------------------+
|  AI Recommendation vs. Human Decision Alignment                                                   |
|  (Matrix / Cross-Tab)                                                                             |
|   - Rows: AI Suggested Decision (Approve / Review / Reject)                                       |
|   - Columns: Actual Human Decision (Approved / Rejected)                                          |
|   - Values: Application Count (Highlights model alignment vs. human overrides)                     |
+---------------------------------------------------------------------------------------------------+

```

---

# Page 5: Executive Portfolio Yield & Forecast

### DAX Measures

```dax
// 1. Total Originated Portfolio Value ($)
Total Originated Value = 
CALCULATE(
    SUM(cr_application[Amount]),
    cr_application[Status] = "Approved"
)

// 2. Projected Annual Interest Yield ($)
Projected Interest Yield = 
SUMX(
    FILTER(cr_application, cr_application[Status] = "Approved"),
    cr_application[Amount] * cr_application[InterestRate]
)

// 3. Month-over-Month Growth %
MoM Volume Growth % = 
VAR CurrentMonth = [Total Applications]
VAR PriorMonth = 
    CALCULATE(
        [Total Applications],
        DATEADD('Calendar'[Date], -1, MONTH)
    )
RETURN
DIVIDE(CurrentMonth - PriorMonth, PriorMonth, 0)

```

### Layout Mockup

```text
+---------------------------------------------------------------------------------------------------+
|  [KPI: Originated Value ($)]  [KPI: Projected Yield ($)]  [KPI: MoM Growth %]  [KPI: Rejection Rate %]|
+---------------------------------------------------------------+-----------------------------------+
|  Origination Trend: Actual vs. Forecast                       | Yield Analysis by Loan Type       |
|  (Combo Line and Clustered Column Chart)                      | (Waterfall Chart)                 |
|   - Columns: Monthly Approved Volume ($)                      |   - Categories: Auto, Business,   |
|   - Line: Forecasted Origination Target                       |     Home, Personal                |
|                                                               |   - Values: Projected Interest    |
+---------------------------------------------------------------+-----------------------------------+
|  Rejection Cause Decomposition                                                                    |
|  (Decomposition Tree)                                                                             |
|   - Root: Rejected Applications Count                             |                                   |
|   - Branch 1: Loan Type -> Branch 2: Primary Rejection Factor (DTI, Income Unverified, High Risk) |
+---------------------------------------------------------------------------------------------------+

```
