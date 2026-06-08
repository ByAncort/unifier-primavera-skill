# Cost Manager — Oracle Primavera Unifier
## Technical Reference for Functional Consultants

---

## What Is the Cost Manager?

The **Cost Manager** is the central financial control module in Oracle Primavera Unifier. It provides a unified view of project budget, commitments, actuals, forecasts, and variances across the full project lifecycle.

It functions as the **project general ledger**: every financial movement triggered by a Cost-type Business Process (BP) is reflected here. It is also the primary integration point with ERP systems (Oracle EBS, SAP, Oracle Fusion) and Oracle Primavera P6.

### Object Hierarchy Within the Cost Manager:

```
Cost Manager (Shell)
  ├── CBS (Cost Breakdown Structure)       ← cost code hierarchy
  ├── Cost Sheet                           ← master budget control sheet
  │     ├── Columns (configured per project)
  │     └── Rows (one per active CBS Code)
  ├── SBS (Summary Budget Sheet)           ← executive-level summary view
  ├── Worksheet                            ← budget estimation and modeling
  ├── Budget Changes                       ← formal modifications to approved budget
  ├── Commitments                          ← contracts, POs, subcontracts
  ├── Spends / Actuals                     ← costs actually incurred and paid
  ├── Forecast / EAC                       ← estimate at completion
  └── Cash Flow                            ← time-phased cost distribution
```

---

## 1. CBS — Cost Breakdown Structure

### What Is It?
The CBS is the **hierarchical cost code structure** of the project. It defines the categories to which all financial transactions are allocated.

Accounting equivalents: **Chart of Accounts / WBS Element / Cost Center**.

### Typical CBS Structure:

```
01          CIVIL WORKS
  01.01     Earthworks
  01.02     Foundations
  01.03     Structure
02          INSTALLATIONS
  02.01     Electrical
  02.02     Mechanical
  02.03     Instrumentation
03          PROJECT MANAGEMENT
  03.01     Engineering
  03.02     Procurement
  03.03     Contract Administration
09          CONTINGENCY
  09.01     Design Contingency
  09.02     Construction Contingency
```

### CBS Configuration in uDesigner:

| Field | Description | Consideration |
|-------|-------------|---------------|
| **CBS Code** | Unique node identifier | Must follow agreed naming convention |
| **Description** | Cost item label | Clear and specific |
| **Level** | Hierarchy level (1, 2, 3…) | Only leaf-level nodes accept direct allocation |
| **Status** | Active / Inactive | Inactive codes block new transactions but preserve history |
| **CBS Type** | Normal / Summary | Summary nodes aggregate children; they do not accept direct posting |
| **GL Account** | ERP chart of accounts code | Critical for ERP integration — must be mapped before go-live |
| **Cost Account** | Oracle P6 code | Required if synchronizing with P6 cost accounts |

### Allocation Rules:
- Only **leaf-level CBS nodes** (no children) accept direct cost posting.
- **Summary-type CBS nodes** automatically aggregate values from their children.
- An **Inactive** CBS blocks creation of new records but does not delete existing transactions.
- The CBS Code is the **primary correlation key** between Unifier, the ERP, and P6.

### Common CBS Errors:

| Error | Root Cause | Resolution |
|-------|-----------|------------|
| "CBS Code not found" when creating a BP | CBS is inactive or a summary-level node | Activate the CBS or use a leaf-level code |
| Cost posts to the wrong CBS | Auto-populate misconfigured on the BP | Review and fix the CBS field auto-populate rule |
| CBS does not appear in the BP selector | CBS not enabled for that Shell type | Review Cost Manager configuration in the Shell Template |
| ERP integration fails on CBS | GL Account not mapped | Complete the CBS → GL Account mapping |
| Rollup mismatch between project and program | CBS code exists in child shell but not in parent | Add missing CBS to the program or correct the rollup mapping |

---

## 2. Cost Sheet

