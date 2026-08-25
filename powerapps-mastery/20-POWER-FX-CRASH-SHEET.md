# 20 — Power Fx Crash Sheet for Tomorrow

## The sentence to remember

Power Fx is a **strongly typed, declarative, functional** formula language. In a canvas app, you normally select a control, choose a property, and write the formula that property should evaluate or execute.

```text
Control.Property = Formula
galRequests.Items = Filter(Requests, Status = "Submitted")
btnSave.OnSelect = SubmitForm(frmRequest)
```

In Studio you enter the right side in the selected property; you usually do not type the `Control.Property =` label itself.

## 1. Know the types before the functions

| Type | Example | Interview trap |
|---|---|---|
| Text | `"Laptop"` | Text `"100"` is not number `100`. |
| Number | `100.5` | Format only when displaying; keep numeric source typed. |
| Boolean | `true` | Used directly in `Visible`, validation, and filter conditions. |
| Date | `Today()` | Date-only versus date-time/time zone must be intentional. |
| Date/time | `Now()` | User-local/UTC behavior depends on source column settings. |
| Blank | `Blank()` | Not the same concept as zero, empty table, or always empty text. |
| Record | `{Title: "Laptop", Amount: 900}` | One item with named fields. |
| Table | `Table({Status:"Draft"},{Status:"Approved"})` | Even a single-column result can still be a table. |
| GUID | `GUID()` | Dataverse row IDs are GUIDs; avoid generating IDs with `Max()+1`. |

Use `IsBlank(value)` for a scalar and `IsEmpty(table)` for a table.

## 2. The eight functions you must write from memory

### `If`

```powerfx
If(
    Amount > 1000,
    "Director approval",
    Amount > 250,
    "Manager approval",
    "Standard review"
)
```

### `Switch`

```powerfx
Switch(
    Status,
    "Draft", "Continue editing",
    "Submitted", "Waiting for review",
    "Approved", "Complete",
    "Unknown"
)
```

### `Filter`

```powerfx
Filter(
    Requests,
    Status = "Submitted" && Amount >= 250
)
```

Returns a table containing every match.

### `LookUp`

```powerfx
LookUp(Requests, RequestId = varRequestId)
```

Returns the first matching record. With a third argument it can return one value:

```powerfx
LookUp(Requests, RequestId = varRequestId, Title)
```

### `Set`

```powerfx
Set(varSelectedRequest, galRequests.Selected)
```

Creates/updates an app-wide global variable.

### `UpdateContext`

```powerfx
UpdateContext({locShowDialog: true})
```

Creates/updates screen-scoped context state.

### `Patch`

```powerfx
Patch(
    Requests,
    Defaults(Requests),
    {
        Title: Trim(txtTitle.Value),
        Amount: Value(txtAmount.Value)
    }
)
```

Creates a record. Updating uses an existing base record instead of `Defaults`.

### `IfError`

```powerfx
IfError(
    Patch(Requests, Defaults(Requests), {Title: Trim(txtTitle.Value)}),
    Notify("Save failed.", NotificationType.Error),
    Notify("Saved.", NotificationType.Success)
)
```

Handles a specific failing expression. `App.OnError` observes otherwise unhandled errors but does not replace the failed result.

## 3. Gallery formulas

### Basic data

`galRequests.Items`:

```powerfx
Requests
```

Inside the template:

```powerfx
ThisItem.Title
```

### Filter and sort

```powerfx
SortByColumns(
    Filter(Requests, Status = "Submitted"),
    "RequestedOn",
    SortOrder.Descending
)
```

### Search

```powerfx
Filter(
    Requests,
    IsBlank(txtSearch.Value) ||
    StartsWith(Title, txtSearch.Value)
)
```

`StartsWith` is a common delegable pattern for supported sources. Delegation depends on connector, function, column type, and exact expression.

### Current user personalization

```powerfx
Filter(
    Requests,
    RequestedByEmailNormalized = Lower(User().Email)
)
```

