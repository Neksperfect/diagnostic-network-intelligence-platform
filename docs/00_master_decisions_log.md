# Diagnostic Network Intelligence Platform (DNIP)

## Master Technical, Data Quality and Financial Decisions Log

**Fictional organisation:** Meridian Diagnostics Network (MDN)\
**Database:** PostgreSQL\
**Analytical period:** 1 November 2022 to 31 October 2026\
**Data:** Fully synthetic\
**Last updated:** 21 November 2026

------------------------------------------------------------------------

## 1. Project data principles

-   The project uses synthetic data only. It contains no real patient
    records.
-   The fictional network has five branches: Lagos Island, Ikeja, Lekki,
    Surulere and Ibadan.
-   The analytical period is 48 months, November 2022 through October
    2026.
-   November 2026 records created for database testing are outside the
    analytical period.
-   Preserve test records for audit history. Exclude them from analysis
    by date and test-record filters rather than deleting them.
-   Synthetic data demonstrates database design, analysis, reporting and
    automation. It does not prove real-world clinical or commercial
    trends.
-   Planned scale: approximately 40,000 patients, 120,000 specimens,
    195,000 test instances, 5 branches, 20 collection centres, 150
    staff, around 50 equipment records, 30,000 QC events, 150,000
    invoice lines, 100,000 payment transactions, 24,000 expenses and
    60,000 inventory movements. These are targets, not loaded counts.

## 2. Database structure and readiness

The `dnip` database contains four schemas:

-   `core`: operational relational tables.
-   `staging`: reserved for incoming/raw data.
-   `mart`: reserved for analytical tables and views.
-   `audit`: ETL and data-quality audit records.

At the readiness check, `core` had 30 physical tables, `audit` had 2
tables, and `staging` and `mart` had 0 tables each. The empty schemas
are intentional at this stage.

The 30 core tables are:

`branches`, `collection_centres`, `test_categories`, `specimen_types`,
`tests`, `test_specimen_types`, `patients`, `staff`, `customers`,
`payers`, `equipment`, `specimens`, `test_instances`,
`equipment_downtime`, `quality_control_events`, `invoices`,
`invoice_lines`, `payment_transactions`, `payment_allocations`,
`payment_allocation_reversals`, `invoice_adjustments`,
`revenue_recognition`, `settlement_batches`, `settlement_lines`,
`receivables_impairments`, `expenses`, `reagents`, `inventory_lots`,
`inventory_movements`, `payment_refunds`.

Audit tables: `audit.etl_runs` and `audit.data_quality_issues`.

Reference data already loaded and checked:

-   5 branches
-   20 collection centres
-   7 test categories
-   15 specimen types
-   60 tests
-   61 test/specimen mappings

Do not recreate these reference records in the generator. Read their IDs
from the database. The separate `learning` database must not be
modified.

## 3. Foreign-key relationships and planned load order

Key dependencies:

-   Collection centres, staff, equipment and expenses depend on
    branches.
-   Equipment downtime and QC events depend on equipment.
-   Specimens depend on patients, collection centres and specimen types.
-   Test instances depend on specimens and tests, and may reference
    staff and equipment.
-   Invoices depend on customers and payers.
-   Invoice lines depend on invoices and test instances.
-   Revenue recognition depends on invoice lines.
-   Payment transactions depend on payers.
-   Payment allocations depend on payment transactions and invoices.
-   Payment refunds depend on payment transactions.
-   Allocation reversals depend on allocations and may reference
    refunds.
-   Settlement lines depend on settlement batches and payments.
-   Inventory lots depend on reagents.
-   Inventory movements depend on inventory lots and branches, and may
    reference staff and test instances.
-   Invoice adjustments and receivables impairments depend on invoices.

Planned loading order:

1.  Existing reference data.
2.  Customers, payers, staff, equipment and reagents.
3.  Patients, specimens and test instances.
4.  QC events, equipment downtime, inventory lots and movements.
5.  Invoices and invoice lines.
6.  Revenue recognition.
7.  Payment transactions, allocations, settlement batches/lines,
    refunds, allocation reversals and invoice adjustments.