### What Is It?
The **Cost Sheet** is the **master cost control sheet** for the project. It presents a consolidated view of all financial values, organized by CBS Code (rows) and cost category (columns).

It is the most important object in the Cost Manager and the primary source for project financial reporting.

### Cost Sheet Anatomy:

```
         | Col A           | Col B            | Col C           | Col D        | Col E      | Col F      |
CBS Code | Original Budget | Approved Budget  | Committed Cost  | Actual Cost  | EAC        | Variance   |
─────────┼─────────────────┼──────────────────┼─────────────────┼──────────────┼────────────┼────────────┤
01       | 1,000,000       | 1,050,000        | 980,000         | 450,000      | 1,020,000  |  30,000    |
  01.01  |   200,000       |   200,000        | 195,000         |  90,000      |   205,000  |  -5,000    |
  01.02  |   350,000       |   375,000        | 360,000         | 180,000      |   375,000  |       0    |
  01.03  |   450,000       |   475,000        | 425,000         | 180,000      |   440,000  |  35,000    |
```

### Column Types:

| Column Type | Data Source | Editable | Description |
|-------------|-------------|----------|-------------|
| **BP Source** | Business Process | No | Auto-populated from approved Cost-type BPs |
| **Manual Entry** | User | Yes | Direct data entry (e.g., initial budget load) |
| **Formula** | Calculation across columns | No | Derived value (e.g., Variance = Budget − EAC) |
| **Rollup from Sub-shells** | Child shells | No | Consolidates from child projects to program |
| **P6 Source** | Oracle Primavera P6 | No | Values synchronized from P6 schedule/cost data |
| **External Source** | ERP integration | No | Values imported from an external system |

### Recommended Standard Columns:

| Column | Type | Accounting Description | Fed By |
|--------|------|------------------------|--------|
| **Original Budget** | Manual Entry | Initial approved budget | Manual load or Budget BP |
| **Budget Transfers** | BP Source | Inter-CBS fund transfers | Budget Transfer BP |
| **Approved Changes** | BP Source | Approved contract changes | Change Order BP (Approved status) |
| **Approved Budget** | Formula | Original + Transfers + Approved Changes | Formula: A + B + C |
| **Pending Changes** | BP Source | Change orders in workflow | Change Order BP (Pending status) |
| **Committed Cost** | BP Source | Signed contracts and POs | Commitment BP |
| **Invoiced** | BP Source | Received and approved invoices | Invoice BP |
| **Actual Cost** | BP Source | Payments executed | Payment BP |
| **Forecast (EAC)** | BP Source / Formula | Estimate at completion | Forecast BP or formula |
| **ETC** | Formula | Remaining cost to complete | Formula: EAC − Actual Cost |
| **Variance** | Formula | Budget deviation | Formula: Approved Budget − EAC |
| **% Budget Used** | Formula | Execution rate | Formula: Actual Cost / Approved Budget × 100 |
| **Available Budget** | Formula | Uncommitted budget remaining | Formula: Approved Budget − Committed Cost |

### Configuring a BP Source Column:

To have a column receive data from a BP, configure:

1. **BP Name** — which Business Process feeds the column
2. **BP Status** — at which workflow status the amount is posted (e.g., "Approved" only)
3. **Line Item Column** — which BP field is summed (e.g., `amount_d`)
4. **CBS Mapping** — how the BP's CBS code maps to the Cost Sheet row
5. **Transaction Type** — Add (increases column) or Subtract (decreases column)

> **Important:** A single Cost Sheet column can receive data from multiple BPs. For example, the "Actual Cost" column may aggregate from both a Payment BP and a Direct Cost BP.

### Common Cost Sheet Formulas:

```
Approved Budget    = Original Budget + Budget Transfers + Approved Changes
EAC                = Actual Cost + ETC (Remaining Forecast)
Variance           = Approved Budget − EAC
% Complete         = (Actual Cost / EAC) × 100
% Budget Used      = (Actual Cost / Approved Budget) × 100
Available Budget   = Approved Budget − Committed Cost
Cost at Risk       = Pending Changes + Contingency Used
Retainage Held     = Invoiced × Retainage Rate
Net Due            = Invoiced − Retainage Held
```

