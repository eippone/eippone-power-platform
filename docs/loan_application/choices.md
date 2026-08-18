Absolutely. Since your MVP already has **Dataverse + Customer Loan Application Canvas App + Employee Review Model-Driven App + Power Automate approval flow + Power BI dashboard + Power Pages**, I would implement the AI enhancement as a **controlled AI layer on top of the existing platform**, rather than redesigning the MVP.

The most important architectural principle is:

> **Copilot Studio should orchestrate conversations; Dataverse remains the operational system of record; AI Builder performs document extraction; Power Automate performs deterministic business processes; Power BI remains the governed analytics layer; and humans remain responsible for lending decisions.**

Microsoft's current architecture for AI document processing follows essentially this pattern: AI Builder extracts information, Power Automate orchestrates it, Dataverse stores the structured data, Power Apps provides human review, and Power BI provides analytics. ([Microsoft Learn][1])

---

# 1. Target AI Architecture

I recommend evolving your existing platform into this architecture:

```text
                         LOAN APPLICATION DIGITALIZATION
                                      PLATFORM
                                            |
                 +--------------------------+--------------------------+
                 |                          |                          |
                 v                          v                          v
        CUSTOMER EXPERIENCE         EMPLOYEE EXPERIENCE        EXECUTIVE EXPERIENCE
                 |                          |                          |
          Power Pages                Model-Driven App             Power BI
                 |                          |                          |
                 v                          v                          v
        Customer Copilot          Employee Copilot              AI Insights
                 |                          |                          |
                 +--------------------------+--------------------------+
                                            |
                                            v
                                  COPILOT STUDIO
                                  AI ORCHESTRATION
                                            |
                    +-----------------------+-----------------------+
                    |                       |                       |
                    v                       v                       v
                Dataverse             Power Automate           Power BI
             Operational Data          Business Logic         Semantic Model
                    |                       |
                    |                       |
                    v                       v
             Loan Applications       Approval Workflow
             Documents               Notifications
             Reviews                 Alerts
             AI Results
                    |
                    v
               AI BUILDER
          Document Intelligence
                    |
                    v
          PDF / ID / Income / Tax
             / Supporting Docs
```

Your existing **EIPPONE AI Platform** Power BI workspace can remain the analytics workspace for this solution, while the AI components are added around it.

---

# 2. First: Create an AI Solution in Power Platform

Before creating agents and flows, put the new components into a dedicated solution.

Go to:

**Power Apps → Solutions → New solution**

Create:

```text
Display name:
EIPPONE Loan AI

Name:
EIPPONE_LoanAI

Publisher:
EIPPONE

Version:
1.0.0.0
```

Eventually your solution should contain:

```text
EIPPONE Loan AI
│
├── Dataverse Tables
│   ├── Loan Application
│   ├── Loan Document
│   ├── AI Extraction Result
│   ├── AI Validation Finding
│   ├── AI Recommendation
│   ├── AI Processing Log
│   ├── AI Configuration
│   └── AI Insight
│
├── Power Automate
│   ├── Document Processing
│   ├── Document Validation
│   ├── High Value Alert
│   ├── Application Summary
│   └── Executive Insight Generation
│
├── Copilot Studio
│   ├── Customer Loan Assistant
│   └── Employee Loan Assistant
│
└── AI Builder
    └── Loan Document Processing Model
```

This will make your architecture much easier to maintain and demonstrate professionally.

---

# 3. Step 1 — Extend Your Dataverse Data Model

Your existing **Loan Application** table should remain the central business entity.

Do **not** put all AI information directly into the Loan Application table.

Instead, create related tables.

## 3.1 Loan Document

Create:

**Loan Document**

Suggested columns:

| Column                | Type           |
| --------------------- | -------------- |
| Document ID           | Autonumber     |
| Loan Application      | Lookup         |
| Document Type         | Choice         |
| Document Name         | Text           |
| Document Status       | Choice         |
| Document Date         | Date           |
| File                  | File           |
| AI Processing Status  | Choice         |
| Extraction Confidence | Decimal        |
| Missing Information   | Multiline text |
| Review Required       | Yes/No         |
| Reviewed By           | Lookup User    |
| Reviewed Date         | Date           |

Document Type:

```text
Application
Identification
Proof of Income
Tax Return
Bank Statement
Supporting Document
Other
```

Document Status:

```text
Uploaded
Processing
Processed
Needs Review
Validated
Rejected
```

---

# 4. Create the AI Extraction Result Table

Create:

**AI Extraction Result**

This table stores what AI Builder extracted.

Suggested columns:

| Column               | Type           |
| -------------------- | -------------- |
| Extraction ID        | Autonumber     |
| Loan Application     | Lookup         |
| Loan Document        | Lookup         |
| Customer Name        | Text           |
| Address              | Multiline Text |
| Loan Amount          | Currency       |
| Loan Term            | Whole Number   |
| Interest Rate        | Decimal        |
| Annual Income        | Currency       |
| Employer Name        | Text           |
| Document Type        | Choice         |
| Overall Confidence   | Decimal        |
| Extraction Status    | Choice         |
| Extraction Timestamp | Date/Time      |

Why separate this from Loan Application?

Because you need to distinguish:

**what the customer actually entered**

from

**what AI extracted from a document.**

That distinction becomes very important during human review.

---

# 5. Create AI Validation Finding

Create:

**AI Validation Finding**

Columns:

| Column            | Type           |
| ----------------- | -------------- |
| Finding ID        | Autonumber     |
| Loan Application  | Lookup         |
| Document          | Lookup         |
| Finding Type      | Choice         |
| Severity          | Choice         |
| Description       | Multiline Text |
| Source Value      | Text           |
| Expected Value    | Text           |
| AI Confidence     | Decimal        |
| Resolution Status | Choice         |
| Reviewer Comment  | Multiline Text |

Finding Type:

```text
Missing Information
Mismatch
Low Confidence
Invalid Value
Duplicate Document
Expired Document
Unsupported Document
Potential Data Quality Issue
```

Severity:

```text
Informational
Warning
High
Critical
```

---

# 6. Create AI Recommendation

Create:

**AI Recommendation**

Suggested columns:

| Column              | Type           |
| ------------------- | -------------- |
| Recommendation ID   | Autonumber     |
| Loan Application    | Lookup         |
| Recommendation      | Choice         |
| Explanation         | Multiline Text |
| Risk Indicators     | Multiline Text |
| Supporting Findings | Multiline Text |
| AI Confidence       | Decimal        |
| Generated Date      | Date/Time      |
| Model Version       | Text           |
| Human Decision      | Choice         |
| Human Comment       | Multiline Text |

Recommendation should be something like:

```text
Requires Review
```

or:

```text
Documentation Complete
```

or:

```text
Potential Issue Detected
```

rather than automatically:

```text
Approve Loan
Reject Loan
```

That reinforces the principle that AI is **decision support**, not the lending decision.

---

# 7. Create AI Configuration

This is especially important for your high-value threshold.

Create:

**AI Configuration**

Example:

| Setting                     |  Value |
| --------------------------- | -----: |
| High Value Loan Threshold   | 150000 |
| Low Confidence Threshold    |   0.80 |
| AI Recommendation Enabled   |    Yes |
| Executive Insight Enabled   |    Yes |
| Document Processing Enabled |    Yes |

You can then change:

```text
150000
```

to:

```text
200000
```

without modifying your Power Automate flows.

---

# 8. Step 2 — Build the AI Builder Document Model

Now implement document intelligence.

Go to:

**Power Apps → AI Hub → AI Builder**

Choose:

**Document processing**

Microsoft's current document-processing model requires you to define the information to extract and train/publish the model before using it in Power Automate or Power Apps. Microsoft notes that five documents can be enough to get started with a model. ([Microsoft Learn][2])

For your project, I recommend starting with:

### Model 1

```text
Loan Application Document Processor
```

Don't attempt every document type immediately.

Start with:

```text
Loan Application PDF
```

Then expand.

---

# 9. Define Fields to Extract

Configure the model to extract:

```text
CustomerName
Address
LoanAmount
LoanTerm
InterestRate
AnnualIncome
EmployerName
ApplicationNumber
ApplicationDate
```

If the document contains tables, you can also extract relevant table information.

AI Builder document processing supports PDF, JPG and PNG input, including scanned documents, with a current maximum document size of 20 MB. ([Microsoft Learn][3])

---

# 10. Train the AI Builder Model

Upload representative sample documents.

For example:

```text
Application_001.pdf
Application_002.pdf
Application_003.pdf
Application_004.pdf
Application_005.pdf
```

Tag:

```text
Customer Name
Address
Loan Amount
Loan Term
Interest Rate
Income
```

Then:

**Train → Evaluate → Publish**

Do not immediately connect it to your production workflow.

First test it with documents it has never seen.

---

# 11. Create the Document Upload Process

Your architecture should now be:

```text
Customer
   |
   v
Power Pages
   |
   v
Loan Application
   |
   v
Upload Document
   |
   v
Dataverse Loan Document
   |
   v
Power Automate
   |
   v
AI Builder
```

You can use a Dataverse file column for the uploaded document.

---

# 12. Build the AI Builder Power Automate Flow

Create:

**PA – Process Loan Document**

Trigger:

```text
When a row is added
```

Table:

```text
Loan Document
```

Condition:

```text
AI Processing Status = Uploaded
```

Then:

```text
Get file content
        |
        v
AI Builder – Process documents
        |
        v
Extract fields
        |
        v
Create AI Extraction Result
        |
        v
Calculate confidence
        |
        v
Validate extracted information
        |
        v
Update Loan Document
```

Microsoft currently calls the AI Builder Power Automate action **Process documents**; the extracted fields can then be used by subsequent actions. ([Microsoft Learn][4])

---

# 13. Store AI Confidence

This is extremely important.

Suppose AI Builder returns:

```text
Loan Amount = $185,000
Confidence = 0.96
```

Store:

```text
Loan Amount = 185000
Confidence = 0.96
```

But if:

```text
Annual Income = $82,000
Confidence = 0.61
```

then create:

```text
AI Validation Finding

Type:
Low Confidence

Severity:
High

Description:
Income value extracted with low confidence.

Confidence:
61%
```

This creates a proper **human-in-the-loop** architecture.

---

# 14. Create Document Validation Flow

After extraction, compare:

```text
Customer-entered data
        VS
AI-extracted data
```

For example:

```text
Application Loan Amount = $175,000
Document Loan Amount    = $175,000
```

→ Match.

But:

```text
Application Loan Amount = $175,000
Document Loan Amount    = $195,000
```

→ Create:

```text
AI Validation Finding

Finding Type:
Mismatch

Severity:
High
```

---

# 15. Create the Approver Summary

Now create a Power Automate flow:

```text
PA – Generate Application AI Summary
```

The flow gathers:

```text
Loan Application
       |
       +--- Customer information
       +--- Loan information
       +--- Documents
       +--- Extraction results
       +--- Validation findings
       +--- AI recommendations
```

Then create a structured summary.

Example:

```text
APPLICATION SUMMARY

Application:
APP-1002

Customer:
John Smith

Requested Loan:
$175,000

Term:
60 months

Interest Rate:
6.5%

Annual Income:
$92,000

Documents Received:
✓ Loan Application
✓ Identification
✓ Proof of Income
✓ Tax Return

Issues:
⚠ Income extracted with 74% confidence

Validation:
1 mismatch detected

AI Assessment:
Requires Human Review

Recommendation:
Verify income documentation before proceeding.
```

---

# 16. Add AI Summary to the Employee Review App

Modify your existing **Employee Review Model-Driven App**.

Add a new tab:

```text
AI Review
```

Add sections:

### Application Summary

```text
Customer
Loan Amount
Term
Rate
Income
```

### Documents

```text
Document
Status
Confidence
```

### AI Findings

```text
Finding
Severity
Confidence
Resolution
```

### AI Recommendation

```text
Recommendation
Explanation
```

### Human Decision

```text
Approve
Reject
Request Information
```

The employee should be able to correct AI-extracted information.

---

# 17. Step 3 — Implement High-Value Loan Detection

Create:

**PA – High Value Loan Alert**

Trigger:

```text
When Loan Application is created or updated
```

Retrieve:

```text
High Value Loan Threshold
```

from:

```text
AI Configuration
```

Then:

```text
Loan Amount >= Threshold
```

If true:

```text
Update Loan Application
    High Value = Yes

Create Alert
    |
    +--> Employee Review Queue
    +--> Email
    +--> Power BI
    +--> Copilot
```

For example:

```text
Configured threshold = $150,000

Application = $175,000

RESULT:

High Value = Yes
```

Do **not** hard-code `$150,000` into multiple flows.

---

# 18. Add High-Value Fields to Loan Application

Add:

```text
High Value Loan       Yes/No
High Value Threshold  Currency
High Value Alert Date Date/Time
```

You can then create a Power BI measure:

```DAX
High Value Applications =
CALCULATE(
    COUNTROWS('Loan Application'),
    'Loan Application'[High Value Loan] = TRUE()
)
```

---

# 19. Step 4 — Build the Customer Copilot

Now implement the customer-facing conversational assistant.

Create an agent in:

**Microsoft Copilot Studio**

Name:

```text
EIPPONE Customer Loan Assistant
```

Purpose:

> Assist customers with loan application questions, application status, required documentation, submission guidance and navigation of the digital loan application.

---

# 20. Customer Copilot Knowledge

Start with **non-sensitive information**.

Create knowledge sources for:

```text
Loan Process FAQ
Required Documents
Application Instructions
Submission Instructions
Contact Information
General Loan Process
```

Do not put customer-specific application data into static knowledge.

For example:

### Knowledge

```text
What documents are required?

Customers generally need identification,
proof of income and supporting documentation.
```

But:

### Transactional information

```text
What is the status of my application?
```

must come from Dataverse.

---

# 21. Create Customer Topics

Create topics for:

```text
Check Application Status
Required Documents
Loan Process
Application Submission
Document Upload
Application Navigation
General Assistance
Escalation
```

---

# 22. Implement Application Status

This is one of the most important parts.

Create a Power Automate flow:

```text
PA – Get Customer Application Status
```

Input:

```text
Authenticated User
```

The flow retrieves the customer's authorized application.

Then returns:

```text
Application Number
Status
Last Updated
```

For example:

```text
APP-1002
Under Review
August 10, 2026
```

The Copilot then responds:

> Your application APP-1002 is currently Under Review.

---

# 23. Security for Customer Copilot

This is where you must be very careful.

**Do not create a flow that simply accepts:**

```text
Application ID = APP-1002
```

and returns the record.

Otherwise someone could potentially ask:

```text
What is APP-1003?
```

and retrieve another customer's information.

Instead:

```text
Authenticated Customer
        |
        v
Identity
        |
        v
Authorized Dataverse relationship
        |
        v
Customer's application
```

Copilot Studio supports authentication using Microsoft Entra ID or OAuth-based identity providers, and Microsoft explicitly recommends authentication for agents handling restricted information. ([Microsoft Learn][5])

Dataverse itself uses role-based security to control access to data and resources. ([Microsoft Learn][6])

---

# 24. Connect Customer Copilot to Power Pages

Your Power Pages architecture becomes:

```text
                     POWER PAGES
                         |
          +--------------+--------------+
          |                             |
          v                             v
 Customer Loan Application      Customer Copilot
          |                             |
          |                             v
          |                      Copilot Studio
          |                             |
          +-------------+---------------+
                        |
                        v
                    Dataverse
```

Microsoft's current Power Pages architecture supports adding a Copilot Studio agent directly to a Power Pages site, including authenticated agent scenarios. ([Microsoft Learn][7])

For your customer application-status scenario, use an authenticated configuration rather than anonymous access.

---

# 25. Step 5 — Build the Employee Copilot

Create a **separate agent**.

Name:

```text
EIPPONE Employee Loan Assistant
```

Do not combine the employee and customer agents.

The employee agent can access much more information.

Purpose:

> Assist authorized employees with operational loan application analysis, application review, document status, summaries and governed business metrics.

---

# 26. Employee Copilot Questions

Create topics/tools for:

```text
Applications Under Review
High Value Applications
Longest Waiting Applications
Missing Documents
Application Summary
Approval Rate
Processing Time
Operational Bottlenecks
```

---

# 27. Use Agent Flows / Power Automate as Tools

This is one of the most important architectural decisions.

Do not allow the LLM to directly construct arbitrary Dataverse queries for sensitive operational data.

Instead create controlled tools.

For example:

```text
Employee asks:
"Which applications are over $150,000 and under review?"

             |
             v
       Employee Copilot
             |
             v
 Get High Value Applications Tool
             |
             v
       Power Automate
             |
             v
          Dataverse
             |
             v
       Authorized Results
             |
             v
       Copilot Summary
```

Copilot Studio supports using agent flows as tools so the agent can call deterministic workflows at runtime. ([Microsoft Learn][8])

---

# 28. Create Tool #1 — Applications Under Review

Flow:

```text
PA – Get Applications Under Review
```

Input:

```text
Employee
```

Query:

```text
Status = Under Review
```

Return:

```text
Application ID
Customer
Loan Amount
Submission Date
Days Waiting
Assigned Reviewer
```

The Copilot summarizes the results.

---

# 29. Tool #2 — High-Value Applications

Create:

```text
PA – Get High Value Applications
```

Inputs:

```text
Minimum Loan Amount
Status
```

Example:

```text
Minimum Loan Amount = 150000
Status = Under Review
```

Return only authorized records.

The employee asks:

> Show me applications over $150,000 that are still under review.

Copilot calls:

```text
Get High Value Applications
```

and returns the results.

---

# 30. Tool #3 — Longest Waiting Applications

Create a calculated field:

```text
Days Waiting
```

or calculate it in Power Automate/Dataverse.

Then:

```text
Sort by Days Waiting DESC
```

Return:

```text
APP-1023 — 17 days
APP-1009 — 15 days
APP-1018 — 13 days
```

---

# 31. Tool #4 — Missing Documents

Create:

```text
PA – Get Missing Documents
```

Query:

```text
Document Status = Missing
```

Return:

```text
Application
Customer
Missing Document
Days Outstanding
```

---

# 32. Tool #5 — Application Summary

Create:

```text
PA – Get Application Summary
```

Input:

```text
Application ID
```

Then retrieve:

```text
Loan Application
Documents
Extraction Results
Validation Findings
AI Recommendation
Approval History
```

Return a structured object.

Copilot converts it into natural language.

---

# 33. Step 6 — Build Governed Power BI Metrics

This is critical for your executive AI.

Do **not** let the executive agent calculate important KPIs from raw records independently.

Instead, define governed measures in your Power BI semantic model.

For example:

```DAX
Total Applications =
COUNTROWS('Loan Application')
```

```DAX
Approved Applications =
CALCULATE(
    [Total Applications],
    'Loan Application'[Status] = "Approved"
)
```

```DAX
Approval Rate =
DIVIDE(
    [Approved Applications],
    [Total Applications]
)
```

```DAX
Applications Under Review =
CALCULATE(
    [Total Applications],
    'Loan Application'[Status] = "Under Review"
)
```

```DAX
Average Processing Time =
AVERAGEX(
    'Loan Application',
    'Loan Application'[Processing Days]
)
```

```DAX
High Value Applications =
CALCULATE(
    [Total Applications],
    'Loan Application'[High Value Loan] = TRUE()
)
```

---

# 34. Improve Your Power BI Semantic Model

Your current Power BI dashboard connects to Dataverse.

Now move toward:

```text
FactLoanApplication
        |
        +---- DimDate
        +---- DimCustomer
        +---- DimLoanStatus
        +---- DimLoanType
        +---- DimDocument
```

This will make your AI reporting considerably more reliable.

For example:

```text
FactLoanApplication
       |
       +-- Application ID
       +-- Loan Amount
       +-- Processing Days
       +-- Approval Flag
       +-- High Value Flag
       +-- Submission Date
       +-- Decision Date
```

---

# 35. Step 7 — Build Executive AI Insights

Create:

```text
PA – Generate Executive Loan Insight
```

Trigger:

```text
Scheduled
```

For example:

```text
Every morning
```

or:

```text
Every Monday
```

Retrieve governed metrics:

```text
Application Volume
Approval Rate
Average Processing Time
High Value Applications
Under Review Backlog
Rejection Rate
Missing Document Rate
```

Compare:

```text
Current period
VS
Previous period
```

For example:

```text
This month:
480 applications

Previous month:
410 applications
```

Calculate:

```text
+17.1%
```

---

# 36. Generate Structured Insight

Rather than asking AI to invent an executive report from raw data, give it structured metrics.

Example input:

```text
Application Volume:
480

Previous Period:
410

Approval Rate:
68%

Previous Approval Rate:
72%

Average Processing Time:
5.8 days

Previous:
4.2 days

Under Review:
83

High Value Under Review:
21
```

Then generate:

```text
Executive Insight

Application volume increased by 17.1%.

Approval rate decreased from 72% to 68%.

Average processing time increased from
4.2 days to 5.8 days.

The under-review queue currently contains
83 applications, including 21 high-value applications.

Management attention is recommended for the
growing review backlog and processing-time increase.
```

---

# 37. Store Executive Insights in Dataverse

Create:

**AI Insight**

Columns:

```text
Insight ID
Insight Type
Reporting Period
Metric
Current Value
Previous Value
Change %
Severity
Insight Text
Source
Generated Date
Model Version
```

Example:

```text
Insight Type:
Executive

Metric:
Average Processing Time

Change:
+38%

Severity:
High

Source:
Power BI Loan Semantic Model
```

This creates traceability.

---

# 38. Display AI Insights in Power BI

Add an executive section to your existing Power BI report:

```text
EXECUTIVE AI INSIGHTS
```

Example cards:

```text
┌─────────────────────────────────────────┐
│ EXECUTIVE AI INSIGHT                    │
│                                         │
│ Application volume increased 17.1%.     │
│                                         │
│ Processing time increased from          │
│ 4.2 to 5.8 days.                        │
│                                         │
│ ⚠ Review backlog requires attention.   │
└─────────────────────────────────────────┘
```

Because the underlying insight is stored alongside the source metrics, the report can show:

```text
Insight
+
Source metrics
+
Reporting period
```

rather than presenting an unexplained AI statement.

---

# 39. Step 8 — Conversational Power BI Analysis

There are actually **two different capabilities** you should distinguish.

### Capability A — Power BI Copilot

Power BI's own Copilot can answer natural-language questions against a semantic model and return results/visualizations. ([Microsoft Learn][9])

### Capability B — Your Employee Copilot

Your Copilot Studio agent can call controlled tools that retrieve governed metrics and operational data.

For your project, I recommend:

```text
Employee Copilot
       |
       +---- Operational questions
       |       |
       |       v
       |    Dataverse
       |
       +---- KPI questions
       |       |
       |       v
       |    Power BI governed metrics
       |
       +---- Document questions
               |
               v
           AI Builder results
```

That is much stronger architecturally than trying to make one LLM directly access everything.

---

# 40. Create KPI Tools

Create Power Automate tools such as:

```text
Get Approval Rate
Get Application Volume
Get Average Processing Time
Get Under Review Count
Get High Value Count
Get Rejection Rate
Get Document Exception Rate
Get Backlog
```

Then the employee asks:

> What is our approval rate?

Copilot calls:

```text
Get Approval Rate
```

which retrieves the governed KPI.

Response:

```text
The current approval rate is 68%.

This represents 326 approved applications
out of 480 applications.
```

---

# 41. Create the "Portfolio Summary" Tool

Create:

```text
PA – Get Portfolio Summary
```

Return:

```text
Total Applications
Approved
Rejected
Under Review
Approval Rate
Average Processing Time
High Value Applications
Missing Document Rate
```

Then the employee can ask:

> Summarize this month's loan portfolio performance.

The Copilot can produce:

```text
Portfolio Summary

480 applications were received this month.

Approval rate:
68%

Average processing time:
5.8 days

Applications under review:
83

High-value applications:
42

The main operational concern is the increase
in processing time and the size of the
under-review backlog.
```

---

# 42. Security Architecture

This should be implemented **before** exposing the AI agents.

You effectively have three security zones:

```text
                  SECURITY
                     |
       +-------------+-------------+
       |             |             |
       v             v             v
   CUSTOMER       EMPLOYEE      EXECUTIVE
       |             |             |
       v             v             v
 Power Pages    Model-Driven    Power BI
       |             |             |
       v             v             v
 Customer       Employee       Management
 Data           Data           Metrics
```

## Customer

Can see:

```text
Own application
Own documents
Own status
Own messages
```

Cannot see:

```text
Other customers
Internal reviews
AI recommendation details
Other loan applications
```

## Employee

Can see authorized operational data.

## Executive

Can see governed aggregate reporting.

---

# 43. Dataverse Security

Configure security roles for:

```text
Customer
Loan Officer
Reviewer
Manager
Administrator
```

Do not give every user organization-level access.

Dataverse supports different privilege access levels, including organization-level access, and Microsoft recommends restricting broad organization access according to the organization's security plan. ([Microsoft Learn][10])

---

# 44. Power Pages Table Permissions

For your customer-facing Power Pages site:

Configure table permissions for:

```text
Loan Application
Loan Document
```

The customer should have:

```text
Read → Own records
Create → Own records
Write → Own records
```

depending on your application workflow.

Do not configure:

```text
Read → All records
```

for customers.

---

# 45. Employee Copilot Security

For employee tools, make the Power Automate flows return only authorized data.

For example:

```text
Employee
   |
   v
Copilot
   |
   v
Power Automate Tool
   |
   v
Dataverse Security
   |
   v
Authorized Applications
```

Do not rely only on the Copilot prompt:

> "Don't show unauthorized applications."

That is not a security mechanism.

**Authorization must be enforced by the underlying data/service layer.**

---

# 46. AI Recommendation Governance

I strongly recommend that you explicitly separate:

```text
AI Recommendation
```

from:

```text
Human Decision
```

Your data model should therefore contain:

```text
AI Recommendation:
Requires Review

AI Explanation:
Income documentation has a low extraction confidence.

Human Decision:
Approved

Human Reviewer:
Jane Doe

Human Decision Date:
2026-08-10

Human Comment:
Income verified against source document.
```

This creates an auditable trail.

---

# 47. Complete End-to-End Document Workflow

Your final document pipeline should look like this:

```text
Customer
   |
   v
Power Pages
   |
   v
Upload PDF
   |
   v
Dataverse
Loan Document
   |
   v
Power Automate
   |
   v
AI Builder
   |
   +------------------+
   |                  |
   v                  v
Extract Data       Confidence
   |                  |
   +--------+---------+
            |
            v
      AI Extraction
         Result
            |
            v
       Validation
            |
       +----+----+
       |         |
       v         v
     Match    Exception
       |         |
       |         v
       |    AI Finding
       |         |
       +----+----+
            |
            v
     AI Recommendation
            |
            v
      Human Reviewer
            |
            v
      Final Decision
```

