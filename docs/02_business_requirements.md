# Business Requirements Document

## 1. Introduction

This document defines the business requirements for the Diagnostic Network Intelligence Platform (DNIP), a synthetic healthcare analytics project developed around the fictional Meridian Diagnostics Network (MDN).

It identifies the operational, financial, and laboratory quality questions the platform is expected to answer.

The requirements will guide database development, data transformation, analytical SQL, dashboard design, and reporting automation.

## 2. Business Objectives

DNIP aims to provide management with a consolidated view of diagnostic network performance.

The platform should:

1. Monitor financial performance across branches, customers, and payers.
2. Measure laboratory productivity and service delivery.
3. Identify specimen processing delays and rejection patterns.
4. Monitor equipment availability and quality control performance.
5. Track inventory usage and operating expenses.
6. Improve the consistency and reliability of management reporting.
7. Support evidence-based operational and financial decisions.

## 3. Stakeholders and Information Needs

| Stakeholder | Information required |
|---|---|
| Executive Management | Overall business performance, revenue, costs, and branch comparisons |
| Finance Team | Invoicing, collections, receivables, refunds, and financial adjustments |
| Laboratory Management | Test volumes, turnaround time, repeat testing, and specimen rejection |
| Quality Assurance Team | QC results, failure rates, and equipment-related quality issues |
| Operations Management | Branch productivity, staffing, equipment downtime, and inventory |
| Business Development Team | Customer activity, payer performance, and service demand |

## 4. Functional Requirements

### 4.1 Healthcare Business Performance

The platform must support:

- Revenue analysis by branch, test category, and individual test.
- Monthly and yearly revenue trends.
- Customer and payer performance analysis.
- Invoice status and outstanding balance monitoring.
- Payment collection and payment method analysis.
- Refund and payment reversal monitoring.
- Accounts receivable ageing.
- Operating expense analysis by branch and category.
- Test-level contribution margin analysis.

**Key business questions:**

1. Which branches generate the highest recognised revenue?
2. Which tests contribute most to recognised revenue?
3. How much revenue has been recognised over time?
4. How much cash has been collected?
5. What proportion of invoices remains outstanding?
6. Which customers or payers have the highest outstanding balances?
7. What are the major operating expense categories?
8. Which tests generate the highest contribution margin?

### 4.2 Laboratory Operations and Quality

The platform must support:

- Specimen collection and receipt monitoring.
- Specimen rejection analysis.
- Test volume and completion monitoring.
- Test turnaround time analysis.
- Repeat testing analysis.
- Equipment downtime monitoring.
- Quality control pass, warning, and failure analysis.
- Staff and branch productivity analysis.

**Key business questions:**

1. How many specimens are collected and processed?
2. Which branches have the highest specimen rejection rates?
3. Which tests experience the longest turnaround times?
4. What percentage of tests are completed and validated?
5. How frequently are tests repeated?
6. Which equipment records the most downtime?
7. What proportion of QC events pass, fail, or generate warnings?
8. Are operational performance indicators different across branches?

### 4.3 Automated Healthcare Reporting

The platform should support:

- Repeatable data extraction and transformation.
- Automated data quality validation.
- Identification of missing, duplicate, and inconsistent records.
- Scheduled analytical reporting.
- Consolidated management reports.
- Exception reporting for operational and financial issues.

**Key business questions:**

1. Are the source datasets complete and consistent?
2. Which validation rules are failing?
3. Have reporting datasets been refreshed successfully?
4. Which operational or financial exceptions require attention?

## 5. Key Performance Indicators

The following KPIs are proposed for the platform.

| KPI | Definition |
|---|---|
| Total Tests | Count of test instances within the reporting period |
| Completed Tests | Count of tests with completed or validated status |
| Test Completion Rate | Completed tests divided by eligible test instances |
| Specimen Rejection Rate | Rejected specimens divided by received specimens |
| Average Turnaround Time | Average elapsed time between specimen receipt and result validation |
| Repeat Test Rate | Repeat test instances divided by eligible test instances |
| QC Failure Rate | Failed QC events divided by eligible QC events |
| Equipment Downtime | Total recorded downtime duration |
| Recognised Revenue | Revenue recognised from eligible completed services |
| Cash Collections | Successful payment transactions received |
| Outstanding Receivables | Valid invoice balances remaining unpaid |
| Operating Expenses | Recorded operating expenses within the reporting period |
| Test Contribution Margin | Recognised test revenue less attributable direct costs |