### When to Trigger a Manual Recalculation:

- After approving a BP that should be reflected but is not yet showing
- After modifying a column formula in uDesigner
- After activating or deactivating a CBS Code
- After a bulk data import (CSV or integration)
- After adjusting the BP Source configuration on a column

**How to recalculate:** Cost Manager → Cost Sheet → **Actions → Recalculate**

---

## 3. SBS — Summary Budget Sheet

### What Is It?
The **SBS** is a high-level summary view that groups CBS codes into **management-level categories** (e.g., by discipline, phase, or cost type). It is the equivalent of an executive budget execution report.

### Cost Sheet vs. SBS:

| | Cost Sheet | SBS |
|--|------------|-----|
| **Granularity** | Per CBS Code (detail) | Per category (summary) |
| **Direct editing** | Yes (manual columns) | Generally No |
| **Primary use** | Day-to-day operational control | Management and executive reporting |
| **Typical row count** | Can be hundreds | Typically 10–30 categories |
| **Formula support** | Yes (column formulas) | Yes (category-level formulas) |

### SBS Configuration Steps:

1. Define **SBS categories** (summary rows)
2. Assign which **CBS Codes** belong to each category
3. Define the **columns** to display (can differ from Cost Sheet columns)
4. Configure **category-level formulas** for subtotals if needed
5. Set **rollup rules** from child shells if this SBS is at program level

### Example SBS Structure:

```
SBS Category              CBS Included         Budget      Committed   Actuals   Variance
──────────────────────────────────────────────────────────────────────────────────────────
Engineering & Design      03.01, 03.02         500,000     480,000     320,000    20,000
Civil Works               01.01, 01.02, 01.03  2,000,000   1,900,000   850,000   100,000
Installations             02.01, 02.02, 02.03  1,500,000   1,450,000   600,000    50,000
Project Management        03.03                  300,000     290,000   180,000    10,000
Contingency               09.01, 09.02           400,000      50,000         0   350,000
──────────────────────────────────────────────────────────────────────────────────────────
TOTAL PROJECT                                  4,700,000   4,170,000 1,950,000   530,000
```

### SBS Cell Types:

| Type | Function |
|------|----------|
| **Rollup** | Sums from child shell Cost Sheets |
| **Formula** | Calculated from other SBS columns |
| **Manual** | Directly entered value |
| **Derived** | Calculated from another SBS |

---

## 4. Worksheet

### What Is It?
The **Worksheet** is a spreadsheet-like tool within the Cost Manager used to **estimate and build the project budget** before formally loading it into the Cost Sheet. It supports unit-rate calculations, quantity takeoffs, and scenario modeling.

### Use Cases:
- Cost estimation during conceptual or detailed engineering phases
- Building the baseline budget before formal approval
- Scenario analysis ("what-if" modeling)
- Contingency calculation by risk item
- Quantity-based cost build-up (unit cost × quantity)

### Typical Structure:

```
Item | Description               | Quantity | Unit | Unit Price | Factor | Total
─────┼───────────────────────────┼──────────┼──────┼────────────┼────────┼──────────
1    | Rock excavation           | 5,000    | m³   | 45.00      | 1.15   | 258,750
2    | Compacted backfill        | 8,000    | m³   | 12.00      | 1.00   |  96,000
3    | H-30 concrete             | 1,200    | m³   | 180.00     | 1.10   | 237,600
```

### Linking Worksheet to Cost Sheet:
The Worksheet total can be automatically transferred to the "Original Budget" column of the Cost Sheet via a rollup rule configured in the design. This preserves the cost-build detail while populating the master control view.

---

## 5. Budget Changes

### What Is It?
The Budget Changes module records all **formal modifications to the approved budget**, maintaining a complete audit trail from the original baseline.

