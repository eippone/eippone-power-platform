<p align="center">
  <img src="https://github.com/eippone/eippone-docs/blob/main/images/EIPPONE_Loan_Application.png" alt="Vulnerability Scorecard" width="800" height="400">
</p


🔗 **Live Dashboard:**
[https://atsuvovor.github.io/power_bi_projects/cyber_attack_insight_dashboard.html](https://atsuvovor.github.io/power_bi_projects/cyber_attack_insight_dashboard.html)

---
## Transforming the Future of Digital Lending

At **EIPPONE**, we are redefining the lending experience through intelligent digital solutions. Our integrated platform brings together **secure online applications, automated approval workflows, real-time application tracking, and interactive business intelligence** to streamline the entire loan lifecycle.

By leveraging **Microsoft Power Platform**, organizations can improve operational efficiency, enhance customer satisfaction, and make faster, data-driven decisions.

### Business Problem

-   Manual paper-based loan applications
-   High processing time and operational effort
-   Data-entry and validation errors
-   Manual document review
-   Limited application-status visibility
-   Delayed customer communication
-   Limited operational and executive analytics

### Solution Objectives

1.  Digitize loan application intake.
2.  Centralize application information and documents in Dataverse.
3.  Validate required information before submission.
4.  Automate acknowledgement, notification, and approval workflows.
5.  Provide employees with a structured review experience.
6.  Allow customers to track application status.
7.  Provide real-time Power BI reporting.
8.  Introduce AI-assisted document extraction, summaries,
    recommendations, and conversational analytics.
9.  Identify high-value applications for enhanced monitoring.
10. Create a scalable foundation for intelligent lending operations.



### System Architecture  

#### Overview

```text
                         EIPPONE LOAN PLATFORM
                                  │
                                  ▼
                           MICROSOFT POWER
                              PLATFORM
                                  │
             ┌────────────────────┼────────────────────┐
             │                    │                    │
             ▼                    ▼                    ▼
          DATAVERSE          POWER AUTOMATE       AI / DOCUMENT
        Data & Relations      Workflows            INTELLIGENCE
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  │
                                  ▼
                           POWER PAGES
                       Web Portal / Experience
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              ▼                   ▼                   ▼
        CUSTOMER PORTAL     EMPLOYEE PORTAL      ANALYTICS
              │                   │                   │
              ▼                   ▼                   ▼
       Canvas App #1       Canvas App #3        Power BI
       Loan Intake         Employee Review       Reports/
                                                  Dashboard
              │                   │                   │
              └───────────────────┼───────────────────┘
                                  │
                                  ▼
                             DATAVERSE
```

#### Power Pages - External web experience/container  
```text
POWER PAGES
│
├── Customer Portal
│     ├── Loan Application Intake
│     │      └── Canvas App
│     │
│     └── Client Application View
│            └── Canvas App
│
├── Employee Portal
│     └── Employee Review
│            └── Canvas App
│
└── Analytics
      └── Power BI
             ├── Executive Dashboard
             ├── Loan Portfolio Reports
             ├── Application Analytics
             └── AI/OCR Quality Metrics

```

#### Architecture Detail

```text
                         EIPPONE LOAN PLATFORM
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
              EXPERIENCE LAYER             INTELLIGENCE
                    │                           │
              POWER PAGES                  AI / OCR
                    │                           │
       ┌────────────┼────────────┐              │
       │            │            │              │
       ▼            ▼            ▼              ▼
   Customer      Employee     Analytics    Document AI
   Portal        Portal       Portal       Processing
       │            │            │              │
       ▼            ▼            ▼              │
   Canvas Apps   Canvas App   Power BI          │
       │            │            │              │
       └────────────┴────────────┴──────────────┘
                            │
                            ▼
                         DATAVERSE
                            │
       ┌────────────────────┼────────────────────┐
       │                    │                    │
       ▼                    ▼                    ▼
  Loan Operations       AI Results          Audit/History
       │                    │                    │
       ▼                    ▼                    ▼
 Applications          Extraction          Processing Log
 Customers             Validation           Status History
 Documents             Recommendation       Reviews
 Approvals                                  Approvals
                            │
                            ▼
                      POWER AUTOMATE
                       Orchestration
```