---

# 48. Complete Conversational Architecture

You will end up with **two conversational agents**.

```text
                         COPILOT STUDIO
                              |
                +-------------+-------------+
                |                           |
                v                           v
       CUSTOMER ASSISTANT           EMPLOYEE ASSISTANT
                |                           |
                v                           v
          Power Pages                Employee Portal
                |                           |
                |                  +--------+--------+
                |                  |        |        |
                v                  v        v        v
          Customer Data       Dataverse  Power BI  AI Builder
```

---

# 49. What the Customer Can Ask

```text
Customer:
What is the status of my application?

Copilot:
Your application APP-1002 is currently Under Review.
```

```text
Customer:
What documents do I need?

Copilot:
You need identification, proof of income,
and the supporting documents listed for your
application.
```

```text
Customer:
How do I upload my documents?

Copilot:
Go to My Application → Documents → Upload Document.
```

---

# 50. What the Employee Can Ask

```text
Employee:
Which applications are under review?

Copilot:
There are currently 83 applications under review.
```

```text
Employee:
Show me applications over $150,000 that are
still under review.

Copilot:
There are 21 authorized applications matching
those criteria.
```

```text
Employee:
Which applications have been waiting the longest?

Copilot:
The five longest-waiting applications are...
```

```text
Employee:
Summarize APP-1002.

Copilot:
APP-1002 requests $175,000 over 60 months...
```

---

# 51. What Executives Can Ask

```text
Executive:
What is our approval rate?

Copilot:
The current approval rate is 68%.
```

```text
Executive:
How many applications are under review?

Copilot:
83 applications are currently under review.
```

```text
Executive:
Are high-value loans increasing?

Copilot:
High-value applications increased 12%
compared with the previous reporting period.
```

```text
Executive:
Summarize this month's performance.

Copilot:
Application volume increased 17.1%, while
average processing time increased from
4.2 to 5.8 days...
```

---

# 52. Recommended Implementation Order

Do **not** build everything simultaneously.

I recommend this sequence:

### Phase 1 — AI data foundation

```text
1. Create EIPPONE Loan AI solution
2. Extend Dataverse
3. Create Loan Document
4. Create AI Extraction Result
5. Create AI Validation Finding
6. Create AI Recommendation
7. Create AI Configuration
8. Create AI Insight
```

### Phase 2 — AI Builder

```text
9. Create document processing model
10. Train model
11. Publish model
12. Create document processing flow
13. Store extraction results
14. Store confidence
15. Create validation findings
```

### Phase 3 — Employee Review

```text
16. Add AI Review tab
17. Display extracted information
18. Display confidence
19. Display validation findings
20. Display AI recommendation
21. Add human correction
22. Add human decision
```

### Phase 4 — High-value automation

```text
23. Create AI Configuration
24. Configure threshold
25. Create high-value flow
26. Add High Value flag
27. Add employee notification
28. Add review queue
29. Add Power BI metric
```

### Phase 5 — Customer Copilot

```text
30. Create Customer Loan Assistant
31. Add FAQ knowledge
32. Add document guidance
33. Add submission guidance
34. Configure authentication
35. Create application-status flow
36. Connect flow to Copilot
37. Add to Power Pages
38. Test customer security
```

### Phase 6 — Employee Copilot

```text
39. Create Employee Loan Assistant
40. Create Applications Under Review tool
41. Create High Value Applications tool
42. Create Longest Waiting tool
43. Create Missing Documents tool
44. Create Application Summary tool
45. Connect tools to Copilot
46. Test authorization
```

### Phase 7 — Executive AI

```text
47. Define governed Power BI measures
48. Create portfolio KPI tools
49. Create executive insight flow
50. Generate period-over-period insights
51. Store AI insights in Dataverse
52. Display insights in Power BI
53. Add conversational KPI analysis
```

---

# 53. The Final Architecture You Can Present

For your project documentation, I would describe the final platform as:

```text
                         EIPPONE
             LOAN APPLICATION DIGITALIZATION
                         PLATFORM
                              |
       +----------------------+----------------------+
       |                      |                      |
       v                      v                      v
 CUSTOMER CHANNEL       EMPLOYEE CHANNEL       EXECUTIVE CHANNEL
       |                      |                      |
 Power Pages             Model-Driven App        Power BI
       |                      |                      |
 Customer Copilot        Employee Copilot        AI Insights
       |                      |                      |
       +----------------------+----------------------+
                              |
                              v
                       COPILOT STUDIO
                      AI ORCHESTRATION
                              |
        +---------------------+----------------------+
        |                     |                      |
        v                     v                      v
    DATAVERSE           POWER AUTOMATE          POWER BI
 Operational Data       Business Logic       Governed Metrics
        |                     |
        |                     +------> Alerts
        |                     +------> Approvals
        |                     +------> AI Tools
        |
        v
   AI BUILDER
Document Intelligence
        |
        +---- Application
        +---- ID
        +---- Income
        +---- Tax
        +---- Supporting Documents
        |
        v
 AI Extraction / Validation
        |
        v
 HUMAN REVIEW
        |
        v
 FINAL LENDING DECISION
```

This architecture is also aligned with Microsoft's current guidance: Copilot Studio agents can integrate with Dataverse and other services through authenticated connections, while agent flows can provide deterministic actions and data retrieval. ([Microsoft Learn][11])

---

## 54. One Important Adjustment to Your Original Roadmap

I would make one architectural change to the wording **"Conversational Power BI Analysis."**

Instead of positioning it as:

```text
Copilot
   ↓
Power BI
   ↓
AI
```

I would document it as:

```text
                         Employee / Executive
                                  |
                                  v
                           Copilot Studio
                                  |
                    +-------------+-------------+
                    |                           |
                    v                           v
             Operational Tools             KPI Tools
                    |                           |
                    v                           v
                Dataverse              Governed Power BI
                                             Semantic Model
                    |                           |
                    +-------------+-------------+
                                  |
                                  v
                         Natural Language
                             Response
```

That distinction is important because **Power BI remains the governed analytical source**, while Copilot Studio is the conversational interface.

Power BI Copilot itself can also answer natural-language questions against a semantic model, but it has its own prerequisites, including administrator enablement and appropriate Fabric capacity. ([Microsoft Learn][9]) Given your current environment, I would therefore build the **Dataverse + Power Automate + governed Power BI semantic model + Copilot Studio** architecture first, and treat deeper Power BI/Fabric Copilot capabilities as a later enhancement.

---

## 55. Your MVP → AI Platform Evolution

The progression of your project becomes very clear:

| Stage                  | Capability                                 |
| ---------------------- | ------------------------------------------ |
| **MVP**                | Digital loan application                   |
| **Automation**         | Power Automate approvals                   |
| **Analytics**          | Power BI dashboard                         |
| **Customer AI**        | Copilot Studio customer assistant          |
| **Employee AI**        | Conversational operational assistant       |
| **Document AI**        | AI Builder extraction                      |
| **Intelligent Review** | Validation + confidence + AI summary       |
| **Proactive AI**       | High-value alerts                          |
| **Executive AI**       | Automated governed insights                |
| **Conversational BI**  | Natural-language KPI analysis              |
| **Future**             | Predictive lending analytics / advanced ML |

The key is that **AI is not replacing your existing Power Platform architecture**. It is becoming an intelligence layer over the architecture you have already built.

If you implement it in this order, your existing **Loan Application → Employee Review → Approval → Power BI** MVP remains intact while progressively adding **Document AI → Customer Copilot → Employee Copilot → Executive AI**.

[1]: https://learn.microsoft.com/en-us/power-platform/architecture/reference-architectures/ai-document-processing?utm_source=chatgpt.com "Streamline document processing with AI Builder - Power Platform | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/ai-builder/form-processing-model-overview?utm_source=chatgpt.com "Document processing model overview - AI Builder | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/ai-builder/form-processing-model-requirements?utm_source=chatgpt.com "Requirements and limitations for document processing models - AI Builder | Microsoft Learn"
[4]: https://learn.microsoft.com/en-us/ai-builder/form-processing-model-in-flow?utm_source=chatgpt.com "Use the document processing model in Power Automate | Microsoft Learn"
[5]: https://learn.microsoft.com/en-us/microsoft-copilot-studio/configuration-end-user-authentication?utm_source=chatgpt.com "Configure user authentication - Microsoft Copilot Studio | Microsoft Learn"
[6]: https://learn.microsoft.com/en-us/power-platform/admin/database-security?utm_source=chatgpt.com "Role-based security roles for Dataverse - Power Platform | Microsoft Learn"
[7]: https://learn.microsoft.com/en-us/power-pages/getting-started/enable-agent?utm_source=chatgpt.com "Add an agent from the setup workspace | Microsoft Learn"
[8]: https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-flow?utm_source=chatgpt.com "Use agent flows with your agent - Microsoft Copilot Studio | Microsoft Learn"
[9]: https://learn.microsoft.com/sr-latn-rs/Power-Bi/create-reports/copilot-ask-data-question?utm_source=chatgpt.com "Ask Copilot questions about your data - Power BI | Microsoft Learn"
[10]: https://learn.microsoft.com/en-us/power-platform/admin/security-roles-privileges?utm_source=chatgpt.com "Security roles and privileges for Dataverse - Power Platform | Microsoft Learn"
[11]: https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-connections?utm_source=chatgpt.com "Create and manage connections - Microsoft Copilot Studio | Microsoft Learn"


Yes — **absolutely**. In fact, for your **Loan Application Digitalization Platform**, I recommend that you design the Dataverse data model so it supports a **star-schema-style analytical model**.

There is one important distinction, though:

> **Dataverse is primarily your operational database (OLTP), while Power BI is your analytical/semantic layer (OLAP).**

So you *can* create fact and dimension tables in Dataverse, but you should be deliberate about which tables are operational versus analytical. Dataverse supports one-to-many, many-to-one, and many-to-many relationships through table relationships and lookup columns. ([Microsoft Learn][1])

Microsoft also recommends star-schema modeling for Power BI because dimensions can filter facts through one-to-many relationships and provide flexible reporting. ([Microsoft Learn][2])

## For your loan platform, I would use this model

```text
                         DIM_DATE
                            |
                            |
DIM_CUSTOMER ------ FACT_LOAN_APPLICATION ------ DIM_LOAN_PRODUCT
                            |
                            |
                     DIM_APPLICATION_STATUS
                            |
                            |
                     DIM_REVIEWER
                            |
                            |
                     DIM_LOAN_TYPE
```

Then have additional fact tables for processes that have their own grain:

```text
                         DIM_DATE
                            |
                            v
                  FACT_LOAN_APPLICATION
                    /       |       \
                   /        |        \
                  v         v         v
          DIM_CUSTOMER  DIM_LOAN   DIM_STATUS


                         DIM_DATE
                            |
                            v
                  FACT_DOCUMENT_PROCESSING
                     /       |       \
                    v        v        v
             DIM_DOCUMENT DIM_CUSTOMER DIM_APPLICATION


                         DIM_DATE
                            |
                            v
                    FACT_APPROVAL
                     /       |       \
                    v        v        v
             DIM_APPLICATION DIM_REVIEWER DIM_STATUS
```

### The key concept

Don't think of **one giant `Loan Application` table** as the entire data warehouse.

Instead, identify the **business processes and their grain**.

---

# 1. Your current table

You currently have something like:

```text
Loan Application
-----------------------------
Application ID
Customer Name
Address
Loan Amount
Loan Term
Interest Rate
Income
Employment
Status
Submission Date
Approval Date
Reviewer
...
```

This is fine for your MVP and Power Apps.

But as you add:

* AI Builder
* documents
* approvals
* reviewers
* alerts
* processing times
* executive reporting
* conversational analytics

the single table becomes increasingly difficult to manage.

---

# 2. I would redesign your Dataverse model

For your project, I would create something closer to this:

```text
                    ┌──────────────────┐
                    │    DIM_DATE      │
                    │──────────────────│
                    │ DateKey           │
                    │ Date              │
                    │ Month             │
                    │ Quarter           │
                    │ Year              │
                    └────────┬─────────┘
                             │
                             │
┌──────────────────┐         │         ┌──────────────────┐
│ DIM_CUSTOMER     │         │         │ DIM_LOAN_PRODUCT │
│──────────────────│         │         │──────────────────│
│ CustomerKey      │         │         │ LoanProductKey   │
│ CustomerID       │         │         │ ProductName      │
│ Name             │         │         │ LoanType         │
│ Province         │         │         │ MinAmount        │
│ IncomeRange      │         │         │ MaxAmount        │
└────────┬─────────┘         │         └────────┬─────────┘
         │                   │                  │
         │                   │                  │
         └───────────────────┼──────────────────┘
                             │
                             ▼
                 ┌─────────────────────────┐
                 │ FACT_LOAN_APPLICATION   │
                 │─────────────────────────│
                 │ ApplicationKey          │
                 │ ApplicationID           │
                 │ CustomerKey             │
                 │ LoanProductKey          │
                 │ StatusKey               │
                 │ SubmissionDateKey       │
                 │ DecisionDateKey         │
                 │ LoanAmount              │
                 │ LoanTerm                │
                 │ InterestRate            │
                 │ AnnualIncome            │
                 │ ProcessingDays          │
                 │ HighValueFlag           │
                 │ ApprovedFlag             │
                 │ RejectedFlag            │
                 └───────────┬─────────────┘
                             │
                  ┌──────────┼───────────┐
                  │          │           │
                  ▼          ▼           ▼
           DIM_STATUS   DIM_REVIEWER   DIM_LOAN_TYPE
```

