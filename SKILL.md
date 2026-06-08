---
name: primavera-unifier
description: >
  Specialized skill for functional analysts and technical consultants of Oracle Primavera Unifier.
  Activate ALWAYS when the user mentions Unifier, Primavera, uDesigner, Business Processes (BPs),
  Workflows, Cost Manager, CBS, SBS, Cost Sheets, Cash Flow, Schedule Manager, Document Manager,
  Gates, Data Elements, Data Views, UDRs, Shells, auto-populate, formulas, rollups, permissions,
  capital project management configuration, or any design/configuration/documentation task
  within the Primavera Unifier ecosystem. Also for: integration with Oracle P6, field-level
  permissions, record-level security, BP publication lifecycle, UAT checklists,
  or any mention of "functional requirements" in the context of project controls or capital projects.
  If the user asks about "processes", "modules", "templates", "custom fields", "roles",
  or "configuration" in the context of capital project management or infrastructure — activate
  this skill immediately.
---

# Oracle Primavera Unifier — Functional Analyst Skill

## Skill Structure

```
SKILL.md                          <- This file (skill definition)
logic/
  └── triage.md                   <- Initial triage logic (diagnostic questions)
references/
  ├── business-processes.md       <- BPs: types, configuration, use cases
  ├── cost-manager.md             <- CBS, Cost Sheet, SBS, Forecast
  ├── form-design.md              <- Form design (Upper, Detail, Action/View)
  ├── formulas-and-logic.md       <- Formulas, auto-populate, validations, SBS
  └── workflow-design.md          <- Advanced workflow design
docs/
  ├── Adding Advanced Formulas.pdf
  ├── Advanced Formulas in Business Processes.pdf
  ├── Functions and Available Formats.pdf
  ├── Rules for Setting Fields as a Formula Destination.pdf
  └── Setting Time Zone in Advanced Formula.pdf
```

## Execution Flow

Each time this skill is activated, follow this order:

### 1. Triage (pre-processing)
Run `logic/triage.md` first to diagnose the user's case through initial questions. This ensures more accurate responses and avoids ambiguity.

### 2. Response with references
Once the module and context are identified, use the references in `references/` to structure the response.

---

## Role and Responsibilities

The Unifier functional analyst is the bridge between business requirements and platform configuration:

- **Design and configure Business Processes (BPs)**
- **Develop formulas** (Data Elements, Auto-populate, Conditional logic)
- **Gather and document functional requirements**
- **Design Workflows** (approval flows and routing logic)
- **Configure Shells, CBS, SBS, Cost Sheets**
- **Manage permissions, roles, and access control**
- **Integrate with Oracle P6 and external systems**
- **Perform functional testing and configuration validation**

---

## Object Hierarchy in Unifier

```
Company Workspace
  └── Shell (Program / Project / Asset)
        ├── Business Processes (BPs)
        ├── Cost Manager (CBS, Sheets, SBS, Forecast)
        ├── Schedule Manager
        ├── Document Manager
        ├── Cash Flow
        └── Gates / Milestones
```

**Fundamental rule:** Every object is first designed in **uDesigner**, published, and then deployed to production shells.

---

## uDesigner — Configuration Center

| Object | Description |
|--------|-------------|
| **Business Process** | Transactional forms for business operations |
| **Shell** | Hierarchical container for projects/programs |
| **Data Definitions** | Reusable field types across the platform |
| **Data Structure** | CBS/WBS structure definitions |
| **Cost Manager** | Budget and cost sheet configuration |
| **Gate Process** | Lifecycle controls |
| **Dashboard** | KPI portals and indicator views |

---

## Business Processes (BPs)

Full reference -> `references/business-processes.md`

### BP Types by Function

| Type | Typical Use |
|------|-------------|
| **Cost BP** | Commitments, change orders, payments |
| **Document BP** | Transmittals, RFIs, submittals |
| **Line Item BP** | Tabular items (materials, resources) |
| **Simple BP** | Flat forms without line items |
| **Workflow (WF) BP** | Any BP with an approval flow |
| **Non-WF BP** | Direct records without required approval |

### BP Anatomy

1. **Upper Form** — Header fields
2. **Detail Form / Line Items** — Row-level data
3. **Workflow** — Steps, actors, routing conditions
4. **Permissions** — Who can create/view/edit
5. **Notifications** — Email alerts per step/action
6. **Integration** — Mapping rules for P6 or external systems

---

## Forms (Upper / Detail / Action / View)

Full reference -> `references/form-design.md`

| Type | Purpose |
|------|---------|
| **Upper Form** | Form header with main information |
| **Detail Form** | Line item grid with detailed data |
| **Action Form** | Editable version when the user accepts the task |
| **View Form** | Read-only version |
| **Template** | Reusable field layout for upper forms |

---

## Formulas and Data Logic

Full reference -> `references/formulas-and-logic.md`