8.  Expenses and receivables impairments.
9.  Validation and reconciliation.

The exact sequence within financial tables must respect installed
triggers. Load in small stages and validate each stage.

## 4. Unique constraints discovered

  Table                  Unique rule
  ---------------------- ------------------------------
  branches               branch_code
  collection_centres     centre_code
  customers              customer_code
  equipment              equipment_code
  expenses               expense_code
  inventory_lots         (reagent_id, lot_code)
  invoice_adjustments    adjustment_code
  invoice_lines          test_instance_id
  invoices               invoice_code
  patients               patient_code
  payers                 payer_code
  payment_refunds        refund_code
  payment_transactions   payment_code
  reagents               reagent_code
  settlement_batches     batch_code
  specimen_types         specimen_code, specimen_name
  specimens              accession_code
  staff                  staff_code
  test_categories        category_code, category_name
  test_instances         test_instance_code
  tests                  test_code

Important: each `test_instance_id` can appear in only one invoice line.
The generator must not create multiple invoice lines for the same test
instance.

Financial identity columns use `GENERATED ALWAYS` in the tables
inspected during earlier work. Use generated IDs and `RETURNING` to
capture them. Do not manually assign identity values unless deliberately
required and permitted.

## 5. CHECK constraints and permitted values

### Customers and payers

-   Customer type: `Individual` or `Corporate`.
-   Payer type: `Individual`, `Corporate` or `Insurance`.
-   Payer credit terms must be zero or greater.

### Patients and staff

-   Patient sex: `Female`, `Male`, `Unknown` or `NULL`.
-   Staff employment status: `Active`, `On Leave`, `Resigned` or
    `Terminated`.

### Equipment and downtime

-   Equipment status: `Operational`, `Under Maintenance`,
    `Out of Service` or `Retired`.
-   Downtime status: `Open`, `Under Repair` or `Resolved`.
-   Resolved downtime requires `resolved_at`; non-resolved downtime
    requires `resolved_at` to be `NULL`.
-   A populated resolution timestamp must be on or after `started_at`.

### Specimens

-   Status: `Registered`, `Received`, `In Processing`, `Completed`,
    `Rejected` or `Cancelled`.
-   `collected_at` cannot be later than `received_at`.
-   A non-rejected specimen has `is_rejected = false`, with rejection
    reason and rejection timestamp both `NULL`.
-   A rejected specimen has `is_rejected = true`, with both rejection
    reason and rejection timestamp populated.

### Test instances

-   Status: `Ordered`, `In Progress`, `Completed`, `Validated`, `Failed`
    or `Cancelled`.
-   Processing start and completion timestamps cannot precede
    `ordered_at`.
-   A validation timestamp requires a completion timestamp and cannot
    precede it.
-   If `is_repeat = false`, `repeat_of_test_instance_id` must be `NULL`.
-   If `is_repeat = true`, `repeat_of_test_instance_id` must be
    populated.
-   A test instance cannot reference itself as its repeat source.

### Quality control

-   QC status: `Pass`, `Fail` or `Warning`.
-   If both expected bounds are populated, `expected_min` cannot exceed
    `expected_max`.

### Inventory and expenses

-   Inventory movement types: `Receipt`, `Issue`, `Adjustment In`,
    `Adjustment Out`, `Transfer In`, `Transfer Out`, `Expired`,
    `Return In`, `Return Out`.
-   Movement quantity must be greater than zero.
-   Inventory lot unit cost must be zero or greater when supplied.
-   Expiry date cannot precede received date.
-   Expense amount must be greater than zero.
-   Expense payment status: `Paid`, `Unpaid` or `Partially Paid`.

### Invoices and invoice lines

-   Invoice status: `Draft`, `Issued`, `Partially Paid`, `Paid`,
    `Overdue` or `Cancelled`.
