# Formulas and Data Logic in Unifier

## Table of Contents
1. [Auto-Populate](#auto-populate)
2. [Advanced Formulas in Business Processes](#advanced-formulas-in-business-processes)
3. [Rules for Formula Destination Fields](#rules-for-formula-destination-fields)
4. [Available Functions and Formats](#available-functions-and-formats)
5. [Conditional Routing](#conditional-routing)
6. [Validation Rules](#validation-rules)
7. [SBS Formulas](#sbs-formulas)
8. [Cost Sheet Formulas](#cost-sheet-formulas)
9. [Time Zone in Advanced Formula](#time-zone-in-advanced-formula)
10. [Common Errors and Edge Cases](#common-errors)

---

## Auto-Populate {#auto-populate}

Auto-population fills fields automatically when a record is created or modified.

### Auto-populate types:

#### 1. From another field in the same form
```
Destination field: [project_desc_s]
Source: Field [shell_name_s] from the same form
Trigger: On record creation
```

#### 2. From the parent Shell
```
Destination field: [project_code_s]
Source: Shell -> field [shell_number]
Use: Ensure all records in the shell carry the project code
```

#### 3. From a related BP
```
Scenario: Change Order inherits the contract number from the parent Contract
Destination field: [contract_num_s] in Change Order
Source: BP "Contract" -> field [contract_number_s]
Relationship type: Parent-child BP
```

#### 4. From Dynamic Data Set (DDS)
```
User selects a vendor (DDS)
-> Auto-populated: tax ID, address, contact, bank
Source: Vendor Master BP
```

### Auto-Populate Considerations:
- Determine whether the auto-populated field should be **editable** or **read-only** afterward.
- Define behavior when the source is **null** (leave blank vs. set a default).
- Auto-populate **does not recalculate** if the source changes later (it is one-time at the trigger moment).
- For continuous updates, use **Advanced Formulas** instead of auto-populate.
- **Important:** If you change the input method from Auto-populate to Advanced Formula, all previous formatting will be lost.

---

## Advanced Formulas in Business Processes {#advanced-formulas-in-business-processes}

Advanced Formulas allow calculating the value of fields in Unifier forms. Fields with a formula are **read-only** at runtime. Formulas can be created for fields of type **Numeric**, **String**, and **Date**.

### Formula engine capabilities:
- If-then-else conditions on string, numeric, and date fields.
- Complex compound conditions, for example: `((a OR b) AND (c AND d)) OR (e AND f)`.
- Expression language support beyond simple arithmetic functions.
- Composite fields that combine multiple text data elements into one.

### Where they can be used:
- BP forms (upper form and detail form) and grids.
- BP logs and Line Item logs.
- Search parameters in the BP log and Line Item list log.
- Form-level validations that depend on formula fields.

### When the formula is re-evaluated:
- Manual edits to source fields.
- Web service methods (REST only).
- Mobile app.
- In **Workflow BPs**: when the record is sent to the next step (the new value is visible at the next step).
- In **Non-workflow BPs**: when the record is submitted.

### Restrictions:
- Not available for **Summary Payment Application (SPA)** SOV type BPs (Base Commit Type BP Line Item, Change Commit Type BP Line Item) or **Payment Application BPs**.
- Hyperlinks in advanced formulas are only validated at runtime, not during creation.
- Unifier checks for **circular referencing** when saving the formula definition.

### How to add an Advanced Formula:
1. In the **Elements Properties window**, select the data element that supports a formula.
2. The **Advanced Formula** window opens for `<Data Element Name> - <Data Element Label>`.
3. Type the Data Element name directly or insert elements and functions from the corresponding fields.
4. There is no character limit in the definition field.
5. Supports standard keyboard shortcuts: `Ctrl+C`, `Ctrl+V`, `Ctrl+Z`, `Ctrl+Y`, `Ctrl+A`.
6. Formulas can be copied and pasted from other definitions.

> **Efficiency tip:** To reduce design time, define the formula once and copy it to other fields with similar definitions. This also applies to "Published" designs.

### Sample Calculations tab:
After passing syntax validation, Unifier displays the message:

> *"There is no syntax error in the formula definition. However, there might be calculation errors. The 'Sample Calculations' tab is now available..."*

On this tab:
- All fields that make up the formula appear as columns.
- Sample data is generated based on the data type.
- If there are errors, the error column will be populated with the corresponding message.
- The user can modify sample values to recalculate.
- It is recommended to review and fix calculation errors **before** saving.
- If saved with errors, Unifier displays: *"There are some calculation errors. Do you still want to save the formula?"*

### Fallback values when the formula cannot be evaluated:
| Field type | Assigned value |
|------------|----------------|
| String     | Blank          |
| Date       | Blank          |
| Numeric    | Blank or 0     |

### General syntax:
```
[result_field] = mathematical_or_logical_expression
```

### Available operators:
| Operator | Use            |
|----------|----------------|
| `+`      | Addition       |
| `-`      | Subtraction    |
| `*`      | Multiplication |
| `/`      | Division       |
| `%`      | Modulo         |
| `( )`    | Grouping       |

### Example with delimiters (string constants):
When the delimiter option does not appear in the interface, they are added as string constants:

```
Submittal number = Project # + " - " + Project Type + " - " + Submittal unique #
```
At runtime: `San Jose Recreation Park - New Construction - SUB009`

### Common formula examples:

#### Total calculation:
```
line_total_d = SUM(lines.subtotal_d)
tax_amount_d = line_total_d * 0.19
grand_total_d = line_total_d + tax_amount_d
```

#### Budget variance:
```
variance_d = approved_budget_d - actual_cost_d
variance_pct_d = IF(approved_budget_d != 0,
                    (variance_d / approved_budget_d) * 100,
                    0)
```

#### Days overdue:
```
overdue_days_i = IF(actual_date_dt > planned_date_dt,
                    dateDiff(planned_date_dt, actual_date_dt),
                    0)
```

#### Progress percentage (with zero protection):
```
progress_pct_d = IF(planned_hours_d != null AND planned_hours_d != 0,
                    (actual_hours_d / planned_hours_d) * 100,
                    0)
```

#### Sequential number with format:
```
full_code_s = concat("PO-", toString(year(today()), "0000"), "-", bp_number)
```

---

## Rules for Formula Destination Fields {#rules-for-formula-destination-fields}

The fundamental product rules for defining formulas on different field types remain the same. The following **Input types are blocked** from being defined as a formula destination:

| Blocked type                              |
|-------------------------------------------|
| Checkbox                                  |
| Pull-down Menu (drop-down list)           |
| Radio Buttons                             |
| Multi-Select Input                        |
| All pickers (including Data Pickers)      |

Additionally, the following data elements **cannot be formula destinations**:
- **QBDE** (e.g. "Pending Changes")
- **BP Creator**
- **Record Number** (`record_no`)

> **Note:** If the designer selects a QBDE as a formula destination, Unifier will prevent the action.

### Fields included and excluded in the Formula Definition Window:

| Field type | Included | Excluded |
|------------|----------|----------|
| **String** | String Pull Down, Multiple Text Lines, Text Box, Radio Buttons (string), Multi-Select Input | The field for which the formula is being defined |
| **Numeric** | Numeric Amount (Decimal, Integer, Currency), Checkbox (integer), Integer Radio button, Integer Pull Down | The field for which the formula is being defined |
| **Date** | All date fields | BP Creators, all pickers (including Data Pickers), Rich Text Field, the field for which the formula is being defined |

> **Note:** Unifier supports linked elements and summary elements as part of the formula definition. Type-ahead and function listing are available in all fields. Each function has a tooltip with its description.

---

## Available Functions and Formats {#available-functions-and-formats}

Functions are classified into three categories. Full official reference from Oracle Unifier (Last Published: February 19, 2026).

### String Functions

| Function | Description |
|----------|-------------|
| `compare(str1, str2)` | Compares two strings |
| `compareIgnoreCase(str1, str2)` | Compares ignoring case |
| `concat(str1, str2)` | Concatenates strings; supports numerics as strings |
| `contains(str, search)` | Checks if a string contains another |
| `containsAny(str, search1, search2, ...)` | Checks if it contains any of the given strings |
| `containsIgnoreCase(str, search)` | Case-insensitive check |
| `endsWith(str, suffix)` | Checks if it ends with the given suffix |
| `endsWithIgnoreCase(str, suffix)` | Same, case-insensitive |
| `indexOf(str, search)` | Position of a string within another (from start) |
| `indexOf(str, search, int)` | Position from a defined point |
| `indexOfAny(str, search1, search2, ...)` | Position of any of the searched strings |
| `indexOfIgnoreCase(str, search)` | Same, case-insensitive |
| `isAlpha(str)` | Checks if the string contains only letters |
| `isBlank(str)` | Checks if blank (null, empty, or spaces only) |
| `isEmpty(str)` | Checks if empty |
| `isNumeric(str)` | Checks if the string contains only digits |
| `isNumericSpace(str)` | Checks if it contains only digits and spaces |
| `lowerCase(str)` | Converts to lowercase |
| `upperCase(str)` | Converts to uppercase |
| `matches(str, regex)` | Checks if the string matches the regex |
| `left(str, len)` | Extracts the first N characters |
| `leftPad(str, size)` | Left-pads with spaces |
| `leftPad(str, size, padStr)` | Left-pads with the given string |
| `mid(str, pos, len)` | Extracts substring from position with given length |
| `right(str, len)` | Extracts the last N characters |
| `rightPad(str, size)` | Right-pads with spaces |
| `rightPad(str, size, padStr)` | Right-pads with the given string |
| `replace(str, search, replacement)` | Replaces all occurrences |
| `replaceAll(str, regex, replacement)` | Replaces based on regex |
| `replaceFirst(str, regex, replacement)` | Replaces first occurrence based on regex |
| `startsWith(str, prefix)` | Checks if it starts with the prefix |
| `startsWithIgnoreCase(str, prefix)` | Same, case-insensitive |
| `size(str)` | String length |
| `strip(str)` | Removes leading and trailing spaces |
| `substring(str, start)` | Extracts substring from position to end |
| `substring(str, start, end)` | Extracts substring between two positions |
| `truncate(str, maxlen)` | Truncates string to the specified maximum length |
| `toString(format)` | Converts number/currency to string with defined format (e.g. value `9` with format `"00000.00"` -> `"00009.00"`) |

> **Note:** String comparison (`=`, `!=`) is **not available** as a separate function.

### Numeric Functions

| Function | Description |
|----------|-------------|
| `abs(num)` | Absolute value |
| `sqrt(num)` | Square root |
| `pow(base, exponent)` | Power |
| `log(num)` | Base-10 logarithm |
| `log10(num)` | Base-10 logarithm |
| `ln(num)` | Natural logarithm |
| `min(num1, num2)` | Minimum of a set of numbers |
| `max(num1, num2)` | Maximum of a set of numbers |
| `ceil(num)` | Round up (integer >= value) |
| `floor(num)` | Round down (integer <= value) |
| `round(num, places)` | Round to N decimal places |
| `isNumber(num)` | Checks if the value is a number |
| `numberFormat(num, pattern)` | Formats number with pattern |
| `toDouble(num)` | Converts to double |
| `toInt(num)` | Converts to integer |
| `toLong(num)` | Converts to long |
| `cos(num)` | Cosine |
| `sin(num)` | Sine |
| `tan(num)` | Tangent |
| `acos(num)` | Arc cosine |
| `asin(num)` | Arc sine |
| `atan(num)` | Arc tangent |
| `cosh(num)` | Hyperbolic cosine |
| `sinh(num)` | Hyperbolic sine |
| `tanh(num)` | Hyperbolic tangent |

> **Note:** Numeric comparison (`=`, `!=`, `>`, `>=`, `<`, `<=`) is **not available** as a separate function.

### Date Functions

| Function | Description |
|----------|-------------|
| `date(year, month, day)` | Builds a date from year, month, and day |
| `day(date)` | Extracts the day from a date |
| `month(date)` | Extracts the month from a date |
| `year(date)` | Extracts the year from a date |
| `weekday(date)` | Day-of-week number |
| `weeknum(date)` | Week number of the year |
| `today()` | Current date (uses the time zone defined in the formula) |
| `now()` | Current date and time |
| `edate(date, months)` | Adds or subtracts months from a date |
| `eomonth(date, months)` | Last day of the month, N months forward or back |
| `addDays(date, amount)` | Adds or subtracts days |
| `addHours(date, amount)` | Adds or subtracts hours |
| `addMonths(date, amount)` | Adds or subtracts months |
| `addWeeks(date, amount)` | Adds or subtracts weeks |
| `addYears(date, amount)` | Adds or subtracts years |
| `dateDiff(start, end)` | Difference between two dates |
| `dateFormat(date, pattern)` | Converts date to string with format |
| `isSameDay(date1, date2)` | Checks if two dates are the same day |
| `toLong(date)` | Converts date to long (timestamp) |
| `toDate(long)` | Converts long (timestamp) to date |
| `toString(format)` | Converts date to string with defined format |

> **Note:** The following date operations are **not available** as direct functions (must be resolved with `addDays`, `addMonths`, etc.): converting datetime to date only, date comparison (`=`, `!=`, `>`, `<`).

---

## Conditional Routing {#conditional-routing}

Conditional routing defines which Workflow step the record goes to based on the value of certain fields.

### Routing condition structure:
```
Field: [total_amount_d]
Condition 1: total_amount_d > 100000 -> Step "Board Approval"
Condition 2: total_amount_d > 10000  -> Step "Management Approval"
Condition 3: (default)               -> Step "Supervisor Approval"
```

### Condition operators:
| Operator    | Meaning                     |
|-------------|-----------------------------|
| `=`         | Equal to                    |
| `!=`        | Not equal to                |
| `>`         | Greater than                |
| `>=`        | Greater than or equal to    |
| `<`         | Less than                   |
| `<=`        | Less than or equal to       |
| `IN`        | Is in list                  |
| `IS NULL`   | Is null/empty               |
| `CONTAINS`  | Contains (for text)         |

### Compound conditions:
```
(total_amount_d > 50000) AND (contract_type_pk = "CRITICAL")
-> Board Approval + Legal Advisor

(country_s = "US") OR (currency_pk = "USD")
-> Step "Finance Review"
```

### Routing best practices:
- Always define a **default** condition (else/otherwise).
- Order conditions from **most restrictive to least restrictive**.
- Document the business logic behind each threshold.
- Avoid more than 4-5 branches (the diagram becomes unmanageable).

---

## Validation Rules {#validation-rules}

Validation rules prevent saving/submitting records with invalid data.

### Validation types:

#### 1. Conditional required field:
```
IF request_type_pk = "URGENT"
THEN field justification_s is REQUIRED
```

#### 2. Range validation:
```
end_date_dt MUST BE >= start_date_dt
```
Error message: "End date cannot be earlier than start date"

#### 3. Sum validation:
```
SUM(lines.distribution_pct_d) MUST EQUAL 100
```
Message: "Percentage distribution must add up to exactly 100%"

#### 4. Cross-field validation:
```
IF total_amount_d > 0 AND contract_number_s IS NULL
THEN ERROR: "Contract number is required for amounts greater than zero"
```

### Error message configuration:
- Write messages **in the end user's language**.
- Indicate exactly **what the user needs to correct**.
- Avoid technical messages such as "Null violation on field X".

---

## SBS Formulas {#sbs-formulas}

Summary Budget Sheets (SBS) aggregate information from multiple projects/CBS codes.

### SBS cell types:
| Type        | Function |
|-------------|----------|
| **Rollup**  | Sum from Cost Sheets of child shells |
| **Formula** | Calculation between other SBS columns |
| **Manual**  | Value entered directly |
| **Derived** | Calculated from another SBS |

### SBS formula example:
```
Column "Variance"       = [Approved Budget] - [Forecast Cost]
Column "% Executed"     = IF([Approved Budget] > 0,
                             ([Actual Cost] / [Approved Budget]) * 100,
                             0)
```

---

## Cost Sheet Formulas {#cost-sheet-formulas}

### Typical calculated columns in Cost Sheet:
```
Final Budget  = Original Budget + Approved Changes
Variance      = Final Budget - Forecast EAC
% Variance    = (Variance / Final Budget) * 100
ETC (Estimate to Complete) = Forecast EAC - Actual Cost
```

### Rollup rules from BPs:
```
Approved "Purchase Order" BP
  -> adds to "Commitments" column of the corresponding CBS
  -> when a Payment Certificate is generated
    -> moves from "Commitments" to "Actual Costs"
```

---

## Time Zone in Advanced Formula {#time-zone-in-advanced-formula}

The **Time Zone** field is **mandatory** in formula definitions that involve date operations.

### Default behavior:
- The system assigns **5 PM** as the default time.
- When designing the formula, the **designer's** time zone is pre-populated, but it can be changed to match the client's time zone.
- The defined time zone applies to functions such as `dateDiff()` (combining Date Picker and Date-Only Picker) and `today()`.

### Time Zone field tooltip:
> *"The Time Zone specified here will be used for any formula calculations involving Date and Date Only Picker fields. In addition, certain Date Functions will also use this value. At runtime, Unifier converts the formula evaluated value to retain the Time Zone of the logged-in user."*

### Difference between Date Picker and Date-Only Picker:

| | Date Picker | Date-Only Picker |
|---|---|---|
| **Stored timestamp** | Server time zone (e.g. 10 PM GMT) | Always 12:00 AM (00:00) |
| **User time zone adjustment** | Yes (Unifier converts based on user preference) | No (no adjustment is made) |
| **Time visible to user** | Yes (e.g. 2:00 PM PDT for User A in CA) | No (date only) |

### Example scenario with `dateDiff`:
- **User A** in California (PDT): sees 2:00 PM.
- **User B** in New York (EDT): sees 5:00 PM.
- **Server** in London (GMT): stores 10:00 PM.

When a Date-Only Picker (timestamp: 12:00 AM) is combined with a Date Picker (timestamp: 2:00 PM), a `dateDiff` may **return 0 days** because the time difference does not exceed 24 hours. To resolve this, Unifier converts the Date-Only Picker to the user's equivalent timestamp (2:00 PM) at calculation time.

> **Edge case:** The result of `dateDiff` can vary depending on the logged-in user's time zone when the two picker types are mixed. Correctly defining the time zone in the formula minimizes inconsistencies.

---

## Common Errors and Edge Cases {#common-errors}

### Division by zero:
```
INCORRECT:
  progress_pct_d = (actual_d / planned_d) * 100
  -> Fails if planned_d = 0

CORRECT:
  progress_pct_d = IF(planned_d != null AND planned_d != 0,
                      (actual_d / planned_d) * 100, 0)
```

### Null dates in dateDiff:
```
INCORRECT:
  days_d = dateDiff(start_date_dt, end_date_dt)
  -> Fails if either date is null

CORRECT:
  days_d = IF(start_date_dt != null AND end_date_dt != null,
              dateDiff(start_date_dt, end_date_dt), 0)
```

### SUM over empty line items:
```
SUM() on a table with no lines returns NULL, not 0.
Behavior when the formula cannot be evaluated: Numeric -> blank or 0, String -> blank, Date -> blank.
Protect with: IF(isNumber(SUM(lines.amount_d)), SUM(lines.amount_d), 0)
```

### Circular reference:
```
Field A depends on Field B
Field B depends on Field A
-> Unifier checks for circular references when SAVING the definition.
-> If detected, it blocks saving.
-> Always review the dependency graph before publishing.
```

### Auto-populate vs. Advanced Formula:
```
Auto-populate: Runs only ONCE (on creation or when the source field changes)
Advanced Formula: Re-evaluates EVERY TIME any source field changes
                  (manual edit, REST web service, or mobile app)
-> Choose the correct type based on whether the value needs to update dynamically.
-> Switching from Auto-populate to Advanced Formula removes all previous field formatting.
```

### Fields blocked as formula destinations:
```
The following types CANNOT be formula destinations:
  - Checkbox, Pull-down Menu, Radio Buttons, Multi-Select Input
  - All pickers (including Data Pickers)
  - QBDE, BP Creator, Record Number
-> If you attempt to define it, Unifier will block the action.
```

### Formulas in unsupported BPs:
```
Advanced Formulas are NOT available in:
  - Summary Payment Application (SPA) SOV type BPs
  - Base Commit Type BP Line Item
  - Change Commit Type BP Line Item
  - Payment Application Business Processes
```

### Syntax validation vs. calculation errors:
```
Unifier validates SYNTAX on save, but calculation errors
(e.g. division by zero or incompatible types) are only detected
in the "Sample Calculations" tab or at runtime.
-> Always review the Sample Calculations tab before publishing.
```