### Golden Rule:
> Never modify the "Original Budget" column directly once the project has formally started. All changes must go through a Budget Change process to preserve audit traceability.

### Types of Budget Changes:

| Type | Description | Cost Sheet Impact |
|------|-------------|-------------------|
| **Budget Transfer** | Moves funds between CBS codes without changing the total | +/− in "Budget Transfers" column |
| **Budget Increase** | Increases the total project budget | + in "Approved Changes" column |
| **Budget Decrease** | Reduces the total project budget | − in "Approved Changes" column |
| **Contingency Release** | Releases contingency funds to an operational CBS | Moves amount from contingency CBS to target CBS |
| **Scope Addition** | New scope not in original budget | + in "Approved Changes" column, new CBS row if needed |

### Typical Budget Change Workflow:
```
Request (initiator)
  → Review by Cost Control
     → Approval by PM / Management (threshold-based routing)
        → Posted to Cost Sheet (Approved Budget column updated)
           → Audit trail maintained in Budget History
```

---

## 6. Commitments

### What Is It?
**Commitments** represent formal financial obligations of the project: contracts, purchase orders, and subcontracts. They are Cost-type BPs that impact the "Committed Cost" column of the Cost Sheet.

### Full Commitment Lifecycle:

```
RFP / Tender
      ↓
Award Decision → [Award BP]
      ↓
Contract / PO  → [Commitment BP]     ← posts to Cost Sheet: Committed Cost
      ↓
Change Orders  → [Change Order BP]   ← posts to Cost Sheet: Pending / Approved Changes
      ↓
Certifications → [Invoice BP]        ← posts to Cost Sheet: Invoiced
      ↓
Payments       → [Payment BP]        ← posts to Cost Sheet: Actual Cost
      ↓
Closeout       → [Closeout BP]       ← closes Commitment; triggers final reconciliation
```

### Critical Fields in a Commitment BP:

| Field | Description | Accounting Mapping |
|-------|-------------|-------------------|
| `contract_amount` | Original contract value | Committed Cost |
| `approved_changes` | Sum of approved change orders | Approved Changes |
| `total_commitment` | Contract + Approved Changes | Total Commitment |
| `invoiced_to_date` | Cumulative invoiced amount | Invoiced |
| `paid_to_date` | Cumulative paid amount | Actual Cost |
| `retainage_amount` | Accumulated retention held | Retainage (separate column) |
| `remaining_commitment` | Balance to be executed | Formula: Total Commitment − Paid |
| `cbs_code` | CBS allocation code | Cost Sheet row |
| `vendor_id` | Contractor / Supplier | ERP Vendor Master reference |

### Retainage (Retention) Control:

```
Retainage Held     = Invoice Amount × Retainage Rate (%)
Retainage Released = Per milestone or physical progress threshold
Retainage Balance  = Cumulative Held − Cumulative Released
Net Payment Due    = Invoice Amount − Retainage Held
```

> **Note:** Retainage tracking requires a dedicated "Retainage" column in the Cost Sheet and corresponding field mapping in the Invoice and Payment BPs.

---

## 7. Spends / Actuals

### What Is It?
Actuals record **costs that have been genuinely incurred and paid**. They originate from payment BPs, direct cost entries, or ERP imports.

### Sources of Actuals in Unifier:

| Source | Description |
|--------|-------------|
| **Payment BP** (approved) | Automatic rollup to Cost Sheet on approval |
| **ERP Integration** | Actuals imported from ERP (journal entries, payroll, AP payments) |
| **Journal Entry BP** | Direct cost adjustments to a CBS code |
| **Timesheet / Labor BP** | Project personnel cost by CBS and period |
| **Manual Entry (Cost Sheet)** | Direct manual posting when no BP workflow applies |

### Cost Recognition Trigger:
The moment a cost appears in "Actual Cost" depends on the BP configuration:

- On **Approved status** of the payment BP
- At a **specific workflow step** (e.g., when Accounting marks the step "Posted to ERP")
- On **ERP integration trigger** (when the ERP confirms the payment was processed)

