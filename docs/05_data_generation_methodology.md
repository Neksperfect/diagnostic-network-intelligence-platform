# DNIP Data Generation Methodology

## 1. Purpose

This document defines how synthetic data will be generated for the
Diagnostic Network Intelligence Platform (DNIP), a portfolio project
modelling the operations and financial performance of a fictional
diagnostic laboratory network.

The methodology is designed to produce data that is relationally
consistent, analytically useful, and sufficiently realistic to support
healthcare business reporting, laboratory operations analysis, and
automated reporting demonstrations.

## 2. Data status and scope

All records generated for DNIP are synthetic. They do not represent real
patients, healthcare providers, diagnostic results, or financial
transactions.

The fictional organisation is **Meridian Diagnostics Network (MDN)**, a
diagnostic network operating across five branches:

-   Lagos Island
-   Ikeja
-   Lekki
-   Surulere
-   Ibadan

The model includes 20 collection centres and operational, laboratory,
inventory, quality, billing, payment, settlement, and expense records.

The main analytical period is **1 November 2022 to 31 October 2026**,
covering 48 months. Test and validation records should be generated with
dates that support this reporting period. Separate technical test
records created during database development may fall outside the
analytical period and must be excluded from the main business dataset.

## 3. Planned data volumes

The following figures are planning targets, not claims about final
loaded row counts. Actual generated volumes must be checked and
documented after each load.

  Data area                    Planned volume
  ------------------------ ------------------
  Patients                             40,000
  Specimens                           120,000
  Test instances                      195,000
  Branches                                  5
  Collection centres                       20
  Staff                                   150
  Equipment                  Approximately 50
  Quality control events               30,000
  Invoice lines                       150,000
  Payment transactions                100,000
  Expenses                             24,000
  Inventory movements                  60,000
  Test catalogue                     60 tests

The reference data already loaded includes 5 branches, 20 collection
centres, 7 test categories, 15 specimen types, 60 tests, and 61
test-to-specimen mappings.

## 4. Generation principles

Data generation must follow these principles:

1.  **Referential integrity:** Child records must reference valid parent
    records. Foreign keys must not be bypassed to make loading easier.
2.  **Valid domain values:** Generated values must comply with the
    database's CHECK constraints and permitted status values.
3.  **Chronological consistency:** Related timestamps and dates must
    follow valid business sequences.
4.  **Financial consistency:** Invoice totals, payment allocations,
    refunds, settlement amounts, and recognised revenue must reconcile
    according to the project's financial rules.
5.  **Reproducibility:** Use fixed random seeds where practical so that
    a dataset can be regenerated consistently during development.
6.  **Traceability:** Record generation and load activity in the audit
    schema where appropriate.
7.  **Separation of test records:** Development and concurrency-test
    records must not be mistaken for the main synthetic business
    dataset.

## 5. Generation tools and workflow

Python is the planned tool for generating synthetic records and
coordinating data loads. PostgreSQL is the system of record and enforces
relational constraints and business rules.

The intended workflow is:

1.  Generate or prepare reference and parent data.
2.  Generate dependent operational records using valid parent
    identifiers.
3.  Load records into PostgreSQL in dependency order.
4.  Run database constraints and data-quality checks.
5.  Reconcile expected and actual row counts.
6.  Record the load outcome and resolve rejected records before
    proceeding.

The `staging` schema is reserved for future landing and preparation
workflows. It currently contains no tables. Staging structures should be
designed when the Python ETL process is implemented, rather than assumed
to exist now.

## 6. Data generation domains

### 6.1 Reference and organisational data

Use the established reference data for branches, collection centres,
test categories, specimen types, tests, and test-to-specimen mappings.

Generate staff and equipment records with valid branch relationships.
Staff employment dates and statuses must be plausible within the
reporting period. Equipment must have a valid branch, installation date,
and operational status.

### 6.2 Patient and customer data

Generate synthetic patient identifiers and demographic attributes.
Patient records must use permitted sex values: `Female`, `Male`,
`Unknown`, or `NULL`.

Generate customer records separately from patient records. Customers
represent the entities billed by the network and may be individuals or
corporate customers. Do not assume that every patient is a customer or
that every customer is a patient.

### 6.3 Payer data

