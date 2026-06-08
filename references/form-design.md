
At runtime: San Jose Recreation Park - New Construction - SUB009

Delimiters must be added as string constants within the formula.

---

### 13.3 Complete Advanced Formula Function Catalog

#### String Functions

| Function | Description |
|----------|-------------|
| compare(str1,str2) | Compares two strings |
| compareIgnoreCase(str1,str2) | Compares ignoring case |
| concat(str1,str2) | Concatenates two strings |
| contains(str,search) | Checks if contains a substring |
| containsAny(str,search1,search2,...) | Checks if contains any of the substrings |
| containsIgnoreCase(str,search) | Contains ignoring case |
| endsWith(str,suffix) | Checks if ends with a suffix |
| endsWithIgnoreCase(str,suffix) | EndsWith ignoring case |
| indexOf(str,search) | First position of a substring |
| indexOfAny(str,search1,search2,...) | First position of any substring |
| indexOfIgnoreCase(str,search) | IndexOf ignoring case |
| isAlpha(str) | Checks if only alphabetic |
| isBlank(str) | Checks if blank (empty or spaces) |
| isEmpty(str) | Checks if empty |
| isNumeric(str) | Checks if numeric |
| isNumericSpace(str) | Checks if numeric (allows spaces) |
| lowerCase(str) | Converts to lowercase |
| upperCase(str) | Converts to uppercase |
| matches(str,regex) | Checks if matches regular expression |
| left(str,len) | Extracts first N characters |
| leftPad(str,size) | Left pads with spaces |
| leftPad(str,size,padStr) | Left pads with a string |
| mid(str,pos,len) | Extracts from position with length |
| right(str,len) | Extracts last N characters |
| rightPad(str,size) | Right pads with spaces |
| rightPad(str,size,padStr) | Right pads with a string |
| replace(str,search,replacement) | Replaces substring |
| replaceAll(str,regex,replacement) | Replaces all occurrences (regex) |
| replaceFirst(str,regex,replacement) | Replaces the first occurrence (regex) |
| startsWith(str,prefix) | Checks if starts with a prefix |
| startsWithIgnoreCase(str,prefix) | StartsWith ignoring case |
| size(str) | String length |
| strip(str) | Removes leading and trailing spaces |
| substring(str,start) | Substring from position to end |
| substring(str,start,end) | Substring between positions |
| truncate(str,maxlen) | Truncates to maximum length |

#### Numeric Functions

| Function | Description |
|----------|-------------|
| abs(num) | Absolute value |
| max(num1,num2) | Maximum of two numbers |
| min(num1,num2) | Minimum of two numbers |
| ceil(num) | Rounds up to the nearest integer |
| floor(num) | Rounds down to the nearest integer |
| log(num) | Base 10 logarithm |
| log10(num) | Base 10 logarithm (explicit) |
| pow(base,exponent) | Power |
| round(num,places) | Rounds to N decimal places |
| sqrt(num) | Square root |
| isNumber(num) | Checks if valid number |
| numberFormat(num,pattern) | Formats number according to pattern (e.g., "00000.00") |
| toDouble(num) | Converts to Double |
| toInt(num) | Converts to Integer |
| toLong(num) | Converts to Long |
| cos(num) | Cosine |
| sin(num) | Sine |
| tan(num) | Tangent |
| acos(num) | Arc cosine |
| asin(num) | Arc sine |
| atan(num) | Arc tangent |
| cosh(num) | Hyperbolic cosine |
| sinh(num) | Hyperbolic sine |
| tanh(num) | Hyperbolic tangent |

#### Date Functions

| Function | Description |
|----------|-------------|
| date(year,month,day) | Creates a date |
| day(date) | Extracts the day |
| edate(date,months) | Adds/subtracts months to a date |
| eomonth(date,months) | Last day of the month, adding/subtracting months |
| month(date) | Extracts the month |
| now() | Current server date and time |
| today() | Current server date (without time) |
| weekday(date) | Day of the week (1=Monday, 7=Sunday) |
| weeknum(date) | Week number of the year |
| year(date) | Extracts the year |
| addDays(date,amount) | Adds days to a date |
| addHours(date,amount) | Adds hours to a date |
| addMonths(date,amount) | Adds months to a date |
| addWeeks(date,amount) | Adds weeks to a date |
| addYears(date,amount) | Adds years to a date |
| dateDiff(start,end) | Difference in days between two dates |
| dateFormat(date,pattern) | Formats date according to pattern |
| isSameDay(date1,date2) | Checks if they are the same day |
| toLong(date) | Converts date to long (timestamp) |
| toDate(long) | Converts long (timestamp) to date |