> **Design decision:** Align the cost recognition trigger with the finance team's accounting policy — committed accounting vs. cash accounting.

---

## 8. Forecast / EAC

### What Is It?
The **Forecast** or **EAC (Estimate at Completion)** is the projected total final cost of the project, combining costs already incurred with the estimated cost to complete the remaining work.

### EAC Calculation Methods:

| Method | Formula | When to Use |
|--------|---------|-------------|
| **Bottom-up** | Actual Cost + Remaining Estimate | When the team re-estimates the remaining scope |
| **Performance-based (CPI)** | Budget at Completion / CPI | When the cost performance trend is reliable |
| **Conservative** | Actual Cost + (Budget − Earned Value) | Baseline conservative projection |
| **Manual override** | Direct entry by project team | When qualitative factors dominate the forecast |

```
CPI (Cost Performance Index)  = Earned Value / Actual Cost
SPI (Schedule Performance Index) = Earned Value / Planned Value
EAC (Performance-based)       = Budget at Completion / CPI
ETC (Estimate to Complete)    = EAC − Actual Cost
VAC (Variance at Completion)  = Budget at Completion − EAC
TCPI                          = (BAC − EV) / (BAC − AC)
```

### Forecast in Unifier:
- Entered manually via a **Forecast BP** (periodic updates by the project team)
- Calculated automatically via **Cost Sheet column formulas** using existing actuals and budget data
- Updated on a **monthly** cadence or at each reporting period close
- The Forecast column in the Cost Sheet can combine BP-fed values and formula overrides

---

## 9. Cash Flow

### What Is It?
**Cash Flow** shows the **time-phased distribution** of project costs — how much is planned to be spent in each period (month, quarter, year). It is essential for treasury planning and contract milestone payment management.

### Cash Flow Curve Types:

| Curve Type | Description |
|-----------|-------------|
| **Budget Curve** | Time distribution of approved budget |
| **Commitment Curve** | When contract obligations are scheduled to be executed |
| **Actual Curve** | Cumulative payments made per period |
| **Forecast Curve** | Projected future spending per period |
| **S-Curve (Baseline)** | Combined planned vs. actual view for progress tracking |

### Distribution Methods:

| Method | Description |
|--------|-------------|
| **Manual** | User inputs amounts per period directly |
| **Uniform** | Distributes equally across all periods |
| **Front-loaded** | Higher spend at the start (inverted S) |
| **Back-loaded** | Higher spend at the end |
| **S-Curve** | Standard project distribution (slow–fast–slow) |
| **From P6 Schedule** | Derives distribution from Oracle P6 activity dates and resource assignments |

### Cash Flow Configuration Steps:

1. Define the **project calendar** (periods: monthly or quarterly)
2. Select the **Cost Sheet columns** to time-phase (Budget, Commitment, Actuals, Forecast)
3. Choose the **distribution method** per column and per CBS
4. Link to **P6 schedule** if using activity-based distribution
5. Activate the Cash Flow module in the **Shell Template**

---

## 10. Cost Manager Configuration — Step by Step

### Phase 1: Design (uDesigner)

```
Step 1 — Define CBS Structure
   └── Create Data Structure with codes, hierarchy, and GL Account mapping

Step 2 — Configure Cost Sheet Template
   └── Define columns: type (BP Source, Manual, Formula, Rollup, P6)
   └── Set BP Source triggers (which BP status posts to which column)
   └── Build column formulas (Budget, Variance, %, ETC)

Step 3 — Design Cost BPs
   └── Budget BP, Commitment BP, Change Order BP, Invoice BP, Payment BP
   └── Configure Cost Attributes on each BP (CBS mapping, transaction type)
   └── Set workflow trigger step for Cost Sheet posting

Step 4 — Configure SBS (if applicable)
   └── Define categories and assign CBS codes to each

Step 5 — Configure Cash Flow (if applicable)
   └── Define periods, select columns to distribute, set methods

Step 6 — Publish all objects
```