Final KPI calculations will be implemented in SQL and validated against the underlying transactional records.

## 6. Financial Reporting Requirements

Financial reporting must distinguish between the following concepts:

### 6.1 Revenue Recognition

Revenue is recognised when the associated diagnostic service has been completed and validated according to the project's accounting rules.

Payment receipt alone does not establish recognised revenue.

### 6.2 Cash Collections

Cash collections represent successful payments received from customers or payers.

They must be analysed separately from recognised revenue.

### 6.3 Accounts Receivable

Outstanding receivables represent valid invoice balances that remain unpaid after considering applicable allocations and approved financial adjustments.

### 6.4 Direct Costs and Contribution Margin

Test-level contribution margin is calculated as recognised test revenue less attributable direct costs.

It should not be presented as net profit because it does not necessarily include all operating expenses.

### 6.5 Operating Expenses

Operating expenses must be reported separately from direct test costs to prevent double-counting.

### 6.6 Refunds and Financial Adjustments

Refunds, payment allocation reversals, credit notes, write-offs, and other adjustments must be represented transparently.

Their effects must follow the project's financial validation rules.

## 7. Data and Reporting Requirements

The reporting environment must:

- Use PostgreSQL as the primary relational database.
- Preserve relationships between operational and financial records.
- Apply database constraints and validation rules.
- Maintain an audit trail for selected data processing activities.
- Support reporting across branches and reporting periods.
- Allow financial and operational metrics to be reconciled to their source records.
- Exclude test and administrative records from the main analytical reporting population.

## 8. Dashboard Requirements

The planned Power BI reporting environment will contain three main dashboards.

### Dashboard 1: Business Performance

Expected content:

- Recognised revenue trends.
- Cash collection trends.
- Branch revenue comparison.
- Test category and test-level revenue.
- Accounts receivable and ageing.
- Operating expense analysis.
- Contribution margin analysis.

### Dashboard 2: Laboratory Operations and Quality

Expected content:

- Specimen and test volumes.
- Specimen rejection rates.
- Test turnaround time.
- Repeat testing trends.
- QC performance.
- Equipment downtime.
- Branch operational comparisons.

### Dashboard 3: Automated Reporting and Data Quality

Expected content:

- Data quality issue counts.
- Validation results.
- ETL execution history.
- Reporting refresh status.
- Exception monitoring.

## 9. Non-Functional Requirements

The platform should:

1. Maintain referential integrity across related database tables.
2. Use clear and consistent naming conventions.
3. Support repeatable data processing.
4. Provide traceability between reports and source records.
5. Protect database credentials and sensitive information.
6. Avoid exposing the production development database to public access.
7. Support controlled access to the public demonstration environment.
8. Maintain documentation for database structures, business rules, and analytical calculations.

## 10. Assumptions and Limitations

- Meridian Diagnostics Network is fictional.
- All project records are synthetic.
- The reporting period is 1 November 2022 to 31 October 2026.
- The platform is an analytical demonstration, not a production clinical system.
- KPI definitions may be refined during implementation as the data model and business rules are validated.
- Analytical findings cannot be interpreted as actual clinical or commercial performance in Nigeria.

## 11. Acceptance Criteria

The business requirements will be considered implemented when:

- The database contains the required operational and financial relationships.
- Synthetic records can be generated and validated.
- Financial calculations reconcile with the underlying transactions.
- Laboratory operational KPIs can be calculated using SQL.
- Reporting views support the required dashboard metrics.
- Power BI dashboards present the agreed business indicators.
- Selected reporting workflows can be executed through Python automation.
- The project can be demonstrated through documented and controlled access.

---

**Project:** Diagnostic Network Intelligence Platform (DNIP)  
**Organisation:** Meridian Diagnostics Network (Fictional)  
**Reporting period:** November 2022 – October 2026  
**Data classification:** Fully synthetic