That is a much better analytical foundation.

---

# 3. But don't turn every Dataverse table into a dimension

This is important.

For example:

### Good dimension

```text
Dim Customer
```

because a customer has descriptive attributes:

```text
Customer
   |
   +-- Name
   +-- Province
   +-- Employment Type
   +-- Income Range
```

### Good fact

```text
Fact Loan Application
```

because an application represents a business event/process and contains measurable values:

```text
Loan Amount
Processing Days
Interest Rate
Approved Flag
High Value Flag
```

---

# 4. Your AI Builder tables should also have their own grain

This becomes particularly interesting for your AI enhancement.

I would have:

```text
FACT_DOCUMENT_PROCESSING
```

with one row per **document processing event**.

For example:

```text
DocumentProcessingKey
ApplicationKey
DocumentKey
ProcessingDateKey
DocumentTypeKey
ExtractionConfidence
ProcessingTime
ValidationIssueCount
LowConfidenceFlag
SuccessfulExtractionFlag
```

Then:

```text
DIM_DOCUMENT
```

contains:

```text
DocumentKey
DocumentType
DocumentDescription
RequiredFlag
```

This lets Power BI answer:

> What percentage of income documents have extraction confidence below 80%?

without touching your operational application table.

---

# 5. You can also create an AI fact table

For your AI roadmap, I would go one step further.

Create:

```text
FACT_AI_REVIEW
```

with:

```text
AIReviewKey
ApplicationKey
DocumentKey
DateKey
AIRecommendationKey
ExtractionConfidence
ValidationIssueCount
LowConfidenceFlag
MismatchFlag
HumanOverrideFlag
HumanDecision
```

Then you can analyze:

```text
AI extraction accuracy
AI confidence
Human overrides
Document issues
AI recommendation vs human decision
```

This will make your project much more impressive from a **data analytics + AI governance** perspective.

---

# 6. Your approval process should also become a fact

You already have a Power Automate approval workflow.

Instead of putting everything into `Loan Application`, consider:

```text
FACT_APPROVAL
```

Example:

```text
ApprovalKey
ApplicationKey
ReviewerKey
StatusKey
SubmittedDateKey
DecisionDateKey
ApprovalDurationHours
Decision
```

Now Power BI can calculate:

```DAX
Average Approval Time
```

or:

```DAX
Approval Rate
```

without confusing application creation with approval events.

---

# 7. Your final Dataverse architecture could look like this

For the version of the project you're building, I would target:

```text
DATAVERSE
│
├── OPERATIONAL TABLES
│   │
│   ├── Customer
│   ├── Loan Application
│   ├── Loan Document
│   ├── Application Review
│   └── Approval
│
├── AI TABLES
│   │
│   ├── AI Extraction Result
│   ├── AI Validation Finding
│   ├── AI Recommendation
│   ├── AI Processing Log
│   └── AI Insight
│
└── ANALYTICAL TABLES
    │
    ├── Fact Loan Application
    ├── Fact Document Processing
    ├── Fact Approval
    ├── Fact AI Review
    │
    ├── Dim Customer
    ├── Dim Date
    ├── Dim Loan Product
    ├── Dim Loan Type
    ├── Dim Application Status
    ├── Dim Document
    └── Dim Reviewer
```

However, I **wouldn't necessarily duplicate all of these into Dataverse immediately**.

---

# 8. My recommendation for YOUR project

Because you already have a working Power Platform MVP, I would use a **hybrid architecture**:

```text
                 POWER APPS / POWER PAGES
                          │
                          ▼
                     DATAVERSE
                    Operational
                       Model
                          │
          ┌───────────────┼────────────────┐
          │               │                │
          ▼               ▼                ▼
    Power Automate     AI Builder      Copilot Studio
          │               │                │
          └───────────────┼────────────────┘
                          │
                          ▼
                 CURATED ANALYTICS
                          │
                          ▼
                  POWER BI / FABRIC
                          │
                          ▼
                    STAR SCHEMA
```

In other words:

### Dataverse

Optimized for:

* Power Apps
* Power Pages
* workflows
* application transactions
* security
* AI processing results

### Power BI/Fabric

Optimized for:

* fact tables
* dimension tables
* historical analysis
* DAX
* KPIs
* executive reporting
* conversational analytics

This separation is usually cleaner than trying to make Dataverse simultaneously behave like an OLTP database and a data warehouse.

---

# 9. For your current 50 applications

You don't need a huge architecture yet.

I would start with:

```text
                    DIM_DATE
                       |
                       |
DIM_CUSTOMER ---- FACT_LOAN_APPLICATION ---- DIM_STATUS
                       |
                       |
                  DIM_LOAN_TYPE
```

Then add:

```text
FACT_DOCUMENT_PROCESSING
FACT_APPROVAL
FACT_AI_REVIEW
```

as your AI implementation progresses.

---

# 10. The most important relationship design

For example:

```text
DIM_CUSTOMER
CustomerKey = 101
        |
        | 1
        |
        | *
        v
FACT_LOAN_APPLICATION

ApplicationKey
CustomerKey
LoanAmount
ProcessingDays
ApprovalFlag
```

So:

```text
DIM_CUSTOMER  1 ───────── * FACT_LOAN_APPLICATION
```

And:

```text
DIM_DATE 1 ───────── * FACT_LOAN_APPLICATION
```

And:

```text
DIM_STATUS 1 ───────── * FACT_LOAN_APPLICATION
```

This is exactly the relationship pattern you want. Microsoft recommends one-to-many relationships between dimensions and fact tables for robust star-schema reporting. ([Microsoft Learn][2])

Dataverse lookup relationships are well suited to implementing these parent/child relationships. ([Microsoft Learn][1])

---

## Bottom line

**Yes, you can create a star-schema-style database in Dataverse.** But for your Loan Application platform, I would **not make Dataverse your full data warehouse**.

I'd use:

**Dataverse = operational + AI application data**

**Power BI/Fabric = analytical star schema**

That gives you a much stronger architecture:

```text
               CUSTOMER
                  │
                  ▼
            POWER PAGES
                  │
                  ▼
              DATAVERSE
                  │
       ┌──────────┼───────────┐
       ▼          ▼           ▼
   POWER APPS  AI BUILDER  COPILOT
       │          │           │
       └──────────┼───────────┘
                  ▼
          POWER BI / FABRIC
                  │
                  ▼
            STAR SCHEMA
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
      FACT      FACT       FACT
     LOAN     APPROVAL      AI
   APPLICATION             REVIEW
        │         │         │
        └─────────┼─────────┘
                  │
                  ▼
             DIMENSIONS
                  │
                  ▼
          EXECUTIVE ANALYTICS
```

**For your project specifically, I would next redesign your existing `Loan Application` Dataverse table into a concrete Fact/Dimension model and show you exactly which columns should remain in `Loan Application`, which should move to `DimCustomer`, `DimLoanType`, `DimStatus`, etc., and how to create each relationship in Power Apps.**

[1]: https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-entity-lookup?utm_source=chatgpt.com "Create a relationship between tables by using a lookup column - Power Apps | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/power-bi/guidance/relationships-many-to-many?utm_source=chatgpt.com "Many-to-many relationship guidance - Power BI | Microsoft Learn"


The simplest way to think about it is:

> **OLTP is designed to run the business. A data warehouse is designed to analyze the business.**

For your **Loan Application Digitalization Platform**, Dataverse is naturally suited to the **OLTP side**, while Power BI/Fabric is better suited to the **data warehouse/analytics side**.

## 1. OLTP — Online Transaction Processing

An **OLTP database** stores the current operational transactions of an organization.

In your platform, examples are:

```text
Customer submits loan application
          ↓
Dataverse
          ↓
Loan Application record created
          ↓
Employee reviews application
          ↓
Approval record updated
          ↓
Customer receives status
```

The database needs to handle transactions such as:

* Create a loan application
* Update an application
* Upload a document
* Change application status
* Assign a reviewer
* Record an approval
* Update customer information

### Example

Your Dataverse table might contain:

| Application ID | Customer   | Loan Amount | Status       | Reviewer |
| -------------- | ---------- | ----------: | ------------ | -------- |
| APP-1001       | John Smith |    $120,000 | Approved     | Sarah    |
| APP-1002       | Mary Jones |    $175,000 | Under Review | David    |
| APP-1003       | Robert Lee |     $90,000 | Submitted    | Sarah    |

The objective is:

> **Make sure the current application information is accurate and available to the applications and workflows.**

---

# 2. Data Warehouse — OLAP

A **data warehouse** is designed primarily for **analysis rather than transaction processing**.

Instead of asking:

> "What is the status of APP-1002?"

you ask:

> "How has the approval rate changed over the last 12 months?"

or:

> "Which loan types have the longest processing times?"

or:

> "Are high-value applications increasing?"

The data warehouse brings together historical and analytical data so you can perform those types of analyses efficiently.

---

# 3. The difference in one picture

```text
                    OLTP
              "RUN THE BUSINESS"
                     |
                     v
              ┌─────────────┐
              │  Dataverse  │
              └──────┬──────┘
                     |
        ┌────────────┼────────────┐
        │            │            │
        v            v            v
   Power Apps    Power Pages   Power Automate
        │            │            │
        └────────────┼────────────┘
                     |
              CURRENT DATA
```

versus:

```text
                    OLAP
             "ANALYZE THE BUSINESS"
                     |
                     v
             ┌───────────────┐
             │ Power BI/Fabric│
             │ Data Warehouse │
             └───────┬───────┘
                     |
          ┌──────────┼──────────┐
          v          v          v
       Reports     KPIs       AI Insights
          │          │          │
          └──────────┼──────────┘
                     |
              HISTORICAL DATA
```

---

# 4. OLTP is optimized for transactions

OLTP databases typically have:

* lots of small transactions
* frequent inserts/updates/deletes
* current operational data
* strong consistency
* transactional integrity
* many users/processes simultaneously modifying data

For example:

```text
UPDATE LoanApplication
SET Status = 'Approved'
WHERE ApplicationID = 'APP-1002'
```

That's an OLTP operation.

The question is:

> **"What is the current state of this application?"**

---

# 5. Data warehouses are optimized for analysis

A warehouse typically has:

* large volumes of historical data
* analytical queries
* aggregations
* trend analysis
* dimensions and facts
* relatively fewer writes
* many reads
* reporting/BI workloads

For example:

```text
What was the average processing time
for approved loans by month and loan type
during the last three years?
```

That's an analytical/OLAP question.

---

# 6. Why star schema is common in warehouses

This is where your previous question about **star databases** comes in.

A typical warehouse model looks like:

```text
                     Dim Date
                        |
                        |
Dim Customer ---- Fact Loan Application ---- Dim Loan Type
                        |
                        |
                   Dim Status
                        |
                        |
                   Dim Reviewer
```

The **fact table** contains measurable business events.

For your project:

```text
Fact Loan Application

ApplicationKey
CustomerKey
DateKey
LoanTypeKey
StatusKey
LoanAmount
InterestRate
ProcessingDays
ApprovedFlag
HighValueFlag
```

The **dimension tables** describe those events:

```text
Dim Customer

CustomerKey
CustomerName
Province
EmploymentType
IncomeRange
```

```text
Dim Loan Type

LoanTypeKey
LoanType
Description
```

```text
Dim Date

DateKey
Date
Month
Quarter
Year
```

This makes Power BI analysis much easier.

---

# 7. Why you shouldn't necessarily use the same table structure for both

Consider your current `Loan Application` table.

For an application system, you might need:

```text
Application ID
Customer
Address
Loan Amount
Income
Status
Reviewer
Documents
Approval
```

But imagine you want to analyze **three years of application history**.

You might want to know:

```text
2024:
Applications = 3,200
Approval Rate = 62%
Average Processing = 7.2 days

2025:
Applications = 4,100
Approval Rate = 66%
Average Processing = 6.1 days

2026:
Applications = 5,300
Approval Rate = 69%
Average Processing = 5.4 days
```

If your operational system only stores the **current state** of each application, some historical information can be lost.

A warehouse is designed to preserve that analytical history.

---

# 8. Another important difference: normalization

OLTP systems often favor **normalization**.

For example:

```text
Customer
   |
   +---- Customer Address
   |
   +---- Loan Application
             |
             +---- Documents
             |
             +---- Approval
```

You avoid unnecessarily duplicating information.

This helps maintain data integrity.

For example, if a customer's address changes, you update it in one place.

---

A data warehouse often uses a **star schema**.

You intentionally structure data for analytical queries:

```text
                    Dim Customer
                         |
                         |
Dim Date ---- Fact Loan Application ---- Dim Loan Type
                         |
                         |
                    Dim Status
```

The goal isn't primarily minimizing duplication.

The goal is:

> **fast, understandable analytical queries.**

---

# 9. Your Loan Platform is actually a perfect example

Your architecture can be:

```text
                  CUSTOMER
                     |
                     v
                POWER PAGES
                     |
                     v
                 DATAVERSE
                  (OLTP)
                     |
        +------------+------------+
        |            |            |
        v            v            v
   Power Apps   Power Automate  AI Builder
        |            |            |
        +------------+------------+
                     |
                     v
                Data Pipeline
                     |
                     v
              POWER BI / FABRIC
                  (OLAP)
                     |
              STAR SCHEMA
                     |
        +------------+------------+
        |            |            |
        v            v            v
      Facts      Dimensions    Metrics
        |            |            |
        +------------+------------+
                     |
                     v
              EXECUTIVE REPORTING
                     |
                     v
                 AI INSIGHTS
```

