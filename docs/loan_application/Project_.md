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

If you mean the **AI Extraction Result** table in the redesigned EIPPONE Loan Platform, I recommend structuring it as follows.

### AI Extraction Result

This table stores the **structured data extracted by AI from a Loan Document**—for example, extracting borrower name, income, loan amount, address, employer, etc. from an uploaded PDF.

| Column                        | Data Type              | Purpose                                               |
| ----------------------------- | ---------------------- | ----------------------------------------------------- |
| **AI Extraction Result ID**   | Autonumber             | Human-readable business ID, e.g. `EXT-1000`           |
| **AI Extraction Result GUID** | Unique Identifier      | Dataverse technical primary key                       |
| **Loan Document**             | Lookup                 | Links the extraction result to the **Loan Document**  |
| **Extraction Status**         | Choice                 | Pending, Processing, Completed, Failed                |
| **AI Model**                  | Text                   | Model used for extraction                             |
| **Extraction Version**        | Text                   | Version of the extraction process/model               |
| **Extracted Data**            | Multiple Lines of Text | Stores the extracted structured data, preferably JSON |
| **Confidence Score**          | Decimal Number         | Overall confidence score                              |
| **Extraction Date**           | Date and Time          | When extraction was performed                         |
| **Processing Duration**       | Whole Number           | Processing time, e.g. seconds                         |
| **Error Message**             | Multiple Lines of Text | Error details if extraction failed                    |
| **Created By**                | User                   | Dataverse system column                               |
| **Created On**                | Date and Time          | Dataverse system column                               |
| **Modified By**               | User                   | Dataverse system column                               |
| **Modified On**               | Date and Time          | Dataverse system column                               |

### Important relationship

The key relationship should be:

**Loan Document (1) → AI Extraction Result (Many)**

A single document may be processed multiple times—for example, if you change the AI model or rerun extraction.

So:

```text
Loan Application
      │
      └── Loan Document
              │
              └── AI Extraction Result
                       │
                       ├── Extraction Status
                       ├── AI Model
                       ├── Extracted Data
                       ├── Confidence Score
                       └── Extraction Date
```

### One important design decision

I would **not create a separate column for every possible extracted field** in this table.

For example, don't initially create:

* Extracted First Name
* Extracted Last Name
* Extracted Income
* Extracted Employer
* Extracted Address
* Extracted Loan Amount
* etc.

Instead, keep the raw/structured AI output in **Extracted Data** and use the downstream AI Validation Finding / AI Recommendation tables to interpret and validate it.

That keeps the AI layer flexible when you add new document types and extracted fields later.

**Recommended primary business relationship:**
`AI Extraction Result → Loan Document` = **Lookup**.

If you're creating this table now in Dataverse, I can also give you the **exact Column Name, Data Type, Required/Optional, Schema Name, and Choice values** for every column so you can enter them one by one.

Yes. Following the same architecture we established for **AI Extraction Result**, I would structure the remaining four AI tables like this.

## 8. AI Validation Finding

This table stores **individual problems, warnings, or validation results identified by AI** after extraction.

| Column                         | Data Type              | Purpose                                                                        |
| ------------------------------ | ---------------------- | ------------------------------------------------------------------------------ |
| **AI Validation Finding ID**   | Autonumber             | Business ID, e.g. `VAL-1000`                                                   |
| **AI Validation Finding GUID** | Unique Identifier      | Dataverse technical primary key                                                |
| **AI Extraction Result**       | Lookup                 | Links the finding to the AI extraction that produced it                        |
| **Loan Document**              | Lookup                 | Links directly to the source document                                          |
| **Finding Type**               | Choice                 | Missing Data, Invalid Data, Inconsistency, Fraud Risk, Policy Violation, Other |
| **Severity**                   | Choice                 | Low, Medium, High, Critical                                                    |
| **Finding Status**             | Choice                 | Open, Reviewed, Resolved, Dismissed                                            |
| **Field Name**                 | Text                   | Extracted field involved, e.g. `AnnualIncome`                                  |
| **Finding Description**        | Multiple Lines of Text | Explains what AI found                                                         |
| **Expected Value**             | Multiple Lines of Text | Expected value/rule where applicable                                           |
| **Actual Value**               | Multiple Lines of Text | Value actually extracted                                                       |
| **Confidence Score**           | Decimal Number         | AI confidence in the finding                                                   |
| **Resolution Notes**           | Multiple Lines of Text | Human/reviewer resolution                                                      |
| **Detected On**                | Date and Time          | When finding was created                                                       |
| **Resolved On**                | Date and Time          | When resolved                                                                  |

