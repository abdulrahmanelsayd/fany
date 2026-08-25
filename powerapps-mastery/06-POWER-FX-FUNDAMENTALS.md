# 06 — Power Fx Fundamentals

## Goal

Read and write Power Fx confidently, understand its value types and evaluation model, and build formulas instead of copying them blindly.

## What Power Fx is

Power Fx is a strongly typed, declarative, functional language influenced by spreadsheet formulas.

Declarative formula:

```powerfx
lblTotal.Text = Text(Sum(galLines.AllItems, Quantity * UnitPrice), "$#,##0.00")
```

The label recalculates when its dependencies change. You do not write an event listener to keep it synchronized.

Behavior formula:

```powerfx
btnSave.OnSelect =
    Set(varSaving, true);
    SubmitForm(frmRequest)
```

Behavior properties perform actions in sequence.

## Core value types

| Type | Example |
|---|---|
| Text | `"Equipment"` |
| Number | `125.5` |
| Boolean | `true` |
| Date | `Today()` |
| Date and time | `Now()` |
| Blank | `Blank()` |
| Record | `{Title: "Monitor", Amount: 210}` |
| Table | `Table({Status: "Draft"}, {Status: "Approved"})` |
| Color | `ColorValue("#0F6CBD")` |

A record is one item with named fields. A table is a set of records.

## Operators

| Category | Operators/examples |
|---|---|
| Arithmetic | `+`, `-`, `*`, `/` |
| Comparison | `=`, `<>`, `<`, `<=`, `>`, `>=` |
| Boolean | `&&`, `||`, `!` or their word equivalents |
| Text concatenation | `&` |
| Membership | `in`, `exactin` where supported |

Examples:

```powerfx
Amount > 500 && Status = "Submitted"
```

```powerfx
"REQ-" & Text(RequestId, "0000")
```

## Blank, empty string, and zero

These are not interchangeable:

- `Blank()` means no value.
- `""` is zero-length text.
- `0` is a number.
- An empty table has no records.

Use:

```powerfx
IsBlank(txtTitle.Value)
```

For controls that can return an empty string, this is often safer:

```powerfx
IsBlank(Trim(txtTitle.Value))
```

Use `IsEmpty(tableExpression)` for a table.

## Conditions

### If

```powerfx
If(
    Amount > 1000,
    "Director approval",
    Amount > 250,
    "Manager approval",
    "Automatic approval"
)
```

`If` checks conditions in order.

### Switch

```powerfx
Switch(
    Status,
    "Draft", "Continue editing",
    "Submitted", "Waiting for review",
    "Approved", "Approved",
    "Unknown status"
)
```

Use `Switch` when comparing one expression with several values.

## Record scope: ThisItem, ThisRecord, and As

Inside a gallery template:

```powerfx
ThisItem.Title
```

Inside table functions such as `Filter`:

```powerfx
Filter(Requests, ThisRecord.Amount > 100)
```

Usually the field name alone is sufficient:

```powerfx
Filter(Requests, Amount > 100)
```

Use `As` to clarify nested scopes:

```powerfx
ForAll(
    colDraftLines As line,
    Patch(
        RequestLines,
        Defaults(RequestLines),
        {
            Quantity: line.Quantity,
            UnitPrice: line.UnitPrice
        }
    )
)
```

## Essential table functions

### Filter: zero or more matches

```powerfx
Filter(Requests, Status = "Submitted")
```

### LookUp: first matching record or selected formula result

```powerfx
LookUp(Requests, RequestId = varRequestId)
```

```powerfx
LookUp(Requests, RequestId = varRequestId, Title)
```

### Sort and SortByColumns

```powerfx
SortByColumns(Requests, "RequestedOn", SortOrder.Descending)
```

### AddColumns

```powerfx
AddColumns(
    Requests,
    TotalWithTax,
    Amount * 1.14
)
```

This shapes the returned table; it does not add a physical source column.

### ShowColumns

```powerfx
ShowColumns(Requests, Title, Status, RequestedOn)
```

Use Studio autocomplete because column-name syntax has evolved and varies with display/logical names.

### First and FirstN

```powerfx
First(SortByColumns(Requests, "RequestedOn", SortOrder.Descending))
```

## Aggregates

```powerfx
Sum(colDraftLines, Quantity * UnitPrice)
```

