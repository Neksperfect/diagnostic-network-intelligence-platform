# DNIP Database Design and Data Model

## 1. Introduction

This document describes the relational database structure of the Diagnostic Network Intelligence Platform (DNIP).

The database is implemented in PostgreSQL and models the operational, laboratory, financial, and administrative activities of the fictional Meridian Diagnostics Network (MDN).

The data model is designed to maintain data integrity, support transactional operations, and provide a reliable foundation for analytical reporting.

## 2. Database Design Objectives

The database design aims to:

1. Represent the relationships between patients, specimens, tests, and laboratory operations.
2. Connect laboratory services to invoices, payments, and revenue recognition.
3. Support branch-level operational and financial reporting.
4. Maintain referential integrity through primary and foreign keys.
5. Enforce business rules through database constraints and triggers.
6. Support data validation, audit tracking, and future ETL workflows.
7. Provide a structured foundation for analytical data marts.

## 3. Database Architecture

The DNIP database is named `dnip` and is organised into four schemas.

| Schema | Purpose | Current status |
|---|---|---|
| `staging` | Temporary landing area for incoming and generated data | Created, tables pending |
| `core` | Main relational database containing operational and financial records | 30 tables created |
| `mart` | Analytical models and reporting structures | Created, tables pending |
| `audit` | ETL execution history and data quality issue tracking | 2 tables created |

The `core` schema is the primary transactional data layer.

The `staging` schema will support data loading and transformation, while the `mart` schema will contain reporting-oriented structures.

The `audit` schema supports monitoring and traceability.

## 4. Core Database Tables

The core schema contains 30 relational tables grouped by business function.

### 4.1 Organisational and Reference Data

| Table | Purpose |
|---|---|
| `branches` | Stores diagnostic network branch information |
| `collection_centres` | Stores specimen collection centre information |
| `test_categories` | Defines laboratory test categories |
| `specimen_types` | Defines supported specimen types |
| `tests` | Stores the diagnostic test catalogue |
| `test_specimen_types` | Maps tests to compatible specimen types |

### 4.2 Patient and Laboratory Operations

| Table | Purpose |
|---|---|
| `patients` | Stores synthetic patient demographic records |
| `staff` | Stores employee and staff information |
| `equipment` | Stores laboratory equipment records |
| `specimens` | Tracks collected and received specimens |
| `test_instances` | Records individual test orders and processing activities |
| `equipment_downtime` | Records equipment downtime and maintenance events |
| `quality_control_events` | Stores laboratory quality control records |

### 4.3 Customer and Financial Operations

| Table | Purpose |
|---|---|
| `customers` | Stores customer account information |
| `payers` | Stores individuals, corporate entities, and insurance payers |
| `invoices` | Stores customer and payer invoices |
| `invoice_lines` | Connects invoiced services to individual test instances |
| `payment_transactions` | Records payment transactions |
| `payment_allocations` | Links payments to invoices |
| `payment_allocation_reversals` | Records reversals of payment allocations |
| `payment_refunds` | Records payment refund requests and outcomes |
| `invoice_adjustments` | Records invoice credit notes, write-offs, and debit notes |
| `revenue_recognition` | Records revenue recognition and reversal events |
| `settlement_batches` | Records payment processor settlement batches |
| `settlement_lines` | Connects settlement batches to payment transactions |
| `receivables_impairments` | Records provisions against receivable balances |
| `expenses` | Records operational expenses |

### 4.4 Inventory Management

| Table | Purpose |
|---|---|
| `reagents` | Stores laboratory reagent catalogue information |
| `inventory_lots` | Tracks individual reagent batches and their costs |
| `inventory_movements` | Records inventory receipts, issues, transfers, and adjustments |

## 5. Audit Tables

The audit schema contains two tables.

| Table | Purpose |
|---|---|
| `audit.etl_runs` | Tracks data extraction, transformation, and loading executions |
| `audit.data_quality_issues` | Records detected data quality issues |

These tables support operational monitoring and traceability of future data processing workflows.

## 6. Key Entity Relationships

The database uses primary keys and foreign keys to maintain relationships between business entities.

### 6.1 Organisational Relationships

- One branch can have multiple collection centres.
- One branch can have multiple staff members.
- One branch can have multiple equipment records.
- One branch can have multiple operating expenses.
- One test category can contain multiple tests.
- Tests and specimen types have a many-to-many relationship through `test_specimen_types`.

### 6.2 Patient and Laboratory Relationships

- One patient can have multiple specimens.
- One collection centre can receive multiple specimens.
- One specimen type can be associated with multiple specimens.
- One specimen can have multiple test instances.
- Each test instance references a test from the test catalogue.
- Test instances can reference staff and equipment.
- Repeat test instances can reference an earlier test instance.
- Equipment can have multiple downtime and quality control records.

