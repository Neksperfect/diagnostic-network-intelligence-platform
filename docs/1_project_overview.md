# Diagnostic Network Intelligence Platform (DNIP)

## 1. Project Overview

The Diagnostic Network Intelligence Platform (DNIP) is a synthetic healthcare analytics project designed to demonstrate how data can support operational, financial, and laboratory quality management across a network of diagnostic centres.

The project models a fictional diagnostic provider, **Meridian Diagnostics Network (MDN)**, operating across five branches in Nigeria.

DNIP brings together data from patient registration, specimen collection, laboratory testing, quality control, billing, payments, inventory, staffing, equipment, and operating expenses.

The objective is to build a connected data platform that transforms operational records into reliable information for management reporting and decision-making.

## 2. Business Problem

Diagnostic networks generate data across multiple operational and financial processes. When these records are fragmented, poorly structured, or analysed in isolation, management may struggle to answer important questions, including:

- How are branches performing financially?
- Which tests and services contribute most to recognised revenue?
- How efficiently are specimens processed?
- Where are laboratory delays and rejection rates highest?
- Are equipment downtime and quality control issues affecting operations?
- How effectively are inventory and operating expenses managed?
- How much revenue remains outstanding from customers and payers?

DNIP addresses these questions by integrating related datasets into a structured analytical environment and developing reporting solutions around them.

## 3. Project Objectives

The project aims to:

1. Design a relational PostgreSQL database that represents the operations of a multi-branch diagnostic network.
2. Generate realistic but entirely synthetic healthcare, laboratory, operational, and financial data.
3. Apply data validation, integrity constraints, and financial business rules.
4. Develop SQL-based analytical models for operational and financial reporting.
5. Build interactive dashboards for management and laboratory performance monitoring.
6. Automate selected data processing and reporting workflows using Python.
7. Deploy a controlled demonstration platform that allows reviewers to explore the project.

## 4. Organisation Profile

**Organisation:** Meridian Diagnostics Network (MDN)  
**Organisation type:** Fictional diagnostic laboratory network  
**Operating country:** Nigeria  
**Number of branches:** 5  
**Number of collection centres:** 20  
**Reporting period:** 1 November 2022 to 31 October 2026  
**Data classification:** Fully synthetic data

The fictional network operates through five branches:

- Lagos Island
- Ikeja
- Lekki
- Surulere
- Ibadan

The organisation profile and operational records are created solely for this project. They do not represent an actual healthcare provider.

## 5. Core Project Modules

DNIP is organised around three main modules.

### 5.1 Healthcare Business Performance

This module focuses on the financial and commercial performance of the diagnostic network.

Areas of analysis include:

- Revenue recognition and revenue trends
- Branch and service-line performance
- Customer and payer performance
- Invoice and payment analysis
- Accounts receivable and outstanding balances
- Operating expenses and cost analysis
- Test-level contribution margin

### 5.2 Laboratory Operations and Quality

This module examines laboratory workflow, service delivery, and quality performance.

Areas of analysis include:

- Specimen collection and processing
- Specimen rejection rates
- Test turnaround time
- Test completion and validation
- Repeat testing
- Equipment utilisation and downtime
- Quality control performance
- Staff and branch operational performance

### 5.3 Automated Healthcare Reporting

This module focuses on making reporting more consistent and efficient.

Planned capabilities include:

- Automated data extraction and transformation
- Data quality checks
- Scheduled reporting workflows
- Python-based reporting automation
- Consolidated operational and financial reports
- Exception and performance reporting

## 6. Technology Stack

| Technology | Intended use |
|---|---|
| PostgreSQL | Relational database and analytical SQL |
| Python | Data generation, ETL, validation, and automation |
| Pandas | Data manipulation and processing |
| Power BI | Interactive dashboards and business intelligence |
| Streamlit | Public-facing project demonstration interface |
| Git and GitHub | Version control, source code, and documentation |

Cloud hosting will be considered for the public demonstration environment.

## 7. Data Scope and Synthetic Data Disclaimer

All records used in DNIP are synthetically generated. The project does not use real patient records, personally identifiable information, or confidential data from any healthcare organisation.

The synthetic dataset is designed to simulate relationships and workflows within a diagnostic network, including patients, specimens, tests, invoices, payments, equipment, inventory, and operating expenses.

Although the data is designed to support realistic analytical scenarios, it does not establish actual clinical outcomes, healthcare market conditions, or commercial performance in Nigeria.

Any findings and dashboard results must therefore be interpreted as demonstrations of analytical methods applied to simulated data.

## 8. Project Architecture

The intended architecture consists of the following layers:

1. **Data generation:** Creation of synthetic operational and financial records.
2. **Database layer:** Storage and management of structured data in PostgreSQL.
3. **Data processing:** Validation, transformation, and preparation using SQL and Python.
4. **Analytical layer:** Development of reporting views and business logic.
5. **Visualisation layer:** Interactive dashboards built with Power BI.
6. **Application layer:** A Streamlit interface for exploring selected project outputs.
7. **Deployment layer:** A controlled cloud-hosted demonstration environment.

## 9. Project Deliverables

The planned deliverables include:

- PostgreSQL database and relational data model
- Synthetic data generation methodology
- SQL scripts for schema creation, validation, and analysis
- Data dictionary and business rules documentation
- Python ETL and automation scripts
- Power BI dashboards
- Streamlit demonstration application
- Technical documentation and project walkthrough
- Public GitHub repository

## 10. Project Scope and Limitations

DNIP is an analytical and technical demonstration project. It is not a production laboratory information management system, electronic medical record system, clinical diagnostic tool, or replacement for regulated healthcare software.

The project does not make clinical diagnoses or provide treatment recommendations.

Its primary purpose is to demonstrate database design, data engineering, healthcare business intelligence, laboratory operations analytics, financial controls, and reporting automation in a simulated diagnostic network.

## 11. Project Status

The PostgreSQL database has been created, the core relational tables and audit tables have been established, and initial reference data has been loaded.

Database constraints, financial controls, and validation logic are being developed and tested.

The next development stages include synthetic data generation, staging and transformation workflows, analytical reporting models, dashboards, automation, and deployment.

---

**Project:** Diagnostic Network Intelligence Platform (DNIP)  
**Organisation:** Meridian Diagnostics Network (Fictional)  
**Data period:** November 2022 – October 2026  
**Data status:** Fully synthetic  
**Repository:** GitHub