### Relationship

```text
Loan Document
      │
      └── AI Extraction Result
                 │
                 └── AI Validation Finding
                         ├── Missing Data
                         ├── Invalid Data
                         ├── Inconsistency
                         └── Fraud Risk
```

**Key relationship:**
`AI Extraction Result (1) → AI Validation Finding (Many)`

---

# 9. AI Recommendation

This table stores the **AI-generated recommendation or assessment for a loan application**.

Unlike Validation Finding, which identifies individual issues, this table represents the **overall AI assessment/recommendation**.

| Column                     | Data Type              | Purpose                                                                  |
| -------------------------- | ---------------------- | ------------------------------------------------------------------------ |
| **AI Recommendation ID**   | Autonumber             | Business ID, e.g. `REC-1000`                                             |
| **AI Recommendation GUID** | Unique Identifier      | Dataverse technical primary key                                          |
| **Loan Application**       | Lookup                 | Links recommendation to the loan application                             |
| **AI Extraction Result**   | Lookup                 | Links recommendation to the AI analysis used                             |
| **Recommendation Type**    | Choice                 | Approval Recommendation, Risk Assessment, Document Recommendation, Other |
| **Recommendation**         | Choice                 | Approve, Approve with Conditions, Review Required, Reject                |
| **Risk Level**             | Choice                 | Low, Medium, High, Critical                                              |
| **Recommendation Summary** | Multiple Lines of Text | Human-readable AI explanation                                            |
| **Reasoning**              | Multiple Lines of Text | Detailed rationale                                                       |
| **Confidence Score**       | Decimal Number         | AI confidence                                                            |
| **Recommendation Status**  | Choice                 | Draft, Active, Accepted, Rejected, Superseded                            |
| **Reviewer Decision**      | Choice                 | Approve, Reject, Override, Pending                                       |
| **Reviewer Comments**      | Multiple Lines of Text | Human review comments                                                    |
| **Generated On**           | Date and Time          | When AI generated recommendation                                         |
| **Reviewed On**            | Date and Time          | When human reviewed it                                                   |

### Relationship

```text
Loan Application
       │
       └── AI Recommendation
                │
                ├── Recommendation
                ├── Risk Level
                ├── Confidence Score
                └── Reviewer Decision
```

I recommend allowing **multiple recommendations per Loan Application**, because you may eventually run different AI models or generate recommendations at different stages.

**Key relationship:**
`Loan Application (1) → AI Recommendation (Many)`

---

# 10. AI Processing Log

This is your **technical audit/history table**. It records what the AI system actually did.

For example:

> Document uploaded → OCR → classification → extraction → validation → recommendation.

| Column                     | Data Type              | Purpose                                                     |
| -------------------------- | ---------------------- | ----------------------------------------------------------- |
| **AI Processing Log ID**   | Autonumber             | Business ID, e.g. `LOG-1000`                                |
| **AI Processing Log GUID** | Unique Identifier      | Dataverse technical primary key                             |
| **Loan Application**       | Lookup                 | Application being processed                                 |
| **Loan Document**          | Lookup                 | Document being processed                                    |
| **AI Processing Type**     | Choice                 | OCR, Classification, Extraction, Validation, Recommendation |
| **Processing Status**      | Choice                 | Started, Completed, Failed                                  |
| **AI Model**               | Text                   | AI model/service used                                       |
| **Processing Start**       | Date and Time          | Start time                                                  |
| **Processing End**         | Date and Time          | End time                                                    |
| **Processing Duration**    | Whole Number           | Duration in seconds                                         |
| **Input Reference**        | Multiple Lines of Text | Reference to input/source                                   |
| **Output Reference**       | Multiple Lines of Text | Reference to generated output                               |
| **Error Code**             | Text                   | Technical error code                                        |
| **Error Message**          | Multiple Lines of Text | Technical error details                                     |
| **Retry Count**            | Whole Number           | Number of retries                                           |
| **Created On**             | Date and Time          | Dataverse system column                                     |