Generate payer records for the payer types supported by the schema:
`Individual`, `Corporate`, and `Insurance`. Credit terms must be zero or
greater.

The payer recorded on an invoice is authoritative. A payment allocated
to an invoice must have the same payer as that invoice.

### 6.4 Specimens and test instances

Generate specimens with valid patient, collection-centre, and
specimen-type references. Collection and receipt timestamps must be
chronologically valid. Rejection flags, rejection reasons, rejection
timestamps, and specimen status must agree with one another.

Generate test instances using valid specimen, test, staff, and equipment
references where required. Test status and timestamps must follow the
permitted workflow:

-   `Ordered`
-   `In Progress`
-   `Completed`
-   `Validated`
-   `Failed`
-   `Cancelled`

Processing cannot start before the order. Completion cannot precede the
order. A validated test must have a completion timestamp, and validation
must occur at or after completion.

Repeat tests must reference an existing original test instance. A record
marked as a repeat must have a repeat reference; a non-repeat must not.
A test instance cannot reference itself as its original.

### 6.5 Quality control and equipment downtime

Generate quality control events against valid equipment, with optional
staff and test references where appropriate. Use the permitted QC
statuses: `Pass`, `Fail`, and `Warning`. When both expected minimum and
maximum values are present, the minimum must not exceed the maximum.

Generate equipment downtime with valid equipment references and optional
reporting and resolution staff. Permitted statuses are `Open`,
`Under Repair`, and `Resolved`. A resolved event must have a resolution
timestamp, and that timestamp cannot precede the start time.

### 6.6 Inventory and expenses

Generate reagents and inventory lots with valid reagent references. Lot
expiry dates must not precede receipt dates. Unit cost, when present,
must be zero or greater.

Inventory movements must use permitted movement types:

-   `Receipt`
-   `Issue`
-   `Adjustment In`
-   `Adjustment Out`
-   `Transfer In`
-   `Transfer Out`
-   `Expired`
-   `Return In`
-   `Return Out`

Movement quantities must be greater than zero. Branch, lot, staff, and
related test references must be valid when supplied.

Generate expenses with valid branch references, positive amounts, and
permitted payment statuses: `Paid`, `Unpaid`, or `Partially Paid`. Where
an expense is linked to a test instance, the reference must exist.

## 7. Billing, payments, and financial data

Financial data must follow the project's accounting and transaction
rules. Cash received, recognised revenue, VAT, and outstanding
receivables are separate measures and must not be treated as
interchangeable.

### 7.1 Invoices and invoice lines

Generate invoices for valid customers and payers. Invoice due dates must
be on or after invoice dates. Invoice statuses must use the permitted
values:

-   `Draft`
-   `Issued`
-   `Partially Paid`
-   `Paid`
-   `Overdue`
-   `Cancelled`

Invoice totals must reconcile:

-   Line subtotal = quantity × unit price − line discount.
-   Line total = line subtotal + VAT amount.
-   Invoice subtotal = sum of applicable line subtotals.
-   Invoice VAT = sum of applicable line VAT amounts.
-   Invoice total = subtotal + VAT.

The database enforces one invoice line per test instance through a
unique constraint on `invoice_lines.test_instance_id`. Generated invoice
lines must therefore not assign the same test instance to multiple
invoice lines.

### 7.2 Payment transactions and allocations

Generate payment transactions with positive amounts and permitted
payment methods:

-   `Bank Transfer`
-   `POS`
-   `Card`
-   `Cash`
-   `Online Payment`

Payment statuses must use `Successful`, `Pending`, `Failed`, `Reversed`,
or `Refunded`.

Payments and invoices have a many-to-many financial relationship through
payment allocations. Use allocation records to connect payments to
invoices. Do not assume that one payment always settles one invoice or
that one invoice is always settled by one payment.

Only eligible successful payments may be allocated, and the payer on the
payment must match the payer on the invoice. Allocations must not exceed
the available payment amount or the invoice's adjusted outstanding
balance. Allocation corrections must use the project's reversal records
rather than updating or deleting historical allocations.

### 7.3 Refunds

Refund records must reference valid payments and have positive amounts.
Permitted refund statuses are `Pending`, `Successful`, `Failed`, and
`Cancelled`.

Pending and successful refunds reserve payment funds. Failed and
cancelled refunds release that reservation. A successful refund must
respect the payment's remaining available amount and the project's
allocation rules.