-   Due date cannot precede invoice date.
-   Invoice subtotal, VAT and total must be non-negative.
-   Invoice total must equal subtotal plus VAT.
-   Invoice-line quantity must be greater than zero.
-   Unit price, discount, VAT rate, subtotal and VAT amount must be
    non-negative.
-   Discount cannot exceed quantity multiplied by unit price.
-   Line total must equal line subtotal plus VAT.

### Payments and refunds

-   Payment methods: `Bank Transfer`, `POS/Card`, `Cash`,
    `Online Payment`.
-   Payment statuses: `Successful`, `Pending`, `Failed`, `Reversed`,
    `Refunded`.
-   Payment amount must be greater than zero.
-   Allocation amount and refund amount must be greater than zero.
-   Refund statuses: `Pending`, `Successful`, `Failed`, `Cancelled`.

### Allocation reversals

-   Reversal amount must be greater than zero.
-   Reversal types: `Refund` or `Correction`.
-   A Refund reversal must reference a refund.
-   A Correction reversal must not reference a refund.

### Adjustments, revenue recognition and settlements

-   Adjustment types: `Credit Note`, `Write-off`, `Debit Note`.
-   Approval statuses: `Pending`, `Approved`, `Rejected`.
-   Adjustment amount must be greater than zero.
-   Recognition event types: `Recognition`, `Reversal`.
-   Recognition amount must be greater than zero.
-   A Recognition event cannot reference another recognition.
-   A Reversal event must reference the original recognition and include
    a non-empty reason.
-   Settlement statuses: `Pending`, `Settled`, `Failed`.
-   Settlement gross must be greater than zero; processing fee and net
    amount must be zero or greater.
-   Net settlement must equal gross settlement minus processing fee.

## 6. Financial design decisions

### Revenue versus cash

-   Cash received is not the same as revenue recognised.
-   Revenue is recognised when the diagnostic service is completed and
    the result is validated, according to the project's recognition
    event.
-   Payment date records cash timing; it does not determine revenue
    recognition date.
-   VAT is tracked separately from service revenue.
-   Advance payments remain unearned until the service is completed.
-   Payment receipts must not be counted as revenue a second time.

### Contribution margin and operating profit

-   Initial test contribution margin = recognised revenue less direct
    costs attributable to the test.
-   Do not allocate shared costs to individual tests without a
    defensible allocation method.
-   Operating profit = recognised revenue less eligible operating
    expenses, with direct costs counted only once.
-   Processor fees are separate expenses and must not be hidden by
    netting them against revenue.

### Invoices and payers

-   The payer on the invoice is authoritative for the receivable.
-   The payer on a payment must match the payer on the invoice to which
    the payment is allocated.
-   Payments and invoices have a many-to-many relationship through
    `payment_allocations`.
-   Invoice lines reference individual test instances.
-   Revenue recognition references `invoice_line_id`.

### Invoice adjustments

-   Credit notes, write-offs and debit notes are distinct adjustment
    types.
-   Only approved adjustments affect receivable balances.
-   Credit notes and write-offs both reduce receivables but represent
    different accounting events.
-   Bad-debt provisions are separate from write-offs.
-   Pending or rejected adjustments must not reduce receivables.

### Payment allocations

-   A payment can be allocated across invoices; an invoice can receive
    allocations from multiple payments.
-   Allocation corrections use immutable reversal records, not updates
    or deletes to the original allocation.
-   Net allocation = original allocated amount less valid reversals.
-   Allocation amounts must not exceed available payment or invoice
    balances under the installed validation rules.

### Settlement reconciliation

-   Cash is reconciled directly to cash records.
-   Bank transfers are reconciled to bank statements.
-   POS/Card and Online Payment transactions are reconciled through
    processor settlement.
-   Settlement lines record gross amount, processing fee and net
    settlement amount.