```powerfx
Average(Filter(Requests, Status = "Approved"), Amount)
```

```powerfx
CountRows(Filter(Requests, Status = "Submitted"))
```

Delegation rules also apply to aggregates against external data.

## Text, number, and date functions

```powerfx
Trim(txtTitle.Value)
```

```powerfx
Lower(User().Email)
```

```powerfx
Value(txtAmount.Value)
```

```powerfx
Text(ThisItem.Amount, "$#,##0.00")
```

```powerfx
DateAdd(Today(), 14, TimeUnit.Days)
```

```powerfx
DateDiff(ThisItem.RequestedOn, Today(), TimeUnit.Days)
```

Formatting with `Text` produces text. Keep the underlying data typed as a number/date for calculations.

## With: name intermediate values

```powerfx
With(
    {
        cleanTitle: Trim(txtTitle.Value),
        requestAmount: Value(txtAmount.Value)
    },
    If(
        IsBlank(cleanTitle),
        "Title is required",
        requestAmount > 1000,
        "Director approval required",
        "Ready"
    )
)
```

`With` improves readability and prevents repeating an expression. It does not create mutable global state.

## Coalesce

Return the first nonblank value:

```powerfx
Coalesce(ThisItem.ManagerComment, "No manager comment")
```

## Sequence and ForAll

```powerfx
ForAll(
    Sequence(5),
    {StepNumber: Value, Label: "Step " & Text(Value)}
)
```

`ForAll` is useful but often overused. Prefer set-based server operations and built-in table functions. Do not assume iteration order or use it to solve every data transformation.

## Formula construction method

When a formula feels difficult:

1. State the expected return type: text, Boolean, record, table, or behavior.
2. Build the innermost expression.
3. Put it temporarily in a label or collection to inspect the result.
4. Add one condition/transformation at a time.
5. Use `With` and `As` to name ambiguous parts.
6. Check delegation before calling it finished.

## Guided exercises

Assume a `Requests` table with `Title`, `Status`, `Amount`, `RequestedOn`, and `RequestedByEmail`.

### A. Current user's submitted requests

```powerfx
Filter(
    Requests,
    Status = "Submitted" &&
    Lower(RequestedByEmail) = Lower(User().Email)
)
```

Validate whether `Lower` is delegable for your connector. A normalized email column can be a better design for server-side comparisons.

### B. Approval label

```powerfx
If(
    ThisItem.Amount >= 1000,
    "Director approval",
    ThisItem.Amount >= 250,
    "Manager approval",
    "Standard review"
)
```

### C. Age text

```powerfx
With(
    {daysOld: DateDiff(ThisItem.RequestedOn, Today(), TimeUnit.Days)},
    If(daysOld = 0, "Today", Text(daysOld) & " days ago")
)
```

## Common mistakes

- Using `=` as an assignment operator. Power Fx properties are formulas; behavior functions such as `Set` change variables.
- Treating a table as a record or a record as text.
- Converting all values to text too early.
- Copying formulas with the wrong regional separators.
- Writing one giant nested formula without `With`, named formulas, or component boundaries.
- Ignoring delegation because the formula works with ten records.

## Interview questions

**What is the difference between `Filter` and `LookUp`?**  
`Filter` returns a table containing every match. `LookUp` returns the first matching record, or a selected value when a reduction formula is supplied.

**What does declarative mean in Power Fx?**  
You describe what a property should evaluate to based on dependencies; Power Apps recalculates it when those dependencies change.

**When do you use `With`?**  
To name intermediate values and make a single formula clearer without introducing global mutable state.

## Challenge

Return `Overdue` only when a request is not approved/rejected and `NeededBy` is before today.

### Answer

```powerfx
If(
    !(Status in ["Approved", "Rejected"]) && NeededBy < Today(),
    "Overdue",
    Status
)
```

Check whether the chosen `in` expression is delegable when applied to an external table. An explicit pair of comparisons may be safer for some sources.

## Primary references

- [Power Fx overview](https://learn.microsoft.com/power-platform/power-fx/overview)
- [Power Fx formula reference](https://learn.microsoft.com/power-platform/power-fx/formula-reference-canvas-apps)
- [Tables, records, and record scope](https://learn.microsoft.com/power-platform/power-fx/tables)