Store source email normalized if transforming the source column causes delegation problems. This filter is still not authorization.

### Gallery select

```powerfx
Set(varSelectedRequestId, ThisItem.RequestId);
Navigate(scrRequestDetail, ScreenTransition.Fade)
```

Prefer a stable ID over depending indefinitely on `Gallery.Selected` after a refresh/re-sort.

## 4. Form lifecycle

| Intent | Formula |
|---|---|
| Start create | `NewForm(frmRequest); Navigate(scrEdit)` |
| Start edit | `EditForm(frmRequest); Navigate(scrEdit)` |
| Read only | `ViewForm(frmRequest)` |
| Submit | `SubmitForm(frmRequest)` |
| Cancel/reset | `ResetForm(frmRequest); Back()` |

`btnSave.OnSelect`:

```powerfx
If(
    frmRequest.Valid && !locIsSaving,
    UpdateContext({locIsSaving: true});
    SubmitForm(frmRequest),
    Notify("Complete required fields.", NotificationType.Error)
)
```

`frmRequest.OnSuccess`:

```powerfx
UpdateContext({locIsSaving: false});
Set(varSavedRequest, frmRequest.LastSubmit);
Notify("Request saved.", NotificationType.Success);
Navigate(scrRequestDetail)
```

`frmRequest.OnFailure`:

```powerfx
UpdateContext({locIsSaving: false});
Notify(
    Coalesce(frmRequest.Error, "The request could not be saved."),
    NotificationType.Error
)
```

Interview point: navigate after success, not immediately after `SubmitForm`.

## 5. `Patch` patterns

### Create and keep returned record

```powerfx
With(
    {
        saved:
            Patch(
                Requests,
                Defaults(Requests),
                {
                    Title: Trim(txtTitle.Value),
                    Amount: Value(txtAmount.Value),
                    RequestedOn: Now()
                }
            )
    },
    Set(varSelectedRequestId, saved.RequestId)
)
```

### Update one record

```powerfx
Patch(
    Requests,
    LookUp(Requests, RequestId = varSelectedRequestId),
    {Title: Trim(txtTitle.Value)}
)
```

### Update current gallery record

```powerfx
Patch(Requests, ThisItem, {Status: "Submitted"})
```

Only use this when `ThisItem` is the correct authoritative source record and the state transition is authorized server-side.

### Choice and lookup

- A Choice expects its typed option/record, not arbitrary display text.
- A lookup expects a record from the related table, such as `cmbDepartment.Selected`.
- Use Studio autocomplete because exact generated enum and column names vary.

## 6. Variables and collections

| Need | Best first choice |
|---|---|
| Calculated UI value | Direct formula or named formula |
| Across screens | Global variable with `Set` |
| One screen/modal | Context variable with `UpdateContext` |
| Temporary rows/draft lines | Collection |
| Durable business record | Data source |

### Collection operations

```powerfx
ClearCollect(colStatuses, ["Draft", "Submitted", "Approved"])
```

```powerfx
Collect(colDraftLines, {LocalId: GUID(), Description: "Monitor", Quantity: 1})
```

```powerfx
Patch(colDraftLines, ThisItem, {Quantity: ThisItem.Quantity + 1})
```

```powerfx
Remove(colDraftLines, ThisItem)
```

Do not say “collections are databases.” They live in app memory unless explicitly persisted through supported local/data-source mechanisms.

## 7. Text, number, date, and aggregate essentials

```powerfx
Trim(txtTitle.Value)
Lower(User().Email)
Value(txtAmount.Value)
Text(ThisItem.Amount, "$#,##0.00")
Today()
Now()
DateAdd(Today(), 7, TimeUnit.Days)
DateDiff(ThisItem.RequestedOn, Today(), TimeUnit.Days)
Sum(colDraftLines, Quantity * UnitPrice)
CountRows(colDraftLines)
Coalesce(ThisItem.Comment, "No comment")
```

Formatting with `Text` returns text. Keep source values typed for calculations/filtering.