### Phase 2: Shell Deployment

```
Step 1  — Enable Cost Manager in the Shell
Step 2  — Import or create the project CBS
Step 3  — Open and verify the Cost Sheet (rows and columns)
Step 4  — Configure Budget columns (manual entry or Budget BP)
Step 5  — Assign Cost Manager permissions by role
Step 6  — Load initial budget (manual or via Budget BP)
Step 7  — Create a test Commitment BP and verify Cost Sheet posting
Step 8  — Test ERP integration with a sample record
Step 9  — Verify SBS and Cash Flow are displaying correctly
Step 10 — Document configuration and obtain sign-off
```

---

## 11. Cost Manager Permissions

### Permission Levels:

| Permission | Description | Typical Role |
|-----------|-------------|--------------|
| **View Cost Sheet** | Read-only access to the cost sheet | All project members |
| **Edit Cost Sheet** | Edit manual-entry columns | Cost Control, PMO |
| **View Commitments** | View contracts and POs | Project Management |
| **Create/Edit Commitments** | Create commitment BPs | Procurement, Contracts |
| **View Actuals** | View actual costs | Project Management |
| **Edit Actuals** | Create payment/spend BPs | Project Accounting |
| **View/Edit Budget** | View and modify budget | PMO, Cost Control |
| **Approve Budget Changes** | Approve budget modifications | Senior Management |
| **View Cash Flow** | View time-phased projections | Management, Finance |
| **Edit Cash Flow** | Modify period distribution | Cost Control |
| **View SBS** | View summary budget sheet | Management, Executives |
| **Recalculate Cost Sheet** | Trigger manual recalculation | Cost Control, Admin |

---

## 12. ERP Integration

### Cost Event Flow:

| Unifier Event | ERP Result |
|---------------|-----------|
| Budget BP approved | Budget record created in Project module |
| Commitment BP approved | PO / Contract created in Procurement |
| Change Order approved | PO modification in ERP |
| Invoice BP approved | Vendor invoice posted in AP / Asset module |
| Payment BP approved | Payment processed in Treasury / Banking |
| Closeout BP approved | Asset capitalization in Fixed Assets module |

### Primary Correlation Fields:

| Field in Unifier | Field in ERP | Purpose |
|-----------------|-------------|---------|
| CBS Code | Cost Center / WBS Element | Cost allocation |
| Shell ID / Project Code | Project Number | Project identifier |
| Vendor ID | Vendor Master Number | Supplier reference |
| PO Number | Purchase Order Number | Commitment correlation |
| Invoice Number | Invoice Document Number | AP document correlation |
| GL Account | G/L Account | Chart of accounts posting |
| Period / Fiscal Year | Accounting Period | Temporal posting alignment |

### Integration Error Handling:
- Always log both the Unifier record ID and the ERP document number in a dedicated **integration log BP** for traceability.
- Configure **duplicate prevention rules** in the integration layer — the most common issue is a BP being processed twice due to a retry, creating a duplicate posting in the ERP.
- Define **rollback procedures**: if an ERP posting fails, the Unifier record should not advance to "Posted" status until the ERP confirms success.

---

## 13. Reporting

### Standard Reports Available:

| Report | Description | Audience |
|--------|-------------|----------|
| **Cost Sheet Summary** | Full budget vs. actuals view | PMO, Cost Control |
| **Variance Report** | Deviation analysis by CBS | Project Management |
| **Commitment Status** | Contracts and POs by status | Procurement, Contracts |
| **Invoice Log** | Invoices by period and CBS | Project Accounting |
| **Cash Flow Report** | Planned vs. actual cash flow | Finance, Executives |
| **Budget History** | Audit trail of all budget changes | Audit, PMO |
| **Forecast Report** | EAC vs. Approved Budget | Project Management |
| **Retainage Report** | Retention held, released, and balance | Contracts, Finance |
| **Over-Commitment Alert** | CBS codes where Committed > Approved Budget | Cost Control |