### 6.3 Financial Relationships

- One customer can have multiple invoices.
- One payer can be associated with multiple invoices and payment transactions.
- One invoice can contain multiple invoice lines.
- Each invoice line references a test instance.
- One payment transaction can be allocated across multiple invoices.
- One invoice can receive allocations from multiple payment transactions.
- Payment allocations can have reversal records.
- Payment transactions can have refund records.
- Invoice adjustments reference their associated invoices.
- Revenue recognition records reference invoice lines.
- Settlement batches contain settlement lines linked to payment transactions.

### 6.4 Inventory Relationships

- One reagent can have multiple inventory lots.
- One inventory lot can have multiple inventory movements.
- Inventory movements can reference a branch, staff member, and related test instance.

## 7. Primary Keys and Referential Integrity

Each core entity uses a primary key to uniquely identify its records.

Foreign keys enforce valid relationships between related tables.

The database also uses unique constraints to prevent duplicate business identifiers, including:

- Patient codes
- Specimen accession codes
- Test instance codes
- Invoice codes
- Payment codes
- Customer codes
- Payer codes
- Branch codes
- Equipment codes
- Reagent codes

Additional unique constraints apply to selected reference data and transaction relationships.

## 8. Database Integrity and Business Rules

The database uses several mechanisms to protect data integrity.

### 8.1 Primary and Foreign Key Constraints

These ensure that records are uniquely identified and that referenced records exist.

### 8.2 Unique Constraints

These prevent duplicate identifiers and enforce selected business relationships.

For example, each test instance can appear in only one invoice line.

### 8.3 Check Constraints

Check constraints enforce valid values and logical conditions, including:

- Positive payment and refund amounts.
- Valid payment methods and statuses.
- Valid invoice statuses.
- Valid specimen and test statuses.
- Valid staff employment statuses.
- Valid inventory movement types.
- Valid equipment operational statuses.
- Consistent invoice totals and VAT calculations.
- Valid date relationships.

### 8.4 Triggers and Validation Functions

Database triggers and validation functions enforce more complex financial and operational rules.

Examples include:

- Preventing unauthorised changes to payment allocations.
- Validating payment allocation amounts.
- Restricting refunds to eligible payments.
- Validating settlement lines against payment balances.
- Controlling payment status transitions.
- Protecting financial transaction records from inappropriate modification.

Detailed business rules and validation behaviour are maintained in the master technical and financial decisions log.

## 9. Financial Data Relationships

Financial records are connected through a controlled transaction flow.

A diagnostic service is represented by a test instance. The service is invoiced through an invoice line, and revenue recognition is recorded against that invoice line.

Payments are recorded separately and allocated to invoices.

This separation allows the platform to distinguish:

- Services performed and revenue recognised.
- Cash received from customers or payers.
- Outstanding invoice balances.
- Payment refunds and allocation reversals.
- Payment processor settlements.
- Financial adjustments and receivables impairments.

Cash collection and revenue recognition are therefore treated as separate financial events.

## 10. Data Flow Overview

The intended data flow is:

1. Reference data establishes branches, collection centres, test categories, specimen types, and tests.
2. Patient and customer records establish the entities involved in service delivery.
3. Specimen records capture collection and receipt activities.
4. Test instances capture laboratory processing and validation.
5. Invoice records connect completed services to financial billing.
6. Payment transactions and allocations record cash collection.
7. Revenue recognition records capture recognised service revenue.
8. Inventory, equipment, staffing, and expense records support operational cost and performance analysis.
9. Staging and transformation workflows prepare data for analytical reporting.
10. Reporting models provide structured datasets for dashboards and automated reports.

## 11. Analytical Data Model

The `mart` schema is reserved for analytical reporting structures.

Planned reporting models will support:

- Business performance analysis.
- Laboratory operations and quality analysis.
- Financial reporting and reconciliation.
- Inventory and operating cost analysis.
- Data quality and ETL monitoring.

The analytical structures will be designed after the source data generation and validation processes are established.

## 12. Current Implementation Status

The following components have been implemented:

- PostgreSQL database `dnip`.
- Four database schemas.
- Thirty core relational tables.
- Two audit tables.
- Primary and foreign key relationships.
- Unique and check constraints.
- Initial reference data.
- Financial validation functions and triggers.
- Payment, allocation, refund, and settlement controls.

Staging tables, analytical reporting structures, and the full synthetic operational dataset remain part of the next development stages.

---

**Project:** Diagnostic Network Intelligence Platform (DNIP)  
**Database:** PostgreSQL  
**Database name:** `dnip`  
**Core schema:** `core`  
**Audit schema:** `audit`  
**Reporting period:** November 2022 – October 2026  
**Data classification:** Fully synthetic