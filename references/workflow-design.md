# Workflow Design in Oracle Unifier (uDesigner)

Based on official Oracle documentation:
https://docs.oracle.com/cd/F74686_01/help/uDesigner/en/77757.htm

## Table of Contents
1. [Fundamental Concepts](#concepts)
2. [General Design Steps](#general-steps)
3. [Workflow Elements](#elements)
4. [Creating and Starting a Workflow](#create-workflow)
5. [Step Properties](#step-properties)
6. [Linking Steps (Actions)](#linking)
7. [Conditional Steps](#conditional)
8. [Sub-Workflows](#subworkflows)
9. [Auto-Creation of Records](#auto-create)
10. [Validation (Error-Checking)](#error-checking)
11. [Workflow Summary](#summary)
12. [Step Order (Configuration in Unifier)](#step-order)
13. [Print Workflow to PDF](#print)
14. [Copy a Workflow](#copy)
15. [Official Best Practices](#best-practices)
16. [Design Patterns](#patterns)
17. [Advanced Configuration and SLAs](#advanced)
18. [Design Checklist](#checklist)

---

## 1. Fundamental Concepts {#concepts}

A Workflow in Unifier illustrates every step of a Business Process (BP) from start to finish, specifying how forms are routed and the behavior at each step.

### uDesigner vs. Unifier Administrator
Understanding the separation of responsibilities is critical. In **uDesigner** you create the workflow schemas: design the step flow, define actions (links), and determine which forms are used at each step. In **Unifier (Administration)**, the administrator takes that schema and configures the "who, what, where, and when" by assigning real actors, SLA times, and notifications.

### Non-workflow BPs
Workflows are entirely optional. Some BPs exist solely to store data (e.g. a company directory). These "Non-workflow BPs" remain editable after completion unless a "terminal status" is assigned to them.

---

## 2. General Design Steps {#general-steps}

Design the workflow on paper **before** moving to uDesigner. The recommended order is:

1. **Name the workflow.**
2. **Add steps** to the canvas.
3. **Specify properties for each step:** name, description, View Forms (read-only) and Action Forms (required response).
4. **Link the steps (Actions):** define the action (Submit, Reject, etc.) and the post-action Record Status.
5. **Group Sub-workflows** if applicable.
6. **Add Conditional Steps** for automatic routing based on form data.

> **Important:** Oracle recommends NOT deleting a workflow once created in the Development environment — it can cause unexpected results. Instead, deactivate it and republish the BP with a new workflow.

---

## 3. Workflow Elements {#elements}

### Steps

| Type | Description |
|------|-------------|
| **Create (Start)** | Entry point of the BP. Executed by the record creator. Can only have outbound links. Only accepts Action Forms (no View Forms). A link cannot be sent back to the Create Step. |
| **Standard** | Intermediate step for review or approval. Requires an Action Form. |
| **Conditional** | Automatically evaluates form fields for intelligent routing. Appears as a diamond on the canvas. |
| **End** | Closes the record. Only receives inbound links. Cannot be renamed. Accepts Action or View Forms. |

### Create and End Steps — Details

**Create Step (in uDesigner):**
- Can be renamed.
- Only supports outbound links.
- Only supports attached Action Forms.
- A link cannot be sent back to the Create Step. If the workflow requires returning to the creator, add a separate "back step" or "revision step".

**End Step (in uDesigner):**
- Cannot be renamed.
- Only supports inbound links.
- Can have an Action Form or View Form attached.

**Create Step (in Unifier):**
- Users cannot be "Cc'd" on the Create Step.
- If there is a "back step" that returns to the creator, specify "match step <Creation>" for the Assignees of that step.

**End Step (in Unifier):**
- Users can be "Cc'd" on the End Step.
- The form can be sent to assigned editors.
- Comments can be added on the End Step and at any workflow state, including "terminated".

### Actions / Links
Actions are the paths connecting steps. Each action moves the record to the next step and updates the Record Status. Each action generates a potential notification email.

---

## 4. Creating and Starting a Workflow {#create-workflow}

1. Go to the **Company Workspace** tab and switch to **Admin** mode.
2. In the left Navigator, click **uDesigner > Business Processes**.
3. Open the BP for which you want to design a workflow.
4. In the Navigator, click **Workflows**.
5. Choose **New > Manual**. The Workflow Properties window opens.
6. Enter a **Name** and (optionally) a **Description**.
7. Click **OK**. The new workflow appears in the Workflows panel.
8. Select the workflow and click **Open**.

uDesigner opens a canvas with a Create and End Step by default. For large workflows, you can expand the canvas by dragging the window borders or using the maximize icon.

---

## 5. Step Properties {#step-properties}

To specify the properties of a step, double-click it on the canvas.

| Field | Description |
|-------|-------------|
| **Step Name** | Use a noun (e.g. "Customer Approval", "Finance Review"). Max 50 characters. The Create Step can be renamed; the End Step cannot. |
| **Description** | Description of what the step entails. |
| **View Form** | Read-only forms available to the recipient. Not available for the Create Step. |
| **Edit Form** | Action forms that the recipient must complete. |
| **Sub-Workflow Name** | If the step belongs to a sub-workflow, enter the sub-workflow name. |
| **Trigger Element** | (Optional) Form field that will serve as a trigger for a Conditional Step. Multiple data elements can be specified — all must match to activate the condition. A multi-select data element cannot be used as a trigger. |
| **Add additional task assignees from data elements** | Allows adding additional assignees when the record reaches this step, using User Data Pickers or User Pickers from the forms. Not available for Create or End Steps. |
| **Add additional Cc users from data elements** | Allows copying additional users when the record reaches this step. Not available for the Create Step. |

---

## 6. Linking Steps (Actions) {#linking}

Connect steps using the **Line** (straight arrow) or **Elbow** (angled arrow) tool. The Create Step can only have one outbound line; the End Step can have multiple inbound lines, but only one from each individual step.

### To connect steps:
1. Click **Line** or **Elbow**.
2. Hover over a step until a border of red X's (connection points) appears.
3. Drag from the source step to the destination step.

### To modify a line:
- Use the red squares to move the start and end points.
- For elbows, use the yellow diamond to expand the angle distance.

### Action Properties

When double-clicking a connection line:

| Field | Description |
|-------|-------------|
| **Action Name** | Use a verb (e.g. "Send for Review", "Return for Clarification"). Max 40 characters. |
| **Description** | (Optional) Details about the action. Max 4000 characters. |
| **Record Status** | Status of the record after the action (e.g. "Pending", "Approved"). Statuses come from Administration mode > Data Structure. |
| **Select data elements to capture action taken** | Allows "stamping" the form with action information: Taken By (user picker), Action Name (text field), Taken On (date picker). This info also appears in the BP log. |
| **Generate Event Notification** | If checked, Unifier generates a notification for users when this action is taken. |
| **Auto-create other record** | Check this box to have this action automatically create another record (see [Auto-Creation](#auto-create) section). |

### Terminal Status Rules

- The status of any link leading to an End Step **must** be a terminal status.
- If a step has an inbound action with terminal status, **all** outbound actions from that step must have the **same** terminal status.
- A terminal status creates a permanent record that feeds BP logs, line items, and assets with all information generated during the workflow.
- Once a record reaches a terminal status, that status **cannot change**.
- Mixing terminal and non-terminal statuses in the same step produces an error.

---

## 7. Conditional Steps {#conditional}

A Conditional Step (represented as a **diamond**) evaluates form field values to determine the path to follow.

### To add a Conditional Step:
1. Click the **Condition** button.
2. Click on the canvas. A diamond appears.
3. Specify its properties (same as a standard step).
4. In **Trigger Element**, click **Add** and select the field that will serve as the trigger.
5. Connect the Conditional Step: it must have **1 inbound connector** and **2 outbound connectors**.
6. Double-click each outbound line to open **Conditional Properties** and name each condition (e.g. "Under 1 Million", "Over 1 Million").

> **Tip:** If your BP allows users to add line items, be aware they could add them after the Conditional Step, bypassing the routing logic. To prevent this, create an additional Action Form that prevents adding line items and use it in the steps after the conditional step.

You can include multiple Conditional Steps in a workflow, but only one condition per step.

---

## 8. Sub-Workflows {#subworkflows}

A sub-workflow is a grouping of one or more steps into a mini-workflow. It allows assigning configuration properties to a group of steps at once in Unifier.

### To create a sub-workflow:
1. Select the steps to group (Shift or Ctrl + click).
2. Click **Sub-workflow** > **Group**.
3. Enter a **Name** and (optionally) a **Description**.
4. Optionally, assign a **Color** for visual identification.
5. Click **OK**.

### To add or remove steps:
1. Select the sub-workflow.
2. Click **Sub-workflow** > **Ungroup**.
3. Add or remove steps.
4. Re-group with **Sub-workflow** > **Group**.

> **Note:** A sub-workflow cannot be created inside another sub-workflow, nor can parallel sub-workflows be created.

---

## 9. Auto-Creation of Records from a Workflow {#auto-create}

You can design a workflow that automatically creates a new record when the form reaches a specific step. The new record contains an exact copy of the original (forms, attachments, comments). The owner of the original record becomes the owner of the new record.

### To configure auto-creation:
1. Double-click the connection line of a non-conditional step.
2. Check **Auto-create other record**.
3. Click **Add** and select the destination BP.
4. Optionally, choose **Auto Populate** to populate fields in the new record (Upper Form, or Upper + Detail Form from the Standard tab).
5. Click **OK**.

An "S" icon appears on the link to indicate that the action will auto-create a record.

> **Notes:**
> - Only applies to project/shell-level BPs (not company-level or single-record).
> - For multi-tab forms, only the Standard tab (system-defined) is auto-populated.
> - Auto-populated values are shown only when the user accepts the task.
> - If an action will not auto-create a record, check **Do not evaluate condition based autocreation** to avoid unnecessary processing.

---

## 10. Workflow Validation (Error-Checking) {#error-checking}

Before publishing, validate the workflow to verify it is complete and error-free.

1. In the Workflow Designer window, click **Error Check**.
2. If there are errors, a window listing them opens.
3. Fix the errors and click **Error Check** again.
4. Repeat until no error messages appear.

---

## 11. Workflow Summary {#summary}

You can view the workflow as a textual table-format summary:

1. In the Workflow Designer, click **Summary**.
2. The **Workflow Summary** window shows:
   - Step Name
   - View Form
   - Edit Form
   - Action Name
   - Record Status (after action)
   - Event Notification (after action)
   - Condition Name
   - Next Step

A blue circle next to an action name indicates it is a "resolving action". This format helps verify the workflow against your paper design.

---

## 12. Step Order (Facilitating Configuration in Unifier) {#step-order}

When designing, steps appear in the Settings tab of Unifier in the **order they were added to the canvas**, not in chronological order. To facilitate configuration for the administrator:

1. In the Workflow Designer, click **Step Order**.
2. In the **Step and Sub-Workflow Order** window, reorder steps using **Move Up** / **Move Down**.
3. Use the **Step Links** section to order actions by frequency of use (most used first) so users do not have to scroll through the **Workflow Actions** dropdown on the form.
4. Click **OK**.

This only affects presentation; it does not modify the workflow itself.

---

## 13. Print Workflow to PDF {#print}

1. In the Workflow Designer, go to **File** > **Print to PDF**.
2. Choose **Save** (save to local system) or **Open** (open as PDF).
3. From the PDF, go to **File > Print**.

---

## 14. Copy a Workflow {#copy}

You can copy an existing workflow to create a new one:

1. Go to **uDesigner > Business Processes** and select the BP.
2. Click **Workflows** to open the Workflow log.
3. From the menu, select **Copy > Copy From**.
4. In the **Business Process Workflows** window, find and select the workflow to copy.
5. Click **Copy**.
6. In **Workflow Properties**, enter the new **Name** and (optionally) **Description**.

### What is copied:
- All steps and their names.
- All action links.
- All sub-workflows.
- Auto-creation configuration (S-Step) and the associated BP.
- Step Name, Description, View Form, Edit Form.
- Trigger Elements and action capture fields (Taken By, Action Name, Taken On) — **only if the Data Elements exist in the destination BP**.

### What is NOT copied:
- The action forms and view forms associated with the workflow (must be manually re-assigned in each step of the destination BP).

---

## 15. Official Best Practices {#best-practices}

1. **Create forms first:** Design all View Forms and Action Forms **before** starting the workflow.
2. **Naming conventions:**
   - Steps -> **Nouns** (e.g. "Client Approval", "Finance Review"). Test: complete the phrase "Sent for ____"
   - Links/Actions -> **Verbs** (e.g. "Submit for Clarification", "Reject").
3. **Optimize Notifications:** Each action generates a potential email. Keep the number of steps to a minimum.
4. **Avoid unnecessary loops:** Do not recycle actions by sending records back to users unless strictly necessary.
5. **Prioritize Multiple Approvers:** Use "Multiple-approvers" steps instead of serial approvals whenever possible.
6. **Consistency in Terminal Statuses:** If the inbound action of a step has terminal status, all outbound actions must have the same terminal status.
7. **Unified Final Form:** If a BP has multiple workflows, use the same form for the End Step across all of them.
8. **Do not delete workflows in Development:** Deactivate them and republish with a new workflow.
9. **Design on paper first:** Before touching uDesigner.
10. **Use Roles instead of direct Users:** Facilitates configuration and maintenance in Unifier.

---

## 16. Design Patterns {#patterns}

### Pattern 1: Multiple Approval (Oracle Recommended)
Instead of chaining "Reviewer 1 -> Reviewer 2 -> Reviewer 3", group them into a single step:

```
[Create Step] Creator -> [Submit]
[Multiple Approvers Step] Committee Review
  -> [Action: Approve] Rule: Majority/All -> [End Step: Approved]
  -> [Action: Reject]  Rule: Any One     -> [End Step: Rejected]
```

### Pattern 2: Conditional Routing
Use system logic to route based on financial thresholds or metadata:

```
[Create Step] Creator -> [Submit]
[Conditional Step] Amount Evaluation (Automatic)
  -> [Logic: Amount <= 10k]  -> [Standard Step: Direct Manager]
  -> [Logic: Amount > 10k]   -> [Standard Step: General Management]
```

### Pattern 3: Chained Auto-Creation
```
[Create Step] Service Request -> [Submit]
[Standard Step] SR Approval -> [Approve + Auto-Create Work Order]
  -> [End Step: Approved]
[Conditional Step] WO Processing -> [Automatic]
  -> [Completed] -> [End Step: Closed]
```

---

## 17. Advanced Configuration and SLAs (Unifier Admin Phase) {#advanced}

Once the schema is published from uDesigner to Unifier, the Administrator configures:

### Actor Types
- **Named Users:** Specific users (not recommended for scalability).
- **Groups:** Unifier security groups.
- **Roles (recommended):** Project/shell roles (e.g. "Project Manager", "Financial Reviewer").
- **Manager of Creator:** The manager of the record creator.

### Delegation
Allows a user to temporarily delegate their inbox. The system respects confidentiality permissions.

### SLAs (Response Times)
- **Calendar Days vs. Business Days:** Selection of day type for time computation.
- **Warning Alerts:** Reminder emails before the deadline expires.
- **Escalation Actions:** Automatic reassignment to the manager if the deadline is missed.

---

## 18. Design Checklist {#checklist}

Before publishing and moving a Workflow to production:

### uDesigner (Logical Design)
- [ ] All View and Action forms were created before starting the workflow.
- [ ] Step names are nouns; link names are verbs.
- [ ] There is exactly one Create Step and clear paths to the End Steps.
- [ ] Terminal actions entering a step match the terminal actions leaving that step.
- [ ] All workflows within the BP use the same form for their End Steps.
- [ ] Parallel approvals (Multiple-approvers) were prioritized over long sequential chains.
- [ ] The workflow was validated with **Error Check** with no errors.
- [ ] **Step Order** was adjusted to facilitate configuration in Unifier.
- [ ] The workflow was printed to PDF for documentation.

### Unifier (Administrative Configuration)
- [ ] Each step has a valid actor assigned (preferably a Project Role).
- [ ] SLAs are configured to prevent bottlenecks.
- [ ] Email notifications are strictly limited to necessary actions.
- [ ] The process was tested end-to-end (happy path, rejections, returns, and edge conditions).
- [ ] Terminal status rules were verified across all paths.