### Useful Data View Structure (Commitment Status):

```
Source:   Commitment BP (all instances in shell)
Fields:   CBS Code, CBS Description, Contract Number, Vendor,
          Contract Amount, Approved Changes, Total Commitment,
          Invoiced to Date, Paid to Date, Remaining Commitment,
          Retainage Balance, Status
Filters:  Shell = [current project], Status ≠ Void
Group by: CBS Level 1
Order:    CBS Code ASC
```

---

## 14. Troubleshooting — Quick Reference

| Problem | Root Cause | Diagnostic Steps | Resolution |
|---------|-----------|-----------------|------------|
| Approved BP not appearing in Cost Sheet | Cost Attributes not configured on the BP | Check Cost Attributes tab on the record | Configure CBS + target column in BP Cost Attributes |
| Column shows 0 despite approved BPs | Column not configured to receive that BP | Review column configuration in uDesigner | Add BP as a source for the column |
| Cost Sheet appears stale / out of date | Recalculation not triggered | — | Actions → Recalculate Cost Sheet |
| SBS not consolidating all CBS codes | CBS not assigned to any SBS category | Review SBS configuration → categories | Assign missing CBS to the correct category |
| Budget Transfer not reflected | "Budget Transfers" column missing or misconfigured | Check column exists and has correct BP source | Create or correct the Budget Transfers column |
| Duplicate commitment amount | BP processed twice or integration retry loop | Check BP log and integration log for duplicates | Void duplicate record; fix deduplication rule in integration |
| Inactive CBS blocking new BP creation | CBS deactivated incorrectly | Review CBS status in Cost Manager setup | Reactivate CBS if appropriate; or redirect to an active code |
| Variance sign is reversed | Formula operand order is incorrect | Check formula: Budget − EAC or EAC − Budget? | Correct formula in uDesigner |
| Cash Flow shows no data | Module not activated or no distribution loaded | Verify Shell Template + Cash Flow setup | Activate module and load period distribution |
| Program-level rollup incorrect | CBS in sub-shell not present in program | Compare CBS between child and parent shells | Add missing CBS to parent or fix rollup mapping |
| Retainage column blank | Retainage not mapped in Invoice or Payment BP | Check BP Cost Attributes for retainage field mapping | Add retainage field mapping to the BP |
| EAC = 0 on active CBS | Forecast BP not submitted or formula not evaluating | Check Forecast BP status and column formula | Submit Forecast BP or fix EAC formula |

---

## 15. Configuration Checklist

### For New Projects:

```
DESIGN (uDesigner):
  [ ] CBS structure defined and approved by the client
  [ ] Cost Sheet columns designed: types, sources, formulas
  [ ] All Cost BPs have Cost Attributes configured (CBS mapping + transaction type)
  [ ] SBS categories defined with CBS assignments
  [ ] Cash Flow configured (if applicable)
  [ ] ERP integration: CBS → GL Account mapping complete
  [ ] Retainage columns and BP field mappings defined (if applicable)

SHELL DEPLOYMENT:
  [ ] Cost Manager enabled in the Shell
  [ ] CBS imported or manually created in the Shell
  [ ] Cost Sheet verified: correct rows and columns
  [ ] Initial budget loaded and reconciled
  [ ] Cost Manager permissions configured by role
  [ ] Test Commitment BP processed and verified in Cost Sheet
  [ ] ERP integration tested with a sample record end-to-end
  [ ] Cash Flow baseline loaded (if applicable)
  [ ] SBS reviewed and totals validated

VALIDATION:
  [ ] Cost Sheet reconciled with formally approved initial budget
  [ ] SBS totals match Cost Sheet totals
  [ ] All roles can view/edit per the permissions matrix
  [ ] Variance report producing correct output
  [ ] Budget History showing baseline and all changes
  [ ] Integration log operational and capturing all events
  [ ] Configuration documentation updated and signed off
```