-   Example: ₦50,000 gross less ₦500 fee = ₦49,500 net settlement.
-   Pending settlement batches reserve payment funds.
-   Settled batches confirm settlement.
-   Failed batches release their reservations.
-   Partial settlements across batches are allowed.
-   Cumulative settlement gross must not exceed the original payment
    amount.
-   Historical settled lines are retained as audit history.

### Refunds

-   Refunds are tracked separately from the original payment.
-   Pending and Successful refunds reserve payment funds.
-   Failed and Cancelled refunds release reservations.
-   Refunds plus pending settlement reservations must not exceed the
    original payment amount.
-   A successful full refund requires successful refund total to equal
    the original payment amount, no pending refunds, no net allocations
    and no pending settlement batch.
-   Historical Settled batches are retained after a refund and are not
    treated as pending reservations.
-   A full refund uses payment status `Refunded`, not `Reversed`.
-   Payment reversal is blocked when pending or settled settlement lines
    exist, or when refunds or net allocations remain.

## 7. Trigger and function validation work

Installed functions/triggers cover allocation validation, allocation
reversal validation, invoice adjustment validation, payment allocation
validation, payment refund validation, payment status transitions,
protection against financial changes to payments and invoices,
settlement line validation and settlement batch validation.

Key implementation rules: - Payment allocations are immutable after
insertion; corrections use reversal records. - Refunds are inserted as
`Pending`. - Only a Pending refund may be updated. Financial details
remain immutable; status may change. - Payment rows are locked during
refund and settlement validation to protect against concurrent
reservations. - Settlement lines are insert-only; updates and deletes
are blocked. - Settlement lines require a Pending batch and an eligible
Successful payment using `POS/Card` or `Online Payment`. - Settlement
gross across Pending and Settled batches is checked against the payment
amount. - Pending and Successful refunds are included in refund
reservation checks. - A batch may transition from Pending to Settled or
Failed, subject to validation. - A Settled batch must contain at least
one line, and its payments must still be Successful. - Failed batches
release reservations, but historical lines remain.

### Known implementation risks

-   The successful-refund allocation aggregation may overcount allocated
    amounts if an allocation has multiple reversal rows, because a join
    can duplicate the original allocation amount. This possible defect
    has not yet been corrected or fully retested. Correct it before
    relying on successful-refund edge cases.
-   There may be a lock-order risk: settlement-line validation locks the
    batch before the payment, while refund validation locks the payment
    and then reads settlement batches. Do not claim production-grade
    concurrency control without further review and testing.

## 8. Tests performed and observed results

### Database readiness

-   Confirmed four expected schemas.
-   Confirmed 30 core tables and 2 audit tables.
-   Confirmed reference counts: 5 branches, 20 centres, 7 categories, 15
    specimen types, 60 tests and 61 test/specimen mappings.
-   Inspected core table columns and data types.
-   Inspected foreign keys.
-   Inspected CHECK constraints and unique constraints.

### Test payer and payments

A test payer was inspected: - Payer ID 6. - Code
`CONC-TEST-PAYER-001`. - Name `Concurrency Test Payer`. - Type
`Individual`. - Registration date 16 November 2026.

This is outside the analytical period. Preserve it as test history and
exclude it from the 48-month analytical dataset.

Two test payments were inspected: - Payment ID 12, code
`CONC-TEST-PAYMENT-001`, ₦1,000, Successful, POS/Card, dated 16 November
2026. - Payment ID 13, code `REFUND-RACE-PAYMENT-001`, ₦1,000,
Successful, POS/Card, dated 20 November 2026.

These are test records, not analytical-period transactions.

### Concurrent settlement reservation test

-   A genuine two-session test was performed with auto-commit disabled.
-   Session A inserted a ₦700 settlement line and held its transaction
    open.
-   Session B attempted another ₦700 line against the same ₦1,000
    payment.
-   Session B waited for the payment-row lock.
-   After Session A committed, Session B resumed and validation rejected
    the additional settlement because cumulative reservations would
    exceed the payment amount.
