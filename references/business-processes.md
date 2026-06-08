# Business Processes (BPs) — Complete Reference for Oracle Unifier

## Table of Contents
1. [BP Classification](#bp-classification)
2. [BP Configuration in uDesigner](#bp-configuration-in-udesigner)
3. [Upper Form vs. Detail Form](#upper-form-vs-detail-form)
4. [Data Elements and Field Types](#data-elements-and-field-types)
5. [Workflow Configuration](#workflow-configuration)
6. [Cost BPs](#cost-bps)
7. [Document BPs](#document-bps)
8. [Line Item BPs](#line-item-bps)
9. [BP Relationships and Hierarchy](#bp-relationships-and-hierarchy)
10. [Common Use Cases by Industry](#common-use-cases-by-industry)
11. [Design Guidelines](#design-guidelines)

---

## BP Classification {#bp-classification}

### By Data Type:

| Type | Description | Typical Example |
|------|-------------|-----------------|
| **Simple** | Single-form BP with no line items | Information Request, Meeting Minutes |
| **Line Item** | Upper form + detail grid with multiple rows | Material Requisition, Punch List |
| **Cost** | Impacts the project Cost Sheet via CBS codes | Contract, Change Order, Invoice |
| **Document** | Manages file attachments with revision control | Transmittal, Submittal, RFI |
| **Text** | Structured notes and log entries | Daily Report, Site Diary |
| **Asset** | Tracks physical assets or equipment | Equipment Register, Asset Inspection |
| **Resource Booking** | Manages resource allocation | Labor Assignment |

### By Workflow Type:

| Type | Description | When to Use |
|------|-------------|-------------|
| **Workflow BP** | Record moves through defined approval steps | Any process requiring review, approval, or multi-party action |
| **Non-Workflow BP** | Creator saves and submits directly | Logs, registers, or data capture without approval chain |

---

## BP Configuration in uDesigner {#bp-configuration-in-udesigner}

### Steps to Create a New BP:

1. Open **uDesigner** → **Company Workspace** → **Business Processes**
2. Click **New** → Select the BP type (Simple, Line Item, Cost, Document, etc.)
3. Set **General Properties**: name, description, BP prefix/number format
4. Design the **Upper Form** (main header fields)
5. Design the **Detail Form** or **Line Item grid** (if applicable)
6. Configure **Workflow** steps, actions, and routing conditions (if Workflow BP)
7. Define base **Permissions** (who can create, view, edit)
8. Configure **Notifications** (email alerts per step)
9. Click **Save** and **Publish** to make the BP available in shells

> **Note:** A BP design can also be edited in **Published** state. Advanced Formulas and auto-populate settings are available for published designs without needing to unpublish first.

### General BP Properties:

| Property | Description |
|----------|-------------|
| **BP Number Format** | Auto-generated record identifier (e.g., `CONT-{YYYY}-{NNNN}`) |
| **Auto-Number by Shell** | When enabled, the sequence resets per shell |
| **Record Status** | Configurable statuses: Open, Closed, Cancelled, Void, etc. |
| **Record Locking** | Prevents edits once a record reaches a defined status or workflow step |
| **Allow Multiple Drafts** | Controls whether a user can have more than one in-progress record |
| **Help Text** | Field-level or form-level guidance shown to end users at runtime |

---

## Upper Form vs. Detail Form {#upper-form-vs-detail-form}

### Upper Form:
The main header section of a BP record. Contains fields that describe the record as a whole — dates, parties, summary amounts, status fields, and identifiers.

- Supports all data element types (string, numeric, date, pulldown, pickers, etc.)
- Advanced Formula fields in the upper form re-evaluate on any field change
- Auto-populate runs once on record creation (or on trigger field change)

### Detail Form (Line Items):
The repeating rows section, also called the **line item grid**. Each row is an individual entry linked to the parent record.

- Configured separately under **Line Item** or **Detail Form** in uDesigner
- Can have its own set of fields, independent of the upper form
- Supports SUM rollups from line items to upper form fields (e.g., `SUM(lineitems.amount_d)`)
- Line item logs can be searched and filtered independently in the BP log view

---

## Data Elements and Field Types {#data-elements-and-field-types}

### Core Data Element Types:

| Type | Stores | Example Use |
|------|--------|-------------|
| **String** | Free text | Description, justification, reference number |
| **Integer Amount** | Whole numbers | Quantity, number of days, headcount |
| **Decimal Amount** | Decimal numbers | Contract amount, percentage complete |
| **Currency Amount** | Monetary values with currency symbol | Invoice amount, budget allocation |
| **Date** (Date-Only Picker) | Calendar date, no time | Start date, expiry date |
| **Date** (Date Picker) | Date and time, timezone-aware | Submission timestamp, approval time |
| **Checkbox** | Boolean (Yes/No) | Requires tender? Urgent? |
| **Pull-down Menu** | Single selection from a fixed list | Status, priority, contract type |
| **Radio Button** | Single selection (visible options) | Approval decision |
| **Multi-Select Input** | Multiple selections from a list | Disciplines affected |
| **Dynamic Data Set (DDS)** | Selection from a BP or Shell data source | Contractor code, CBS from master list |
| **User/Group Picker** | Unifier user or group | Assigned approver, reviewer |
| **BP Picker** | Links to another BP record | Parent contract reference |
| **Cost Code (CBS Picker)** | CBS code from the Cost Breakdown Structure | Cost allocation code |
| **Rich Text** | Formatted text with HTML | Technical specification, comments |
| **Hyperlink** | URL or document reference | Link to external document |

### Naming Conventions (Recommended Suffixes):

```
_d   = Decimal amount
_i   = Integer amount
_c   = Currency amount
_s   = String / text
_dt  = Date or DateTime
_b   = Boolean / checkbox
_pk  = Pull-down (picklist key)
_u   = User picker
_bp  = BP picker reference
```

> **Tip:** Consistent naming conventions across all BPs in a project significantly reduce configuration time, improve formula readability, and simplify integrations via REST API.

---

## Workflow Configuration {#workflow-configuration}

### Workflow Anatomy:

A Workflow BP moves through a defined sequence of **steps**. Each step has:

| Element | Description |
|---------|-------------|
| **Step Name** | Label visible to users (e.g., "Pending Approval", "Under Review") |
| **Step Type** | Creation, Approval, Review, Revision, End |
| **Actions** | Buttons available at that step (e.g., Submit, Approve, Return, Reject) |
| **Assignees** | Users or groups who can act on the record at this step |
| **Due Date** | Optional SLA deadline per step |
| **Routing Conditions** | Logic that determines the next step based on field values |
| **Notifications** | Automated emails triggered when a record enters or exits the step |

### Routing Conditions:

Routing conditions use field values to direct the record to different next steps. Always define a **default** (otherwise) condition.

```
Field: [contract_amount_d]

Condition 1: contract_amount_d > 1,000,000  → Step "Board Approval"
Condition 2: contract_amount_d > 100,000    → Step "Management Approval"
Condition 3: (default)                      → Step "Department Head Approval"
```

Supported operators: `=`, `!=`, `>`, `>=`, `<`, `<=`, `IN`, `IS NULL`, `CONTAINS`

Compound conditions: `(A AND B) OR C`

### Workflow Best Practices:
- Always configure a **substitute or backup assignee** for every approval step to avoid bottlenecks.
- Define what happens on **Reject**: does the record return to the creator, go to a revision step, or terminate?
- Keep the total number of steps to **7 or fewer** — complex workflows become unmaintainable.
- Use **parallel steps** when multiple reviewers must act simultaneously (e.g., Legal + Finance review).
- Test every routing path before go-live, including edge cases (null amounts, missing fields).
- Configure **email notifications** on every step transition for process visibility.

---

## Cost BPs {#cost-bps}

Cost BPs are the most critical BP type in Unifier because they directly impact the project **Cost Sheet** via CBS (Cost Breakdown Structure) codes.

### Cost Transaction Types:

| Transaction Type | Cost Sheet Impact | Typical BP |
|-----------------|-------------------|------------|
| **Commit** | Increases Commitments column | Contract, Purchase Order |
| **Commit Change** | Adjusts existing Commitment | Change Order, Amendment |
| **Spend / Actual** | Increases Actuals / Spent column | Invoice, Payment Application |
| **Budget Change** | Adjusts the Approved Budget column | Budget Transfer, Contingency Release |
| **Forecast (EAC)** | Updates Estimate at Completion | Forecast Update |

### Key Configuration for Cost BPs:

**1. CBS Source** — How the cost code is selected:
- User selects from CBS picker on the form
- Inherited from the parent Shell
- Inherited from a parent BP record (e.g., Change Order inherits CBS from its Contract)

**2. Cost Sheet Column Mapping** — Which column is impacted and when:
```
BP: Purchase Order (Commit type)
  → Impacts "Commitments" column
  → Trigger: When record reaches "Approved" status in Workflow

BP: Invoice (Spend type)
  → Impacts "Actuals" column
  → Trigger: When record reaches "Approved" status
  → Also reduces "Commitments" by the invoiced amount (if linked to a PO)
```

**3. Rollup Rules** — How line item amounts aggregate to the Cost Sheet:
```
Purchase Order — Line Items:
  Line 1: CBS=01.02.001, Amount=$50,000 → Commitments
  Line 2: CBS=01.02.003, Amount=$20,000 → Commitments

Result in Cost Sheet:
  CBS 01.02.001: Commitments += $50,000
  CBS 01.02.003: Commitments += $20,000
```

**4. SOV (Schedule of Values)** — Used in Payment Application BPs:
- Links invoiced amounts to contract line items
- Tracks cumulative billed, retained, and net-due amounts per CBS
- Summary Payment Applications (SPA) do not support Advanced Formulas

### Cost BP Design Checklist:
- [ ] Cost transaction type defined (Commit / Spend / Budget / Forecast)
- [ ] CBS source configured and tested
- [ ] Column mapping verified in the Cost Sheet
- [ ] Trigger step in Workflow identified (which step posts to Cost Sheet)
- [ ] Rollup rules configured for line items
- [ ] Behavior on Void or Cancel defined (reversal of posted amounts)

---

## Document BPs {#document-bps}

Document BPs manage the controlled exchange and archival of project files within Unifier's **Document Manager (DM)**.

### Special Configuration for Document BPs:

| Setting | Description |
|---------|-------------|
| **Document Manager Integration** | Attachments are saved to a defined DM folder, not just the BP record |
| **Revision Control** | Enables versioning — each new submission creates a new revision |
| **Distribution List** | Automatically notifies a defined list of recipients on submission |
| **Response Required** | Recipient must formally acknowledge or respond to the document |
| **Due Date Tracking** | SLA tracking for response or review turnaround |

### Typical Transmittal Workflow:
```
Contractor creates Transmittal
  → Attaches documents (drawings, specs, reports)
  → Submits to Owner/Supervision
     → Reviewer opens, reviews attachments
        → Returns with status: Approved / Approved with Comments / Rejected
           → Contractor receives response and comments
              → Transmittal is closed
```

### Common Document BP Types:

| BP Type | Purpose |
|---------|---------|
| **Transmittal** | Formal document delivery between parties |
| **Submittal** | Contractor submits items for Owner review and approval |
| **RFI (Request for Information)** | Field question requiring a technical or contractual answer |
| **Drawing Register** | Master log of all project drawings and their current revision status |
| **Correspondence** | Formal letters or official written communications |

---

## Line Item BPs {#line-item-bps}

Line Item BPs combine an **upper form** (header) with a **detail grid** (repeating rows). They are used when a single BP record needs to track multiple discrete entries.

### Line Item Grid Configuration:

- Each column in the grid corresponds to a data element defined in the **Detail Form**.
- Columns can be **editable**, **read-only**, or **formula-calculated**.
- The grid can have its own validation rules (e.g., quantities must be > 0).
- **SUM rollups** from line items to upper form fields:
```
Upper form field:
  total_amount_d = SUM(lineitems.line_amount_d)
```

### Behavior in Workflow:
- Line items can be **locked** at certain workflow steps (e.g., once approved, lines cannot be edited).
- Individual line items can have their own **status** field, independent of the record status.
- Line Item logs can be configured as a separate view in the shell navigation.

---

## BP Relationships and Hierarchy {#bp-relationships-and-hierarchy}

Unifier supports several types of relationships between BPs:

### Parent-Child BP:
A child BP is created from within a parent BP record. The child inherits fields from the parent.

```
Contract (Parent)
  └── Change Order (Child)
        └── Inherits: contract_number_s, vendor_s, project_code_s
```

### Linked BP (Cross-Reference):
A BP record references another BP without a strict parent-child relationship.

```
RFI → linked to → Drawing (in Drawing Register)
Invoice → linked to → Purchase Order
```

### Shell-Level Inheritance:
BP records automatically inherit shell-level attributes (project code, shell number, company) into designated fields via auto-populate or DDS.

### Rollup to SBS/Cost Sheet:
Cost BPs roll their amounts up through the CBS hierarchy to the **Cost Sheet**, which in turn rolls up to the **Summary Budget Sheet (SBS)** at parent shell levels.

```
Contract BP (record)
  → posts to Cost Sheet (shell level)
     → rolls up to SBS (program level)
        → rolls up to Company SBS
```

---

## Common Use Cases by Industry {#common-use-cases-by-industry}

### Capital Projects / Infrastructure:

| BP | Purpose |
|----|---------|
| Scope Change Request | Documents and approves changes to project scope |
| Contract Change Order | Adjusts contract value or schedule |
| RFI Management | Technical queries raised in the field |
| Submittal / Transmittal | Delivery and control of project deliverables |
| Progress Measurement | Monthly physical progress reporting |
| Payment Application | Contractor payment requests and certifications |
| Non-Conformance Report (NCR) | Defects, deviations, and quality incidents |
| Lessons Learned | Post-execution knowledge capture |
| Risk Register | Active project risk tracking and mitigation |
| Punch List | Outstanding items before project handover |

### Contract Management:

| BP | Purpose |
|----|---------|
| Master Contract | Core contract record with value, parties, and key dates |
| Addendum / Amendment | Formal modification to the contract |
| Performance Bond / Guarantee | Tracking of bonds, insurance, and policy expiry |
| Substantial Completion Certificate | Formal milestone signoff |
| Final Acceptance Record | Contract close-out and acceptance |

### Facilities / Operations:

| BP | Purpose |
|----|---------|
| Work Order | Maintenance or repair task |
| Asset Inspection | Periodic condition reporting on assets |
| Preventive Maintenance Log | Scheduled maintenance tracking |
| Service Request | Internal or tenant-raised service needs |

---

## Design Guidelines {#design-guidelines}

###  Best Practices:

- A BP should have **one clear purpose** — avoid combining two distinct processes in a single BP.
- **Reuse existing Data Elements** across BPs before creating new ones; shared DEs reduce maintenance overhead and enable cross-BP reporting.
- Name fields so users understand them **without a manual** — use plain language, not system codes.
- Keep **mandatory fields to the minimum** required for process integrity; excessive required fields increase rejection rates and user workarounds.
- Use **field sections (groups)** to organize the form logically — header info, financial data, dates, attachments.
- Always configure an **auto-number format** for every BP; manual numbering creates duplication and traceability gaps.
- Define **record statuses** that align with the business process states, not just technical states.
- Set up **email notifications** on every workflow step transition; silent workflows lead to bottlenecks.
- Design for the **mobile app user**: minimize required fields on creation, use dropdowns over free text where possible.
- Always complete **UAT (User Acceptance Testing)** across all workflow paths before publishing to production.

###  Common Mistakes:

- **Circular formula references**: Field A depends on Field B which depends on Field A — Unifier will block saving if detected, but always verify the dependency graph.
- **No null handling in formulas**: Any formula dividing by a field that could be zero or null will fail at runtime — always guard with `IF(field != null AND field != 0, ...)`.
- **Single approver with no substitute**: If the assigned user is unavailable, the workflow is blocked indefinitely.
- **Not testing all routing paths**: Only testing the "happy path" (approve) leaves reject, return, and escalation paths broken at go-live.
- **Overloading a BP with too many fields**: More than ~40 fields on a single form degrades usability; split into sections or separate BPs if needed.
- **Missing Cancel/Void behavior on Cost BPs**: If a Cost BP is voided after it has posted to the Cost Sheet, the reversal logic must be explicitly configured, otherwise the Cost Sheet retains incorrect amounts.
- **Publishing without full permission configuration**: A BP published with default permissions may be visible or editable to unintended users or shells.
- **Confusing Auto-populate with Advanced Formula**: Auto-populate fires **once** at record creation; Advanced Formula re-evaluates **every time** a source field changes. Using the wrong one causes data inconsistencies.
- **Mixing process ownership in one BP**: A BP that handles both the scope change request and the change order approval combines two distinct processes with different owners, timelines, and reporting needs — keep them separate.