### 7.4 Settlement batches and lines

Settlement batches support processor settlement for applicable
electronic payment methods. Permitted batch statuses are `Pending`,
`Settled`, and `Failed`.

Settlement lines must reference valid batches and payments. Gross
amounts must be positive, processing fees cannot be negative, and net
settlement must equal gross amount less processing fee.

Cumulative settlement amounts must not exceed the underlying payment
amount. Pending settlement amounts reserve funds; settled batches
represent completed historical settlement, while failed batches release
the reservation.

### 7.5 Adjustments, revenue recognition, and receivables

Invoice adjustments must use `Credit Note`, `Write-off`, or
`Debit Note`, with positive amounts and an approval status of `Pending`,
`Approved`, or `Rejected`. Only approved adjustments affect receivable
balances.

Revenue recognition is linked to invoice lines. Recognition occurs when
the diagnostic service is completed and validated under the project's
revenue policy, not simply when cash is received. Reversal events must
reference the original recognition event and include a reason.

Advance payments remain unearned until the related service is completed.
VAT is tracked separately from service revenue. Receivables impairments
and write-offs are distinct concepts and must not be combined.

## 8. Date and time generation

Generated records must fall within the intended reporting period where
they are part of the main business dataset. Related dates and timestamps
must respect business sequence rules.

Examples include:

-   Patient signup or registration must not occur after the related
    service event without an explicit business reason.
-   Specimen collection must not occur after specimen receipt.
-   Test processing must not start before the order.
-   Test completion must not precede the order or processing start,
    where supplied.
-   Result validation must not precede test completion.
-   Invoice due date must not precede invoice date.
-   Inventory expiry date must not precede lot receipt date.
-   Equipment resolution time must not precede downtime start.

Development-only test records outside the reporting period should be
identifiable through their test codes, references, dates, or documented
exclusion rules. They must not silently enter dashboard totals.

## 9. Data quality and validation

Each generation and loading stage must be followed by validation. At
minimum, checks should cover:

-   Expected versus actual row counts.
-   Duplicate business identifiers and duplicate records.
-   Nulls in required fields.
-   Orphaned foreign-key references.
-   Invalid status or category values.
-   Invalid date and timestamp sequences.
-   Negative or zero values where prohibited.
-   Invoice line and invoice total reconciliation.
-   Payment allocation and outstanding balance reconciliation.
-   Refund and settlement reservation limits.
-   Revenue recognition and reversal consistency.
-   Inventory movement and lot-reference consistency.

Database constraints and triggers provide an important layer of
protection, but they do not replace analytical validation. Additional
SQL checks should be maintained in the project's data-validation area.

## 10. Auditability and load controls

Load processes should record useful execution details in
`audit.etl_runs`, including the process name, start and end times,
outcome, and relevant row counts or error information supported by the
table design.

Data-quality exceptions should be captured in
`audit.data_quality_issues` where appropriate. Before implementing audit
inserts, confirm the exact table columns and required fields from
PostgreSQL metadata.

Load scripts should be repeatable and should avoid creating duplicate
business records when rerun. Use stable business keys and explicit load
strategies. Identity columns must be generated by PostgreSQL; do not
manually assign identity values. Use returned identifiers or controlled
key lookups to link dependent records.

## 11. Synthetic data limitations

DNIP data is designed to demonstrate database design, SQL analysis,
healthcare operations reporting, financial reconciliation, and reporting
automation.

It is not clinical evidence and must not be used to infer real disease
prevalence, diagnostic accuracy, patient behaviour, actual laboratory
performance, or real market conditions. Synthetic patterns and
distributions are modelling choices. Any dashboard or report using this
data must clearly disclose that the underlying records are synthetic.

## 12. Implementation status

This document defines the intended generation methodology. It does not
mean that all planned records have already been generated or loaded.

The reference data currently loaded consists of 5 branches, 20
collection centres, 7 test categories, 15 specimen types, 60 tests, and
61 test-to-specimen mappings. The core schema contains 30 relational
tables, and the audit schema contains 2 tables. The `staging` and `mart`
schemas have been created but are not yet populated.

The next implementation stage is to build and test the synthetic
data-generation and loading scripts in dependency order, then record
actual row counts and validation outcomes.