-   The relevant batches were subsequently marked Failed. Failed batches
    release reservations; historical lines remain.
-   This demonstrated that the tested concurrent over-allocation
    scenario was prevented by the payment-row lock and cumulative
    settlement check.

### Refund reservation test

-   Payment 13 had a ₦600 line in a Pending settlement batch.
-   A proposed ₦500 Pending refund was tested.
-   The corrected refund function rejected it with:
    `Refund and pending settlement reservations exceed payment amount.`
-   The test insert was rolled back to its savepoint. No refund was
    committed by that test.
-   The test confirmed that ₦600 pending settlement plus ₦500 proposed
    refund exceeds the ₦1,000 payment.

### Test settlement records to preserve

-   Payment 12 has two ₦700 settlement lines in batches marked Failed.
-   Payment 13 has a ₦600 settlement line in a Pending batch.
-   These are November 2026 test records and must be excluded from the
    November 2022 to October 2026 analytical dataset.
-   Do not delete them merely to make the database look clean. They are
    useful test history.

## 9. Transaction and pgAdmin operating notes

-   The user works in pgAdmin with auto-commit disabled in some
    sessions.
-   Explicitly commit successful function creation/replacement
    transactions.
-   Keep diagnostic sessions free of long-running open transactions.
-   An earlier function replacement waited on a transaction lock because
    another session had an open transaction. The blocking session was
    rolled back; the replacement then completed and was committed.
-   If a statement fails inside an explicit transaction, the transaction
    becomes aborted. Run `ROLLBACK` or roll back to the appropriate
    savepoint before continuing.
-   For concurrency tests, use two separate sessions and explicit
    `BEGIN`/`COMMIT`.
-   Do not confuse a query result from one session with another
    session's transaction state.
-   Never run DNIP setup or test scripts against the separate `learning`
    database.

## 10. Known gaps and work still required

-   Synthetic operational data has not yet been generated.
-   `staging` and `mart` have not yet been populated.
-   The possible refund aggregation defect needs correction and a
    targeted test.
-   The possible settlement/refund lock-order issue needs review before
    any production-readiness claim.
-   Key trigger paths need a documented positive, negative and
    concurrency test matrix.
-   Data generation needs reproducibility controls, such as fixed random
    seeds.
-   The generated dataset needs row-count, referential-integrity,
    uniqueness, date-range, status, financial and cross-table
    reconciliation checks.
-   A November 2026 incremental-load demonstration is planned after the
    main 48-month dataset is stable.

## 11. Documentation and code-management policy

-   Keep exploratory SQL and temporary scripts separate from approved
    project code.
-   After validation, save approved code under a permanent, clearly
    named repository path.
-   Document each permanent script's purpose, prerequisites, inputs,
    outputs and validation checks.
-   Never commit passwords, credential-bearing connection strings,
    private keys, real patient information or secrets.
-   Keep a decision log when a schema, business rule, trigger or metric
    changes.
-   Record why a change was made and which tests need to be rerun.
-   Mark superseded decisions rather than silently changing historical
    records.
-   Commit meaningful milestones, not every experimental edit.

## 12. Current project status

**Completed** - Database and schema setup. - Core table creation. -
Reference data loading. - Column and data-type inspection. - Foreign-key
inspection. - CHECK-constraint inspection. - Unique-constraint
inspection. - Initial financial rules and trigger tests. - Concurrent
settlement reservation test. - Refund reservation rejection test.

**Next** 1. Create the GitHub repository and folder structure. 2. Add
this decision log under `docs/`. 3. Prepare the project overview and
business requirements. 4. Review the known refund aggregation issue
before relying on successful-refund edge cases. 5. Build the synthetic
data generator in controlled stages. 6. Validate each load before moving
to the next stage.

**Important:** This log records decisions and tests completed or
identified so far. It does not claim that every financial edge case has
been fully tested or that the database is production-ready.