---

## 14. Value Validation

You can define validation rules for fields in upper and detail forms.

### Oracle Recommendation

To validate Currency Amount or Decimal Amount, use conditions like `< 0.001` or `> -0.001` instead of `= 0`. The `= 0` condition should only be used on Integer data elements.

---

## 15. Reverse Auto-Population (RAP)

Reverse Auto-Population updates fields in the source record when a field in the destination record is modified. Useful for synchronizing data between related BPs.

---

## 16. Auto-Creation of Records and Line Items

You can configure a form to automatically create a new record or line item based on:

- A condition (e.g., dollar amount)
- A frequency (e.g., daily, weekly)
- Condition + frequency

---

## 17. Reference Process

A Reference Process populates fields in a BP picker or line item picker from a reference BP. Used in forms that employ pickers to select records from other BPs.

---

## 18. Snapshots

uDesigner allows saving different versions of a design using snapshots. This creates an audit trail of changes.

### To Take a Snapshot

1. Open the BP/manager/shell in Admin mode
2. Go to File > Snapshots
3. Click New, enter name and description
4. uDesigner timestamps the snapshot with date and time

### When Snapshots Are Taken Automatically

- When the BP is marked as Complete
- When the BP is published to production

### Restore a Snapshot

You can restore all or part of a published design to a previous version. See Restoring a Version of a Design in the official documentation.

---

## 19. Upper Form Options by BP Type

Each BP type has specific options in the Options tab of the Upper Form:

| BP Type | Specific Options |
|---------|------------------|
| Cost | Funding options, line item settings, CBS integration |
| Document | Folder structure, auto-publish path, attachment source, append line items folder structure |
| Line Item | Multiple tabs, hide current tab, line item modifiers |
| Simple | Basic attachments and comments options |
| Text | Text entry area, response list |
| RFB | Send bid invitations, master vendor list filtering, hide received bids |
| Resource (Booking / Timesheet) | Resource and timesheet configuration |
| Project/Shell Creation | Project/shell creation options (Simple or Line Item) |
| Payment Application | SOV auto-population, funding options, CBS integration |
| Base Commits / Change Commits | Commitment funding sheets, SOV type options |

---

## 20. Design Checklist

### Pre-design

- Draft of each form ready on paper
- Required Data Elements (DEs) created in Administration mode
- BP type defined (Workflow vs Non-workflow, Cost, Line Item, etc.)

### Upper Form

- Descriptive and unique name
- Defined as Action Form or View Form as appropriate
- Options configured according to BP type (funding, attachments, comments, etc.)
- Blocks created with adequate number of columns and rows
- Hidden blocks for formula-only fields if necessary
- Fields added from Custom or Standard DEs
- Access type configured (Editable, Required, Read-Only)
- Formulas applied to calculated fields
- Auto-population configured from external sources or constants
- Validations defined for value ranges
- Preview verified visually

### Detail Form

- Item log configured if applicable
- Multiple tabs created if data organization is needed
- Grid summary configured if applicable
- Text entry area or response list configured for Text type BP

### Action and View Forms

- Action Form created for each workflow step requiring editing
- View Form created for read-only viewing
- For non-workflow BPs: additional View Forms created if different visibility levels are needed
- For shells: only one Action Form, multiple View Forms allowed
- Custom DEs on View Form also exist on Action Form

### Integration and Advanced

- Reference process configured for pickers
- Auto-creation of records/line items configured if applicable
- Reverse Auto-Population (RAP) configured if applicable
- Email notification elements selected
- Mobile fields selected
- Auto-sequencing configured if applicable
- Hyperlinks configured on reference fields
- Tooltips added where useful

### Versioning

- Snapshot taken before marking as Complete
- Snapshot taken before publishing to production

### Post-design

- Workflows designed with correct forms at each step
- The upper form of the End Step is the same across all BP workflows
- Successful publication to Unifier
- End-to-end testing in test environment