That's a very strong architecture for your project.

---

# 10. Think about it this way

Imagine a bank branch.

### OLTP

The teller says:

> "What is the customer's current account balance?"

The operational database answers that.

### Data warehouse

The manager asks:

> "How have customer deposits changed by region over the last five years?"

The data warehouse answers that.

### AI layer

The executive asks:

> "Why did deposits decline in Ontario this quarter?"

The AI layer analyzes the governed analytical data and explains the trend.

---

# 11. Applying it directly to your platform

### Dataverse / OLTP

Handles:

```text
Customer
Loan Application
Loan Document
Application Review
Approval
Notification
AI Extraction Result
AI Validation Finding
```

These are operational records.

### Power BI/Fabric / OLAP

Handles:

```text
Fact Loan Application
Fact Approval
Fact Document Processing
Fact AI Review

Dim Date
Dim Customer
Dim Loan Type
Dim Status
Dim Reviewer
Dim Document
```

These are analytical structures.

---

# 12. What happens when an employee asks Copilot:

> "Which applications have been waiting the longest?"

The architecture could be:

```text
Employee
   |
   v
Employee Copilot
   |
   v
Governed analytical query
   |
   v
Power BI Semantic Model
   |
   v
Fact Loan Application
   |
   +---- Dim Date
   +---- Dim Status
   +---- Dim Customer
   |
   v
Result
   |
   v
Copilot
   |
   v
Natural-language answer
```

For:

> "What is the current status of APP-1002?"

you would normally go directly to the operational system:

```text
Employee
   |
   v
Copilot
   |
   v
Dataverse
   |
   v
APP-1002
```

That's the fundamental distinction.

---

# 13. One subtle point: Power BI doesn't necessarily require a physical warehouse

This is important for your current project.

You **do not need to immediately build a separate physical data warehouse** just because you're using a star schema.

Power BI can create a star-schema-style semantic model from your Dataverse data.

For example:

```text
Dataverse
   |
   v
Power BI
   |
   +---- Dim Customer
   +---- Dim Date
   +---- Dim Status
   |
   +---- Fact Loan Application
```

For your current **50-application demo**, that is perfectly reasonable.

As the platform grows, you can introduce Microsoft Fabric components such as a Lakehouse/Warehouse and build a more formal enterprise analytical architecture.

---

# 14. What I recommend for your project right now

Given where your Loan Application Digitalization Platform currently is, I would **not move everything into a warehouse yet**.

Use this:

```text
                  POWER PLATFORM
                       |
                       v
                   DATAVERSE
                    OLTP
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
    Power Apps     Power Pages    Power Automate
        |              |              |
        +--------------+--------------+
                       |
                       v
                  AI BUILDER
                       |
                       v
                 AI RESULTS
                       |
                       v
                 POWER BI
                       |
                       v
              STAR SCHEMA MODEL
                       |
                       v
               SEMANTIC MODEL
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
    Dashboard     AI Insights    Conversational BI
```

Later:

```text
Dataverse
    |
    v
Fabric Data Engineering
    |
    v
Lakehouse / Warehouse
    |
    v
Star Schema
    |
    v
Power BI Semantic Model
    |
    v
Executive AI
```

### In one sentence

**Dataverse/OLTP answers "What is happening now?" while a warehouse/OLAP answers "What has happened, why, and what patterns do we see?"**

For your Loan Application project, this distinction is especially valuable because it lets you position the solution as more than a Power Apps demo: **Dataverse runs the lending operation, while the Power BI/Fabric analytical layer turns those operations into governed business intelligence and AI-assisted decision support.**


Yes. Given the **full scope you have now defined**, I would revise my earlier recommendation.

You are no longer just adding a chatbot to an MVP. You are building an **AI-enabled lending platform** with operational processing, document intelligence, authenticated experiences, governed analytics, and conversational AI. In that situation, adding a proper **Fabric analytical layer now** is a good architectural decision.

The key is **not to replace Dataverse with the warehouse**. Instead, keep Dataverse as the operational system and make Fabric the analytical/AI data platform.

## 1. Your target architecture

I would now recommend this:

```text
                         EIPPONE LOAN PLATFORM
                                  |
              +-------------------+-------------------+
              |                   |                   |
              v                   v                   v
        CUSTOMER CHANNEL     EMPLOYEE CHANNEL    EXECUTIVE CHANNEL
              |                   |                   |
        Power Pages          Model-Driven App     Power BI
              |                   |                   |
        Customer Agent       Employee Agent      Analytics Agent
              |                   |                   |
              +-------------------+-------------------+
                                  |
                                  v
                         MICROSOFT COPILOT STUDIO
                                  |
                  +---------------+---------------+
                  |               |               |
                  v               v               v
             DATAVERSE       AI BUILDER       POWER BI/FABRIC
             Operational      Documents        Analytical
                OLTP           AI Layer           OLAP
                  |               |               |
                  |               v               |
                  |        AI Extraction          |
                  |        AI Summary             |
                  |        AI Findings            |
                  |               |               |
                  +---------------+---------------+
                                  |
                                  v
                         FABRIC DATA PLATFORM
                                  |
                 +----------------+----------------+
                 |                                 |
                 v                                 v
             LAKEHOUSE                         WAREHOUSE
                 |                                 |
                 +----------------+----------------+
                                  |
                                  v
                         STAR SCHEMA / SEMANTIC
                              MODEL
                                  |
                                  v
                              POWER BI
                                  |
                                  v
                       EXECUTIVE / ANALYTICAL AI
```

And authentication sits across the customer and employee experiences:

```text
Customer
   |
   v
Microsoft Entra External ID / Power Pages Authentication
   |
   v
Customer's authorized Dataverse records


Employee / Approver
   |
   v
Microsoft Entra ID
   |
   v
Model-Driven App / Power BI
   |
   v
Authorized organizational data
```

So yes: **I think implementing Fabric now is reasonable for your project.**

---

# 2. But there is one critical distinction

You asked:

> How will we convert the current Loan Application Dataverse table into the warehouse/OLAP?

I would **not convert it in place**.

Instead:

```text
              CURRENT
             DATAVERSE
                 |
                 | Data movement
                 v
              FABRIC
                 |
        +--------+--------+
        |                 |
        v                 v
    Lakehouse          Warehouse
                          |
                          v
                    Star Schema
                          |
                          v
                    Power BI Model
```

So you will have:

### Dataverse

Your **system of record / OLTP**.

### Fabric

Your **analytical platform / OLAP**.

They serve different purposes.

---

# 3. Do we keep both?

## Yes — absolutely.

This is the architecture I recommend.

You will have:

```text
DATAVERSE
=========
Operational truth
```

and:

```text
FABRIC
======
Analytical truth
```

More precisely:

### Dataverse contains the current operational state.

For example:

```text
APP-1002

Customer:
John Smith

Loan:
$175,000

Status:
Under Review

Reviewer:
Jane Smith
```

### Fabric contains analytical history.

For example:

```text
Application
Date        Status          Amount
---------------------------------------
2026-01-05  Submitted       120,000
2026-01-06  Under Review   120,000
2026-01-09  Approved       120,000
```

This becomes extremely important when you want to answer:

> How long did applications spend in each status?

or:

> How has processing time changed over the last six months?

The operational table alone isn't necessarily the best structure for those questions.

---

# 4. Don't make the Fabric warehouse a copy of the Loan Application table

This is the biggest architectural point.

You currently have something like:

```text
Loan Application
-----------------------------
Application ID
Customer Name
Address
Loan Amount
Loan Term
Interest Rate
Income
Status
Reviewer
Submission Date
...
```

Instead, Fabric should transform this into a dimensional model.

For example:

```text
                    DIM_CUSTOMER
                         |
                         |
                         v
DIM_DATE ------ FACT_LOAN_APPLICATION ------ DIM_LOAN
                         |
                         |
                    DIM_STATUS
                         |
                         |
                    DIM_REVIEWER
```

---

# 5. Your Fact Loan Application

In Fabric:

```text
FACT_LOAN_APPLICATION
```

might contain:

| Column            | Example  |
| ----------------- | -------- |
| ApplicationKey    | 1002     |
| ApplicationID     | APP-1002 |
| CustomerKey       | 501      |
| LoanProductKey    | 3        |
| StatusKey         | 2        |
| SubmissionDateKey | 20260811 |
| DecisionDateKey   | NULL     |
| LoanAmount        | 175000   |
| LoanTerm          | 60       |
| InterestRate      | 6.5      |
| AnnualIncome      | 92000    |
| ProcessingDays    | NULL     |
| HighValueFlag     | 1        |
| ApprovedFlag      | 0        |
| RejectedFlag      | 0        |

Notice something important:

**Customer Name isn't necessarily stored repeatedly in the fact table.**

You get it through:

```text
CustomerKey
      |
      v
DIM_CUSTOMER
```

---

# 6. DIM_CUSTOMER

```text
DIM_CUSTOMER
------------
CustomerKey
CustomerID
CustomerName
Province
EmploymentType
IncomeRange
CustomerSegment
```

---

# 7. DIM_LOAN

```text
DIM_LOAN
--------
LoanProductKey
LoanType
LoanProduct
Description
MinAmount
MaxAmount
```

---

# 8. DIM_STATUS

```text
DIM_APPLICATION_STATUS
----------------------
StatusKey
StatusCode
StatusName
StatusCategory
```

For example:

```text
1 = Submitted
2 = Under Review
3 = Approved
4 = Rejected
5 = More Information Required
```

---

# 9. DIM_DATE

This is extremely important for Power BI.

```text
DIM_DATE
--------
DateKey
Date
Day
Month
MonthName
Quarter
Year
FiscalYear
Week
```

Then Power BI can easily answer:

> How many applications were submitted each month?

or:

> How did approval rates change by quarter?

---

# 10. But your AI requirements introduce more fact tables

This is where your project becomes much more interesting.

I would **not create only `FACT_LOAN_APPLICATION`**.

I would create several facts.

## Fact 1 — Loan Application

```text
FACT_LOAN_APPLICATION
```

One row per application.

---

## Fact 2 — Application Status History

```text
FACT_APPLICATION_STATUS
```

One row every time an application's status changes.

For example:

```text
APP-1002
Submitted
2026-08-01

APP-1002
Under Review
2026-08-02

APP-1002
More Information Required
2026-08-04

APP-1002
Under Review
2026-08-06

APP-1002
Approved
2026-08-09
```

This enables:

> How long does an application stay under review?

That is much better than simply storing the current status.

---

# 11. Fact 3 — Document Processing

For your AI Builder implementation:

```text
FACT_DOCUMENT_PROCESSING
```

One row per document processed.

Example:

```text
Application   Document       Confidence
-----------------------------------------
APP-1002      ID             98%
APP-1002      Income         74%
APP-1002      Tax Return     91%
APP-1002      Application    96%
```

Now you can create an executive KPI:

> **Average AI document extraction confidence = 89.7%**

---

# 12. Fact 4 — AI Review

For your AI application summary:

```text
FACT_AI_REVIEW
```

Example:

| Application | Confidence | Findings | Recommendation         | Human Override |
| ----------- | ---------: | -------: | ---------------------- | -------------- |
| APP-1002    |        89% |        1 | Review                 | Yes            |
| APP-1003    |        95% |        0 | Documentation Complete | No             |

This allows you to analyze the AI system itself.

For example:

> How often do reviewers override AI recommendations?

That is an excellent governance metric.

---

# 13. Fact 5 — Approval

You already have Power Automate approval.

Create:

```text
FACT_APPROVAL
```

Example:

```text
Application
Reviewer
Submission Date
Decision Date
Decision
Approval Duration
```

Then you can calculate:

```text
Average approval time
Approval rate
Rejection rate
Approval rate by reviewer
Approval rate by loan type
```

---

# 14. Your complete Fabric model

I would therefore target:

```text
                         DIM_DATE
                            |
                            |
                   +--------+--------+
                   |                 |
                   v                 v
           FACT_LOAN_APPLICATION   FACT_APPROVAL
                   |
                   |
                   v
             DIM_CUSTOMER
                   |
                   |
                   v
              DIM_LOAN
                   |
                   |
                   v
              DIM_STATUS


             FACT_DOCUMENT_PROCESSING
                       |
                       +---- DIM_DOCUMENT
                       |
                       +---- DIM_DATE
                       |
                       +---- DIM_APPLICATION


                  FACT_AI_REVIEW
                       |
                       +---- DIM_APPLICATION
                       +---- DIM_AI_MODEL
                       +---- DIM_DATE


              FACT_APPLICATION_STATUS
                       |
                       +---- DIM_STATUS
                       +---- DIM_DATE
                       +---- DIM_APPLICATION
```

This is much more powerful than simply copying your Dataverse table.

---

# 15. How do we actually move the Dataverse data into Fabric?

This is the next implementation step.

Conceptually:

```text
Dataverse
    |
    v
Fabric ingestion
    |
    v
Bronze/raw data
    |
    v
Transformation
    |
    v
Silver/curated data
    |
    v
Gold/star schema
    |
    v
Power BI semantic model
```

If you are using Microsoft Fabric, you can use Fabric's Data Factory capabilities, pipelines, Dataflows Gen2, shortcuts, notebooks, or other ingestion patterns depending on your environment and scale.