| Type | Description | When to use |
|------|-------------|-------------|
| **Auto-populate** | Fills field once from external source | On record creation |
| **Advanced Formula** | Calculation with if-then-else, string, date | Re-evaluated always |
| **Conditional Routing** | Dynamic routing in workflow | Based on field value |
| **Validation Rule** | Validates data before saving | Data integrity |
| **Rollup / Aggregation** | Aggregates values from child BPs to Cost Sheet | Commitments, forecast |

---

## Workflows

Full reference -> `references/workflow-design.md`

### Common Patterns

```
Simple sequential:
  Initiator -> Reviewer -> Approver -> Closed

Conditional branch:
  Initiator -> Reviewer
               ├─ [Amount > threshold] -> Senior Approver -> Closed
               └─ [Amount <= threshold] -> Junior Approver -> Closed

Rejection and revision cycle:
  Initiator -> Approver
               ├─ Approved -> Closed
               └─ Rejected -> Initiator (Revise) -> (restarts)
```

---

## Cost Manager

Full reference -> `references/cost-manager.md`

| Component | Description |
|-----------|-------------|
| **CBS** | Cost Breakdown Structure — project cost codes |
| **Cost Sheet** | Master budget and cost ledger |
| **SBS** | Summary Budget Sheet — consolidated view by category |
| **Worksheet** | Budget calculation tool |
| **Budget Changes** | Modifications to the approved baseline |
| **Spends** | Actual expenditures against CBS codes |
| **Commitments** | Contracts and obligations (PO, subcontracts) |
| **Forecast** | Estimate at Completion (EAC) projection |

---

## Permissions and Security

### Access Control Levels

1. **Company-level** — Access to the company workspace
2. **Shell-level** — Access to a specific project/program
3. **BP-level** — Who creates/views/edits records in a BP
4. **Record-level** — Access to individual records (based on creator, WF step)
5. **Field-level** — Editable vs. read-only by role

### Best Practices
- Assign permissions to **roles**, not individual users
- Use **Groups** to assign multiple users to the same role
- Document the permissions matrix before configuring
- Review permissions after each BP publication

---

## Unifier <-> P6 Integration

| Integration Type | Description |
|-----------------|-------------|
| **Schedule Integration** | Syncs P6 activities with Unifier Schedule Manager |
| **Cost Integration** | Maps Unifier CBS with P6 WBS/Cost Accounts |
| **Resource Integration** | Shares resource data between systems |

**Integration checklist:**
- Define the correlation field (e.g. CBS Code <-> WBS Code)
- Establish flow direction: Unifier -> P6, P6 -> Unifier, or bidirectional
- Document synchronization frequency
- Validate field mapping before activating in production

---

## Data Views and Reports

| Tool | Use |
|------|-----|
| **Data View** | SQL-like queries on BP and module data |
| **UDR** | Configurable reports for end users |
| **Dashboard** | Consolidated KPI view with portlets |
| **Log View** | Record list for a specific BP |
| **Query** | Advanced filters on record lists |

---

## Publication Lifecycle

```
uDesigner (design)
    |
Save draft
    |
Publish to Dev / QA
    |
Functional testing
    |  (if approved)
Publish to Production
    |
Deploy to target Shell(s)
    |
Configure permissions in Shell
    |
Communicate to users
```

### Pre-Publication Checklist
- [ ] Form fields reviewed (required, data types, labels)
- [ ] Formulas tested with real data
- [ ] Workflow validated for all actor/action paths
- [ ] Permissions correctly configured by role
- [ ] Email notifications reviewed
- [ ] Data mapping validated (if integration exists)
- [ ] Documentation updated
- [ ] UAT completed and signed off

---

## Naming Conventions

| Object | Pattern | Example |
|--------|---------|---------|
| BP | `[Area]-[Name]-BP` | `CONT-Contract-BP` |
| Data Element | `[module]_[name]_[type]` | `cost_budget_amount_d` |
| Shell | `[Type]-[Code]` | `PRJ-2024-001` |
| Workflow | `WF-[BP]-[version]` | `WF-Contract-v2` |
| CBS Code | `[L1].[L2].[L3]` | `01.02.003` |
| UDR | `RPT-[area]-[description]` | `RPT-COST-Variance` |

---

## Skill References

| File | Content |
|------|---------|
| `logic/triage.md` | Initial question logic for case diagnosis |
| `references/business-processes.md` | BP types, configuration, and use cases |
| `references/form-design.md` | Form design (Upper, Detail, Action/View, Templates) |
| `references/formulas-and-logic.md` | Formulas, auto-populate, validations, SBS |
| `references/workflow-design.md` | Advanced workflow design patterns |
| `references/cost-manager.md` | CBS, Cost Sheet, SBS, Forecast |
| `docs/*.pdf` | Official Oracle documentation (Advanced Formulas) |