## 8. `With`, scope, and aliases

```powerfx
With(
    {
        cleanTitle: Trim(txtTitle.Value),
        amount: Value(txtAmount.Value)
    },
    If(
        IsBlank(cleanTitle),
        "Title is required",
        amount > 1000,
        "Director approval",
        "Ready"
    )
)
```

Nested table operations use `As`:

```powerfx
ForAll(
    colDraftLines As line,
    Patch(
        RequestLines,
        Defaults(RequestLines),
        {Description: line.Description, Quantity: line.Quantity}
    )
)
```

Know the syntax; also say that large row-by-row `ForAll` writes can be slow, throttled, partly successful, and better handled server-side/batched.

## 9. Delegation answer and formula review

Before accepting a source formula, ask:

1. Which connector?
2. Is each function/operator supported for that column type?
3. Is the source column wrapped in a transformation?
4. Does Studio show a warning?
5. Did we test with data beyond the row limit?

Bad “fix”:

```powerfx
ClearCollect(colEverything, LargeSource)
```

Better approach:

- Use delegable server filters/sorts.
- Normalize searchable keys at write time.
- Index source columns appropriately.
- Use source-native search/query capability.
- Narrow to a proven-small set before local processing.

## 10. Navigation and deep links

```powerfx
Navigate(scrDetail, ScreenTransition.Fade, {locRequestId: ThisItem.RequestId})
```

```powerfx
Back()
```

```powerfx
Param("requestId")
```

A deep-link ID is not authorization. Query the source and let data security enforce access.

## 11. Loading, error, and concurrency pattern

```powerfx
If(
    !locIsSaving,
    UpdateContext({locIsSaving: true});
    IfError(
        Set(
            varSaved,
            Patch(Requests, varRequest, {Title: Trim(txtTitle.Value)})
        );
        Notify("Saved.", NotificationType.Success),
        Notify("Save failed.", NotificationType.Error)
    );
    UpdateContext({locIsSaving: false})
)
```

For sensitive multi-user edits, also explain the concurrency policy: last-write-wins, conflict detection, immutable event rows, or state-based lock/validation.

## 12. Formula-reading method in an interview

When given a formula:

1. Identify the property and expected return type.
2. Identify the data source and row scope.
3. Read inside-out.
4. State side effects.
5. Check blank/type/error behavior.
6. Check delegation/performance.
7. Check authorization and concurrency separately.

## 13. Ten formula traps

1. `Filter` returns a table; `LookUp` returns one record/value.
2. `.Text` versus `.Value` differs between classic/modern controls.
3. SharePoint Choice/Person and Dataverse Choice/Lookup are structured types.
4. `SubmitForm` success belongs in form `OnSuccess`.
5. `Gallery.Selected` can become stale after data changes.
6. `Max()+1` is not a safe multi-user ID.
7. `Lower(SourceColumn)` may break delegation even if comparison logic is correct.
8. `Visible=false` does not secure an operation.
9. `ForAll` is not an efficient universal bulk-update tool.
10. Locale can change argument/chaining separators; rely on tenant Studio syntax/autocomplete.

## 14. Whiteboard drill

Write these without notes:

1. Status filter plus descending date sort.
2. Required title and numeric amount validation.
3. Create `Patch` with `IfError`.
4. New form and success navigation.
5. Global versus context variable example.
6. Current-user gallery and delegation caveat.

If you can explain every return type and risk, your formula level is sufficient for many junior interviews.

## Primary references

- [Power Fx formula reference](https://learn.microsoft.com/power-platform/power-fx/formula-reference-canvas-apps)
- [Tables and record scope](https://learn.microsoft.com/power-platform/power-fx/tables)
- [Patch](https://learn.microsoft.com/power-platform/power-fx/reference/function-patch)
- [Form functions](https://learn.microsoft.com/power-platform/power-fx/reference/function-form)
- [Delegation overview](https://learn.microsoft.com/power-apps/maker/canvas-apps/delegation-overview)