For your project, I would start simple rather than build an unnecessarily complicated medallion architecture.

---

# 16. I would use three conceptual layers

### Layer 1 — Raw

```text
Dataverse
     |
     v
Fabric Raw
```

Keep the source data close to its original form.

---

### Layer 2 — Curated

Clean:

```text
Names
Dates
Statuses
Data types
Duplicates
Missing values
```

---

### Layer 3 — Analytical

Create:

```text
FACT_LOAN_APPLICATION
FACT_APPROVAL
FACT_DOCUMENT_PROCESSING
FACT_AI_REVIEW
FACT_APPLICATION_STATUS

DIM_CUSTOMER
DIM_DATE
DIM_LOAN
DIM_STATUS
DIM_DOCUMENT
DIM_REVIEWER
```

Then:

```text
                 GOLD
                  |
                  v
             Power BI
```

---

# 17. What happens when a customer submits an application?

This is very important for question #4.

You asked:

> Can the customer application intake form record the data directly in the warehouse/OLAP?

### Technically: you could build something that writes there.

### Architecturally: **I strongly recommend NO.**

Your customer should submit to:

```text
Power Pages
    |
    v
Dataverse
```

not:

```text
Power Pages
    |
    v
Fabric Warehouse
```

Why?

Because the application is an **operational transaction**.

You need:

* transaction integrity
* immediate validation
* record-level security
* relationships
* workflow triggers
* Power Automate
* Power Apps
* Power Pages
* approvals
* document relationships

Dataverse is designed for that operational workload.

---

# 18. The correct flow

The customer experience should be:

```text
Customer
   |
   v
Power Pages
   |
   v
Authenticated
   |
   v
Dataverse
   |
   +----------------------+
   |                      |
   v                      v
Power Automate          AI Builder
   |                      |
   |                      v
   |                 AI Results
   |                      |
   +----------+-----------+
              |
              v
           Dataverse
              |
              v
        Fabric ingestion
              |
              v
        Fabric Warehouse
              |
              v
        Power BI Semantic
             Model
```

That gives you the best of both worlds.

---

# 19. What about real-time data?

You might be thinking:

> "But if Fabric receives the data later, how can my AI agent answer questions about the latest application?"

This is why we shouldn't force every use case through the warehouse.

Use the appropriate source for the question.

### Customer application status

Use:

```text
Copilot
   ↓
Dataverse
```

because it needs current transactional information.

### Employee application review

Use:

```text
Employee Copilot
       ↓
Dataverse
       ↓
Application + Documents + AI Results
```

because the reviewer needs the current application.

### Executive KPI

Use:

```text
Executive Copilot
       ↓
Power BI Semantic Model
       ↓
Fabric
```

because the executive needs governed analytics.

---

# 20. And the AI application summary?

This is another area where **Dataverse should remain central**.

Suppose APP-1002 has:

```text
Application Form
+
ID
+
Income Statement
+
Tax Return
+
Bank Statement
```

The pipeline should be:

```text
                  APP-1002
                     |
          +----------+----------+
          |          |          |
          v          v          v
      Form PDF      ID      Income Doc
          |          |          |
          +----------+----------+
                     |
                     v
                 AI Builder
                     |
                     v
              Extracted Data
                     |
                     v
             Validation Layer
                     |
                     v
               AI Summary
                     |
                     v
                 Dataverse
                     |
                     v
             Employee Review App
```

The reviewer gets the summary **from Dataverse**, not from the warehouse.

Then the analytical representation of that AI processing goes into Fabric:

```text
Dataverse
     |
     v
FACT_AI_REVIEW
     |
     v
Power BI
```

---

# 21. This gives you a very clean separation

## Operational AI

```text
Copilot
  |
  v
Dataverse
  |
  +-- Application
  +-- Documents
  +-- AI Extraction
  +-- AI Findings
  +-- AI Summary
```

## Analytical AI

```text
Copilot
  |
  v
Power BI Semantic Model
  |
  v
Fabric
  |
  +-- Application trends
  +-- Approval trends
  +-- Processing times
  +-- High-value loans
  +-- AI quality
  +-- Document quality
```

---

# 22. Authentication should also be separated

You are correct that authentication is an important part of the architecture.

### Customer

```text
Customer
   ↓
Power Pages authentication
   ↓
Customer identity
   ↓
Dataverse permissions
   ↓
Own application
```

### Employee

```text
Employee
   ↓
Microsoft Entra ID
   ↓
Power Apps / Model-Driven App
   ↓
Dataverse security roles
   ↓
Authorized applications
```

### Power BI

```text
Employee
   ↓
Microsoft Entra ID
   ↓
Power BI
   ↓
Workspace permissions
   ↓
Semantic model
   ↓
RLS where required
```

The conversational agent should **not become a security bypass**.

---

# 23. Customer AI security

For example, customer John asks:

> What is my application status?

The agent should identify John through the authenticated session.

Then:

```text
Authenticated Customer
        |
        v
Customer ID
        |
        v
Dataverse
        |
        v
Applications owned by Customer
```

It should **not** do:

```text
"What application number did the customer type?"
                     |
                     v
              Search Dataverse
```

because a malicious customer could potentially type someone else's application number.

---

# 24. Employee AI security

Similarly:

```text
Employee
   |
   v
Employee Copilot
   |
   v
Authorized tool
   |
   v
Dataverse / Power BI
```

The tool should enforce authorization.

For example:

> "Show me all applications over $150,000."

The underlying tool determines which applications the employee is authorized to see.

---

# 25. Your Power BI reports should use the Fabric model

This is another change I recommend.

Instead of:

```text
Power BI
   |
   v
Direct Dataverse connection
```

your final architecture should become:

```text
Power BI
   |
   v
Power BI Semantic Model
   |
   v
Fabric Warehouse
   |
   v
Dataverse
```

This will give you a much stronger foundation for your **conversational analytics** and executive AI.

---

# 26. Your current Power BI dashboard doesn't need to disappear

You can migrate it.

Currently:

```text
Dataverse
    ↓
Power BI
    ↓
Executive_Loan_Dashboard
```

Eventually:

```text
Dataverse
    ↓
Fabric
    ↓
Star Schema
    ↓
Power BI Semantic Model
    ↓
Executive_Loan_Dashboard
```

Your existing visualizations can then be repointed/rebuilt against the new semantic model.

---

# 27. What I would NOT do

I would avoid this architecture:

```text
Customer
   |
   v
Power Pages
   |
   v
Fabric Warehouse
   |
   +-- Power BI
   +-- Copilot
```

It makes your analytical system responsible for operational transactions.

That's mixing OLTP and OLAP responsibilities.

---

# 28. What I WOULD do

Your final EIPPONE architecture becomes:

```text
                         USERS
                           |
          +----------------+----------------+
          |                |                |
       CUSTOMER         EMPLOYEE        EXECUTIVE
          |                |                |
          v                v                v
    Power Pages       Power Apps        Power BI
          |                |                |
          v                v                |
     Customer AI       Employee AI          |
          |                |                |
          +--------+-------+----------------+
                   |
                   v
              COPILOT STUDIO
                   |
          +--------+--------+
          |                 |
          v                 v
      DATAVERSE          POWER BI
       / OLTP             / OLAP
          |                 |
          |                 v
          |              FABRIC
          |                 |
          |          +------+------+
          |          |             |
          |          v             v
          |      Lakehouse      Warehouse
          |                        |
          |                        v
          |                  Star Schema
          |                        |
          |                        v
          |                 Semantic Model
          |                        |
          +------------------------+
                                   |
                                   v
                            ANALYTICAL AI
                                   |
                         +---------+---------+
                         |                   |
                    Report Insights    Conversation
```

---

# 29. So, answering your four questions directly

### **1. Should you implement Fabric/Power BI OLAP now?**

**Yes.**

Given that you're now implementing:

* AI Builder
* customer conversational AI
* employee/approver conversational AI
* application summarization
* document intelligence
* executive AI insights
* conversational Power BI
* authenticated customer access
* authenticated employee access
* governed reporting

I think **now is the right time to establish the Fabric analytical architecture**.

You don't need to wait until the application has thousands of records.

The important benefit is getting the **architecture right before the AI layer becomes deeply dependent on the existing Power BI → Dataverse structure**.

---

### **2. How do we convert the current Dataverse table?**

**Don't convert it.**

Instead:

```text
Current Dataverse Loan Application
              |
              v
       Fabric ingestion
              |
              v
        Raw / staging
              |
              v
        Transformation
              |
              v
       Star Schema
```

Your current table becomes the **source**, not the warehouse.

---

### **3. Do we keep both?**

**Yes.**

You should have:

```text
Dataverse
= Operational System of Record
```

and:

```text
Fabric Warehouse
= Analytical System
```

Then:

```text
Power Apps / Power Pages / AI Builder / transactional Copilot
                         ↓
                     Dataverse

Power BI / Executive AI / Analytical Copilot
                         ↓
                  Fabric / Semantic Model
```

---

### **4. Can the customer intake form write directly to the warehouse?**

**Technically possible in some architectures, but I would not do it.**

Your customer intake form should write to:

```text
Power Pages
      ↓
Dataverse
```

Then:

```text
Dataverse
      ↓
Fabric
      ↓
Warehouse
      ↓
Power BI
```

That is the architecture I would defend in an enterprise architecture review.

---

# 30. And this changes how I would approach your next implementation step

I would **pause before building the Copilots** and first establish the **data foundation**.

Specifically, I would do these things in this order:

```text
STEP 1
Current Dataverse model
        ↓
STEP 2
Redesign operational Dataverse relationships
        ↓
STEP 3
Create Fabric workspace
        ↓
STEP 4
Connect Dataverse → Fabric
        ↓
STEP 5
Create raw/staging layer
        ↓
STEP 6
Create analytical star schema
        ↓
STEP 7
Create Power BI semantic model
        ↓
STEP 8
Migrate Executive_Loan_Dashboard
        ↓
STEP 9
Implement AI Builder document processing
        ↓
STEP 10
Implement application AI summary
        ↓
STEP 11
Implement authenticated Customer Copilot
        ↓
STEP 12
Implement authenticated Employee Copilot
        ↓
STEP 13
Implement Analytical/Executive Copilot
        ↓
STEP 14
Implement automated executive insights
        ↓
STEP 15
Implement end-to-end security testing
```

**That is the architecture I would use for your Loan Application Digitalization Platform now.**

And importantly, your existing **50-record Dataverse `Loan Application` table does not need to be thrown away or rebuilt from scratch**. We can use those 50 records as the initial source data and build the Fabric model around them. Then every new application submitted through Power Pages will continue entering Dataverse first and subsequently flow into Fabric for analytics.

The next concrete task should therefore be to take **your actual current `Loan Application` Dataverse columns** and design the exact **Dataverse operational model → Fabric staging → Fact/Dimension tables → Power BI semantic model**. That will give us a precise blueprint before we touch AI Builder or Copilot Studio.


Yes. Let's implement **Step 2 properly before touching Fabric**.

Because your current MVP was built around a single **Loan Application** Dataverse table, I recommend we refactor the operational model rather than trying to make that one table serve customers, documents, reviews, approvals, AI processing, and analytics simultaneously.

The target is:

```text
                         DATAVERSE — OPERATIONAL MODEL
                                      |
          +---------------------------+---------------------------+
          |                           |                           |
          v                           v                           v
      CUSTOMER                 LOAN APPLICATION              DOCUMENTS
          |                           |                           |
          |                           |                           |
          +---------------------------+---------------------------+
                                      |
                         +------------+------------+
                         |            |            |
                         v            v            v
                     REVIEWS      APPROVALS    STATUS HISTORY
                         |
                         v
                    AI PROCESSING
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
      EXTRACTION     VALIDATION     RECOMMENDATION
```

The important thing is that **Dataverse remains operational**. Fabric will later consume this clean structure.

---

# Step 2 — Redesign the Operational Dataverse Model

## 1. The target Dataverse tables

I recommend creating the following tables:

### Core operational tables

```text
1. Customer
2. Loan Application
3. Loan Document
4. Application Review
5. Approval
6. Application Status History
```

### AI tables

```text
7. AI Extraction Result
8. AI Validation Finding
9. AI Recommendation
10. AI Processing Log
```

### Configuration

```text
11. AI Configuration
```

You don't need to build all 11 before continuing. We should build the **six core operational tables first**, then the AI tables.

---

# 2. Most important change: separate Customer from Loan Application

Your current table likely contains customer information alongside application information.

Conceptually, you may currently have:

```text
Loan Application
-------------------------
Application ID
Customer Name
Address
Phone
Email
Loan Amount
Loan Term
Interest Rate
Income
Employment
Status
Reviewer
...
```

We want to change this to:

```text
CUSTOMER
----------------
Customer ID
First Name
Last Name
Email
Phone
Address
City
Province
Postal Code
Employment
Annual Income
```

and:

```text
LOAN APPLICATION
-------------------------
Application ID
Customer
Loan Amount
Loan Term
Interest Rate
Status
Submission Date
...
```

The relationship becomes:

```text
Customer
   1
   |
   | has
   |
   *
Loan Application
```

This means one customer can eventually have multiple loan applications.

That is important for your future conversational AI.

For example:

> "Show me my previous loan applications."

The system can retrieve all applications associated with the authenticated customer.

---

# 3. Before changing anything — make a backup

Since your existing Canvas App, Model-Driven App and Power Automate flow already depend on the current table, **do not delete or rename columns yet**.

First:

**Power Apps → Solutions**

Open your existing solution or create:

```text
EIPPONE Loan Platform
```

Add:

```text
Loan Application
```

and all dependent components.

Also export an unmanaged solution as a backup if this is your development environment.

---

# 4. Create the Customer table

Go to:

**Power Apps → Tables → New table**

Create:

```text
Display name:
Customer

Plural:
Customers

Primary column:
Customer Name
```

For your project, I'd use a **Customer Number** as the business identifier.

Add:

| Column          | Type       |
| --------------- | ---------- |
| Customer Number | Autonumber |
| First Name      | Text       |
| Last Name       | Text       |
| Email           | Email      |
| Phone           | Phone      |
| Address         | Text       |
| City            | Text       |
| Province        | Choice     |
| Postal Code     | Text       |
| Employment Type | Choice     |
| Annual Income   | Currency   |

### Important

Do not make `Customer Number` the same thing as the Dataverse GUID.

Dataverse will automatically generate its internal unique identifier.

You want something human-readable such as:

```text
CUS-00001
CUS-00002
CUS-00003
```

Your Dataverse GUID remains the technical key.

---

# 5. Modify Loan Application

Open your existing:

**Loan Application**

Do **not** delete the existing customer fields yet.

First add a new lookup:

```text
Customer
```

Relationship:

```text
Customer 1 : N Loan Application
```

In the relationship designer, the structure should be:

```text
Customer
   |
   | 1
   |
   | N
   |
Loan Application
```

Set the lookup column on `Loan Application`:

```text
Customer
```

---

# 6. What should remain in Loan Application?

The Loan Application table should contain **application-specific information**.

For example:

```text
Loan Application
-----------------------------
Application ID
Customer
Loan Type
Loan Amount
Loan Term
Interest Rate
Application Status
Submission Date
Decision Date
High Value Loan
High Value Threshold
Assigned Reviewer
```

Customer information should gradually move to Customer.

So:

### Customer

```text
Name
Address
Phone
Email
Employment
Income
```

### Application

```text
Loan Amount
Loan Term
Interest Rate
Status
Submission Date
Reviewer
Decision
```

---

# 7. What about Annual Income?

This one requires a design decision.

For your lending platform, I recommend **keeping current customer profile income in Customer**, but also storing the **income declared for a particular application** in Loan Application.

Why?

Suppose:

```text
Customer income today:
$90,000
```

but when they submitted Application A:

```text
Declared income:
$82,000
```

and later Application B:

```text
Declared income:
$95,000
```

You don't want changing the Customer record to overwrite historical application information.

Therefore:

### Customer

```text
Current Annual Income
```

### Loan Application

```text
Declared Annual Income
```

That is an important historical distinction.

---

# 8. Create Loan Type

If your current Loan Application table has something like:

```text
Loan Type
```

I recommend creating a separate table:

```text
Loan Type
```

Example:

| Loan Type ID | Loan Type     |
| ------------ | ------------- |
| LT001        | Personal Loan |
| LT002        | Business Loan |
| LT003        | Mortgage      |
| LT004        | Auto Loan     |

Then create:

```text
Loan Type 1:N Loan Application
```

Your application contains:

```text
Loan Type → Lookup
```

instead of free text.

This will later become:

```text
DIM_LOAN_TYPE
```

in Fabric.

---

# 9. Create Application Status as a Choice

For the operational application, I recommend using a **Choice** column:

```text
Application Status
```

Values:

```text
Draft
Submitted
Under Review
More Information Required
Approved
Rejected
Withdrawn
Completed
```

However, there is an important distinction:

### Current Status

belongs on:

```text
Loan Application
```

### Status History

belongs in a separate table.

---

# 10. Create Application Status History

Create:

```text
Application Status History
```

Columns:

| Column            | Type           |
| ----------------- | -------------- |
| Status History ID | Autonumber     |
| Loan Application  | Lookup         |
| Previous Status   | Choice         |
| New Status        | Choice         |
| Status Date       | Date/Time      |
| Changed By        | User           |
| Comments          | Multiline Text |

Relationship:

```text
Loan Application
       1
       |
       | N
       |
Application Status History
```

This is extremely valuable.

Suppose APP-1002 goes:

```text
Submitted
    ↓
Under Review
    ↓
More Information Required
    ↓
Under Review
    ↓
Approved
```

You preserve every transition.

Later Fabric can calculate:

> Average time spent under review.

---

# 11. Create Loan Document

Now we address your AI Builder requirement.

Create:

```text
Loan Document
```

Columns:

| Column               | Type       |
| -------------------- | ---------- |
| Document ID          | Autonumber |
| Loan Application     | Lookup     |
| Document Type        | Choice     |
| Document Name        | Text       |
| Document Status      | Choice     |
| File                 | File       |
| Upload Date          | Date/Time  |
| AI Processing Status | Choice     |
| AI Confidence        | Decimal    |
| Review Required      | Yes/No     |

Document Type:

```text
Application Form
Identification
Proof of Income
Tax Return
Bank Statement
Supporting Document
Other
```

Relationship:

```text
Loan Application
       1
       |
       | N
       |
Loan Document
```

This is essential because one application can have many documents.

---

# 12. Your document architecture becomes

```text
Loan Application
        |
        | 1
        |
        | N
        |
   Loan Document
        |
        +---- Application PDF
        +---- ID
        +---- Income
        +---- Tax Return
        +---- Bank Statement
```

This is exactly what your AI Builder pipeline needs.

---

# 13. Create Application Review

Now separate the **review activity** from the application itself.

Create:

```text
Application Review
```

Columns:

| Column                | Type           |
| --------------------- | -------------- |
| Review ID             | Autonumber     |
| Loan Application      | Lookup         |
| Reviewer              | User           |
| Review Status         | Choice         |
| Review Start Date     | Date/Time      |
| Review Completed Date | Date/Time      |
| Reviewer Comments     | Multiline Text |
| Review Outcome        | Choice         |

Review Status:

```text
Not Started
In Progress
Completed
Escalated
```

Relationship:

```text
Loan Application
       1
       |
       | N
       |
Application Review
```

This gives you flexibility if an application goes through multiple reviews.

---

# 14. Create Approval

Your existing Power Automate approval process should eventually write to:

```text
Approval
```

Create:

| Column           | Type           |
| ---------------- | -------------- |
| Approval ID      | Autonumber     |
| Loan Application | Lookup         |
| Approver         | User           |
| Approval Status  | Choice         |
| Submitted Date   | Date/Time      |
| Decision Date    | Date/Time      |
| Comments         | Multiline Text |

Relationship:

```text
Loan Application
       1
       |
       | N
       |
Approval
```

This gives you a proper audit trail.

---

# 15. The core operational model now looks like this

```text
                         CUSTOMER
                            |
                            | 1
                            |
                            | N
                    LOAN APPLICATION
                     /      |       \
                    /       |        \
                   N        N         N
                  /         |          \
                 v          v           v
       LOAN DOCUMENT   APPLICATION    APPROVAL
                         REVIEW
                            |
                            |
                            v
                   STATUS HISTORY
```

More accurately:

```text
Customer
   1
   |
   N
Loan Application
   |
   +------ N Loan Document
   |
   +------ N Application Review
   |
   +------ N Approval
   |
   +------ N Status History
```

That is your **operational backbone**.

---

# 16. Now add the AI layer

Once those tables are established, create:

```text
AI Extraction Result
```

Relationship:

```text
Loan Document
       1
       |
       N
AI Extraction Result
```

For example:

```text
Loan Document
     |
     v
AI Extraction Result
     |
     +-- Customer Name
     +-- Address
     +-- Loan Amount
     +-- Income
     +-- Interest Rate
     +-- Confidence
```

---

# 17. AI Validation Finding

Create:

```text
AI Validation Finding
```

Relationship:

```text
Loan Application
       1
       |
       N
AI Validation Finding
```

Example:

```text
APP-1002

Finding:
Income mismatch

Application:
$90,000

Document:
$82,000

Severity:
High

Confidence:
94%
```

---

# 18. AI Recommendation

Create:

```text
AI Recommendation
```

Relationship:

```text
Loan Application
       1
       |
       N
AI Recommendation
```

I'd actually allow multiple recommendations because AI processing may happen more than once.

For example:

```text
AI Recommendation #1
Model Version: v1
Recommendation: Review Required

AI Recommendation #2
Model Version: v2
Recommendation: Documentation Complete
```

Then you have an auditable AI history.

---

# 19. AI Processing Log

Create:

```text
AI Processing Log
```

This will be useful later for governance.

Columns:

```text
Processing ID
Application
Document
AI Service
Model Version
Processing Start
Processing End
Status
Error Message
Confidence
```

This gives you the ability to answer:

> Why did this application fail AI processing?

---

# 20. Your final operational Dataverse model

I recommend getting to this:

```text
                         CUSTOMER
                            |
                            | 1:N
                            v
                    LOAN APPLICATION
                     /      |       \
                    /       |        \
                  1:N      1:N       1:N
                  /         |          \
                 v          v           v
          LOAN DOCUMENT  REVIEW      APPROVAL
                 |
                 | 1:N
                 v
        AI EXTRACTION RESULT
                 |
                 |
        +--------+--------+
        |                 |
        v                 v
 AI VALIDATION       AI PROCESSING
    FINDING               LOG

LOAN APPLICATION
       |
       +---- 1:N STATUS HISTORY
       |
       +---- 1:N AI VALIDATION FINDING
       |
       +---- 1:N AI RECOMMENDATION
```

---

# 21. One change I recommend for your current model

I would **not create a separate `Fact Loan Application` table in Dataverse**.

That's the analytical model.

Keep Dataverse clean:

```text
DATAVERSE
=========
Customer
Loan Application
Loan Document
Application Review
Approval
Status History
AI Extraction
AI Validation
AI Recommendation
AI Processing
```

Then Fabric will later transform those into:

```text
FABRIC
======

FACT_LOAN_APPLICATION
FACT_APPLICATION_STATUS
FACT_APPROVAL
FACT_DOCUMENT_PROCESSING
FACT_AI_REVIEW

DIM_CUSTOMER
DIM_DATE
DIM_LOAN_TYPE
DIM_STATUS
DIM_DOCUMENT
DIM_REVIEWER
```

This is the clean separation we want.

---

# 22. Very important: don't break your existing Canvas App yet

Your current Customer Loan Application Canvas App is already connected to the existing `Loan Application` table.

So **don't delete the existing fields immediately**.

Do this in stages.

### Stage A

Add:

```text
Customer lookup
Loan Type lookup
```

and create the new related tables.

### Stage B

Migrate existing customer information into `Customer`.

### Stage C

Update the Canvas App to use:

```text
Customer
+
Loan Application
```

### Stage D

Update your Model-Driven Employee Review App.

### Stage E

Update Power Automate.

### Stage F

Test the complete MVP.

### Stage G

Only then remove obsolete columns.

This prevents your existing application from breaking.

---

# 23. How your Canvas App will change

Currently you probably have something conceptually like:

```text
Canvas App
    |
    v
Loan Application
    |
    +-- Customer Name
    +-- Address
    +-- Email
    +-- Loan Amount
    +-- Income
```

Eventually:

```text
Canvas App
    |
    +------------------+
    |                  |
    v                  v
Customer          Loan Application
    |                  |
    |                  +-- Loan Amount
    |                  +-- Loan Term
    |                  +-- Rate
    |                  +-- Status
    |
    +-- Name
    +-- Address
    +-- Email
```

The application record has:

```text
Customer = [lookup]
```

rather than storing the entire customer profile repeatedly.

---

# 24. And Power Pages becomes much cleaner

Your authenticated customer experience becomes:

```text
Authenticated Customer
          |
          v
       Customer
          |
          v
  Customer's Applications
          |
          v
   Selected Application
          |
    +-----+-----+
    |           |
    v           v
 Documents    Status
    |
    v
 AI Processing
```

This is particularly important for your Customer Copilot.

The agent can effectively follow:

```text
Authenticated User
        ↓
Customer
        ↓
Loan Application
        ↓
Documents
        ↓
AI Summary
```

rather than trying to identify a customer using an application number supplied in chat.

---

# 25. And the employee experience

Your Model-Driven App becomes:

```text
EMPLOYEE
   |
   v
LOAN APPLICATION
   |
   +---- Customer
   |
   +---- Documents
   |       |
   |       +---- AI Extraction
   |       +---- Validation
   |
   +---- Reviews
   |
   +---- Approvals
   |
   +---- Status History
   |
   +---- AI Recommendation
```

This is exactly the structure we need for the **Employee Conversational Assistant**.

---

# 26. One more table I recommend: AI Configuration

Create:

```text
AI Configuration
```

For example:

| Setting                     |    Value |
| --------------------------- | -------: |
| High Value Threshold        | $150,000 |
| Low Confidence Threshold    |     0.80 |
| AI Recommendations Enabled  |      Yes |
| Document Processing Enabled |      Yes |

Then your Power Automate flow doesn't contain:

```text
Loan Amount >= 150000
```

Instead:

```text
Loan Amount >= AI Configuration[High Value Threshold]
```

This is much better enterprise design.

---

# 27. Relationship map we should implement

Here is the exact relationship blueprint I would use:

```text
CUSTOMER
   |
   | 1:N
   v
LOAN APPLICATION
   |
   +--------------------+
   |                    |
   | 1:N                | 1:N
   v                    v
LOAN DOCUMENT       APPLICATION REVIEW
   |
   | 1:N
   v
AI EXTRACTION RESULT


LOAN APPLICATION
   |
   +---- 1:N ---> APPROVAL
   |
   +---- 1:N ---> STATUS HISTORY
   |
   +---- 1:N ---> AI VALIDATION FINDING
   |
   +---- 1:N ---> AI RECOMMENDATION
   |
   +---- 1:N ---> AI PROCESSING LOG


LOAN TYPE
   |
   | 1:N
   v
LOAN APPLICATION
```

---

# 28. What I want you to do first in Power Apps

Don't create all of these at once.

Let's do this incrementally.

### First create these three tables:

```text
Customer
Loan Type
Loan Document
```

Then create these relationships:

```text
Customer 1:N Loan Application

Loan Type 1:N Loan Application

Loan Application 1:N Loan Document
```

After that, we'll create:

```text
Application Review
Approval
Application Status History
```

Then the AI tables.

That way we can test each layer before moving forward.

---

# 29. Your immediate task

In **make.powerapps.com**:

### A. Open your current solution

Go to:

**Solutions → EIPPONE Loan Platform**

### B. Add the existing

```text
Loan Application
```

### C. Create

```text
Customer
```

### D. Create

```text
Loan Type
```

### E. Create

```text
Loan Document
```

### F. Create these relationships

```text
Customer
    1
    |
    N
Loan Application
```

```text
Loan Type
    1
    |
    N
Loan Application
```

```text
Loan Application
    1
    |
    N
Loan Document
```

**Don't delete any existing columns yet.**

---

## One thing I need from you before we do the migration

The exact columns in your existing **Loan Application** table are not present in our previous conversation—they were only described generally. Since we are now modifying your actual Dataverse schema, I don't want to guess and accidentally tell you to move/delete the wrong fields.

**Send me a screenshot of your current `Loan Application` table's Columns page** (or paste the column names and data types).

Then I can give you the **exact field-by-field migration map**, like:

```text
CURRENT LOAN APPLICATION
          |
          +--> KEEP
          |      Loan Amount
          |      Loan Term
          |      Interest Rate
          |
          +--> MOVE TO CUSTOMER
          |      Customer Name
          |      Address
          |      Phone
          |
          +--> CONVERT TO LOOKUP
          |      Loan Type
          |      Reviewer
          |
          +--> CONVERT TO CHOICE
          |      Status
          |
          +--> NEW
                 Customer Lookup
                 High Value Flag
                 ...
```

Then I'll walk you through the **actual Power Apps UI clicks, relationship configuration, cascade behavior, primary/alternate keys, and how to migrate your existing 50 loan records without breaking your Canvas App, Model-Driven App, or Power Automate flow.**


Absolutely. Since your **Canvas App, Model-Driven App, Power Automate flow, Dataverse table, and Power BI report are already working**, I want you to make a **safe solution backup before we change the Dataverse structure**.

We'll do this carefully. **Do not modify or delete any columns yet.**

## Step 1 — Open Power Apps

Go to:

[Power Apps Maker Portal](https://make.powerapps.com/?utm_source=chatgpt.com)

Make sure you are in the **same environment where your Loan Application platform currently exists**.

At the top-right, check the **Environment**.

You should see the environment where you created:

* Loan Application Dataverse table
* Customer Loan Application Canvas App
* Employee Review Model-Driven App
* Power Automate approval flow
* Power BI integration

**Do not continue if you are in the wrong environment.**

---

# Step 2 — Go to Solutions

In the left navigation:

**Solutions**

You will see a list of solutions.

Look for something similar to:

```text
EIPPONE Loan Platform
```

or perhaps an existing custom solution you created.

### If you already have `EIPPONE Loan Platform`

Open it.

### If you don't have one

Create one.

Click:

**+ New solution**

Enter:

```text
Display name:
EIPPONE Loan Platform

Name:
EIPPONELoanPlatform
```

For the publisher, if you already created an EIPPONE publisher, use that.

If not, create/use something like:

```text
Display name:
EIPPONE

Name:
EIPPONE
```

Then click:

**Create**

---

# Step 3 — Why we are doing this

A solution is essentially a **container for your Power Platform application components**.

Instead of modifying your Dataverse environment without a controlled package, we'll put your application components into:

```text
EIPPONE Loan Platform
```

Conceptually:

```text
EIPPONE Loan Platform
│
├── Dataverse
│   └── Loan Application
│
├── Canvas App
│   └── Customer Loan Application
│
├── Model-Driven App
│   └── Employee Review App
│
├── Power Automate
│   └── Loan Approval Flow
│
└── Other components
```

This makes the redesign much safer.

---

# Step 4 — Check whether Loan Application is already in the solution

Open:

**EIPPONE Loan Platform**

Look at the components inside.

You are looking for:

```text
Tables
    Loan Application
```

If you see **Loan Application**, that's good.

Don't add it again.

---

# Step 5 — If Loan Application is NOT there

Inside your solution click:

**+ Add existing**

Then:

**Table**

Find:

```text
Loan Application
```

Select it.

Then click:

**Next**

You will probably see options related to including objects associated with the table.

For now, choose the option that allows you to add the table and its required components.

If Power Apps asks whether to include:

* Columns
* Relationships
* Views
* Forms
* Charts
* Keys

I recommend including the existing table components because we're going to modify the table while preserving the current application.

---

# Step 6 — Add your Canvas App

Still inside:

**EIPPONE Loan Platform**

Click:

**+ Add existing**

Choose:

**App**

Find:

```text
Customer Loan Application
```

Select it and add it.

You should now have something like:

```text
EIPPONE Loan Platform

Tables
   Loan Application

Apps
   Customer Loan Application
```

---

# Step 7 — Add the Model-Driven App

Again:

**+ Add existing → App**

Find:

```text
Employee Review
```

or whatever the exact name of your Model-Driven App is.

Add it.

Now:

```text
Apps
├── Customer Loan Application
└── Employee Review
```

---

# Step 8 — Add the Power Automate approval flow

Inside your solution:

**+ Add existing → Automation → Cloud flow**

Find your existing approval flow.

It may have a name similar to:

```text
Loan Application Approval
```

Add it.

Your solution should now contain approximately:

```text
EIPPONE Loan Platform
│
├── Tables
│   └── Loan Application
│
├── Canvas Apps
│   └── Customer Loan Application
│
├── Model-driven Apps
│   └── Employee Review
│
└── Cloud Flows
    └── Loan Approval Flow
```

---

# Step 9 — Don't add Power BI yet

For **this backup step**, don't worry about adding your Power BI report.

Your immediate objective is to protect the **Power Platform application layer** while we redesign Dataverse.

We'll deal with the Fabric/Power BI architecture separately.

---

# Step 10 — Check the solution before exporting

At this point, stop and inspect the solution.

You should have at least:

```text
✓ Loan Application table
✓ Customer Loan Application Canvas App
✓ Employee Review Model-Driven App
✓ Loan Approval Power Automate flow
```

If one is missing, **don't export yet**.

---

# Step 11 — Export the solution

Now we create the actual backup.

Inside:

**EIPPONE Loan Platform**

click:

**Export**

Power Apps will launch the export wizard.

You'll normally see something like:

### Before you export

Power Apps may perform a solution checker / dependency check.

Let it run.

If it reports dependencies, don't panic.

The important thing is that the components required by your application are included.

---

# Step 12 — Choose Unmanaged

This is important.

You may be asked:

```text
Export as:

○ Managed
○ Unmanaged
```

Choose:

### **Unmanaged**

For your development environment.

Why?

Because we are actively developing and redesigning the solution.

You want to preserve the editable version.

So:

```text
Development
      ↓
Unmanaged Solution
      ↓
Backup / source package
```

---

# Step 13 — Version the solution

You may be asked for a version number.

Use something like:

```text
1.0.0.0
```

Since this is your **pre-AI / pre-Dataverse redesign backup**, I'd actually use:

```text
1.0.0.0
```

This gives us a clean baseline.

Later:

```text
1.0.0.0 = Current MVP backup

1.1.0.0 = Dataverse redesign

1.2.0.0 = AI Builder

1.3.0.0 = Customer Copilot

1.4.0.0 = Employee Copilot

2.0.0.0 = Fabric + AI production architecture
```

You don't have to follow this exact numbering, but it gives us a useful development history.

---

# Step 14 — Export

Click:

**Export**

Power Apps will generate a ZIP file.

Save it somewhere safe.

I recommend a folder structure such as:

```text
EIPPONE
└── Loan Application Digitalization
    └── Power Platform
        └── Solutions
            ├── Backup
            │   └── EIPPONE Loan Platform 1.0.0.0.zip
            │
            └── Development
```

**Do not modify the ZIP file.**

It is our recovery package.

---

# Step 15 — I recommend one more backup

Because you're about to modify your existing Dataverse schema, I'd also create a **data backup**.

The solution backup protects:

```text
Apps
Flows
Tables
Columns
Relationships
Views
Forms
Configuration
```

But it is not the same thing as having a convenient copy of your **50 existing Loan Application records**.

So before we migrate data, I recommend exporting the current Loan Application data as well.

For example:

```text
Loan Application
        ↓
Export data
        ↓
LoanApplication_MVP_Backup_2026-08-12.csv
```

Keep that alongside the solution ZIP.

---

# Step 16 — Your backup should therefore contain

```text
EIPPONE Loan Platform Backup
│
├── 01_Solution
│   └── EIPPONE Loan Platform 1.0.0.0.zip
│
└── 02_Data
    └── LoanApplication_MVP_Backup_2026-08-12.csv
```

This gives us two levels of protection:

### Application backup

```text
Solution ZIP
```

### Data backup

```text
CSV
```

---

# Step 17 — Very important: don't start redesigning yet

Once you've completed the backup, **stop**.

Do not yet:

* Delete Customer Name
* Delete Address
* Delete Email
* Rename columns
* Change column types
* Delete the Loan Application table
* Change the existing Status column
* Modify the Power Automate flow

We'll do those changes **after the backup is verified**.

---

# Step 18 — What we'll do immediately after the backup

Once you have the backup, our next sequence will be:

```text
CURRENT
Loan Application
       |
       v
BACKUP
       |
       v
CREATE CUSTOMER TABLE
       |
       v
CREATE LOAN TYPE TABLE
       |
       v
CREATE LOAN DOCUMENT TABLE
       |
       v
CREATE RELATIONSHIPS
       |
       v
MIGRATE EXISTING 50 RECORDS
       |
       v
UPDATE CANVAS APP
       |
       v
UPDATE MODEL-DRIVEN APP
       |
       v
UPDATE POWER AUTOMATE
       |
       v
TEST
       |
       v
CREATE AI TABLES
       |
       v
FABRIC INGESTION
```

### One important correction to my previous guidance

Because your existing platform is already working, **don't create a brand-new `EIPPONE Loan Platform` solution and immediately start rebuilding everything inside it if your existing components already belong to another custom solution**.

First inspect **Solutions** and tell me what solutions you currently see.

If you can, **send me a screenshot of your Power Apps → Solutions page**. I can then tell you **exactly which solution to open and what to click next**, rather than having you create a duplicate solution and potentially complicate your environment.

Yes. Since this is for your **EIPPONE Loan Application Digitalization Platform**, I recommend creating a dedicated EIPPONE publisher that you can reuse across your Power Platform solutions.

Use the following values:

| Field                   | Recommended value                                                           | Purpose                                            |
| ----------------------- | --------------------------------------------------------------------------- | -------------------------------------------------- |
| **Display name**        | `EIPPONE`                                                                   | Identifies your organization/publisher             |
| **Description**         | `EIPPONE Simulation Dynamics Inc. Power Platform solutions and components.` | Describes what the publisher is used for           |
| **Prefix**              | `eipp`                                                                      | Unique logical prefix for Dataverse components     |
| **Choice value prefix** | `10000`                                                                     | Numeric prefix for global choice/option-set values |

### Enter them exactly like this

**Display name**

```text
EIPPONE
```

**Description**

```text
EIPPONE Simulation Dynamics Inc. Power Platform solutions and components.
```

**Prefix**

```text
eipp
```

**Choice value prefix**

```text
10000
```

### Why `eipp`?

The publisher prefix becomes part of the logical names of Dataverse components.

For example, instead of a generic logical name such as:

```text
loanapplication
```

you'll get something along the lines of:

```text
eipp_loanapplication
```

Likewise, future components can follow your EIPPONE namespace:

```text
eipp_customer
eipp_loandocument
eipp_applicationreview
eipp_approval
eipp_aiextractionresult
```

This is useful because your environment may eventually contain solutions for:

```text
EIPPONE Loan Platform
EIPPONE SDG Pro
EIPPONE DT-Ops
EIPPONE Corporate Analytics
```

and the prefix makes it much easier to distinguish your components.

### Important: don't use `EIPPONE` as the prefix

Use:

```text
eipp
```

rather than:

```text
EIPPONE
```

Dataverse prefixes are generally kept short and are used in logical/schema names.

### Choice value prefix

For the mandatory **Choice value prefix**, use:

```text
10000
```

This is a numeric namespace used when Dataverse generates values for choices/options. You don't need to manually manage those numbers for your current project.

So your publisher should look like:

```text
Publisher
────────────────────────────────
Display name:          EIPPONE

Description:
EIPPONE Simulation Dynamics Inc.
Power Platform solutions and components.

Prefix:                eipp

Choice value prefix:   10000
────────────────────────────────
```

Then click **Save** or **Create**.

**One important point:** once you create the publisher and start creating Dataverse components with `eipp`, don't change the prefix later. It becomes part of the components' logical names.