### Example

```text
LOG-1001
Loan Document: DOC-1005
Processing Type: Extraction
AI Model: GPT/AI Builder model
Status: Completed
Start: 10:15:02
End: 10:15:14
Duration: 12 seconds
```

### Relationships

You can have:

`Loan Application (1) → AI Processing Log (Many)`

and

`Loan Document (1) → AI Processing Log (Many)`

This table is particularly useful for **monitoring, troubleshooting, auditing, and measuring AI performance**.

---

# 11. AI Configuration

This table is different from the other four.

It stores the **rules and settings that control how your AI processing operates**.

| Column                    | Data Type              | Purpose                                                |
| ------------------------- | ---------------------- | ------------------------------------------------------ |
| **AI Configuration ID**   | Autonumber             | Business ID, e.g. `CFG-1000`                           |
| **AI Configuration GUID** | Unique Identifier      | Dataverse technical primary key                        |
| **Configuration Name**    | Text                   | Primary name, e.g. `Loan Document Extraction`          |
| **AI Function**           | Choice                 | Extraction, Validation, Recommendation, Classification |
| **Document Type**         | Choice/Lookup          | Document type to which configuration applies           |
| **AI Model**              | Text                   | Model/service to use                                   |
| **Model Version**         | Text                   | Model version                                          |
| **Confidence Threshold**  | Decimal Number         | Minimum acceptable confidence                          |
| **Is Active**             | Yes/No                 | Whether configuration is currently active              |
| **Configuration Version** | Text                   | Version of configuration                               |
| **Prompt / Instructions** | Multiple Lines of Text | AI instructions/prompt                                 |
| **Validation Rules**      | Multiple Lines of Text | Rules used for validation                              |
| **Effective From**        | Date and Time          | Start date                                             |
| **Effective To**          | Date and Time          | End date                                               |
| **Notes**                 | Multiple Lines of Text | Administrative notes                                   |

### Example

```text
Configuration Name:
Personal Loan Document Extraction

AI Function:
Extraction

AI Model:
[Your selected AI model]

Confidence Threshold:
0.85

Is Active:
Yes

Configuration Version:
1.0
```

---

# How the five AI tables fit together

This is the architecture I recommend for your EIPPONE platform:

```text
                         LOAN APPLICATION
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
              ▼                 ▼                 ▼
       LOAN DOCUMENT       AI RECOMMENDATION   AI PROCESSING LOG
              │
              ▼
     AI EXTRACTION RESULT
              │
              ▼
     AI VALIDATION FINDING
              
              
              AI CONFIGURATION
                     │
                     │ controls
                     ▼
        ┌────────────────────────────┐
        │ AI Processing              │
        │ • Extraction               │
        │ • Validation               │
        │ • Recommendation           │
        └────────────────────────────┘
```

### The important distinction

| Table                     | Main purpose                                         |
| ------------------------- | ---------------------------------------------------- |
| **AI Extraction Result**  | What AI extracted from the document                  |
| **AI Validation Finding** | What AI found wrong, missing, inconsistent, or risky |
| **AI Recommendation**     | What AI recommends about the application             |
| **AI Processing Log**     | What the AI system did and when                      |
| **AI Configuration**      | How the AI system is configured to operate           |

This gives you a clean separation between **AI output, AI findings, AI decisions, AI audit trail, and AI configuration**.

One adjustment I would make to the earlier design: **AI Recommendation should be tied primarily to Loan Application**, while **AI Extraction Result and AI Validation Finding should be centered on Loan Document**. That matches the business meaning of each table and will scale much better as you add multiple documents to one loan application.




