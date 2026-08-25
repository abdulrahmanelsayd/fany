# 03 — Build Your First Canvas App

## Goal

Build a three-screen request tracker using local sample data, navigation, a form-like input experience, and basic validation.

This prototype is intentionally local. Lesson 05 replaces local data with a real data source.

## What you will build

```text
scrHome -> scrRequestList -> scrRequestNew
              ^                  |
              |______ Save ______|
```

## Step 1: create and name the screens

Open the app from Lesson 02.

1. In Tree View, keep `scrHome`.
2. Select **New screen** and choose a blank or responsive layout.
3. Rename it `scrRequestList`.
4. Add another blank screen and rename it `scrRequestNew`.

Add a title label to each screen. Use `Parent.Width` for its width or place it in a full-width container.

## Step 2: create temporary records

Select **App** in Tree View and choose the `OnStart` property.

```powerfx
ClearCollect(
    colRequests,
    {
        RequestId: 1001,
        Title: "Replacement keyboard",
        RequestType: "Equipment",
        Status: "Submitted",
        Amount: 45,
        RequestedOn: Today()
    },
    {
        RequestId: 1002,
        Title: "Power Apps training",
        RequestType: "Training",
        Status: "Approved",
        Amount: 300,
        RequestedOn: Today() - 7
    }
)
```

Run `OnStart` using **App > Run OnStart** when that command is available, or close and reopen preview. Confirm `colRequests` appears in the Variables/Collections panel.

Why use `ClearCollect` here? It creates a table in memory and makes the prototype independent from an external service. It is not a recommended technique for downloading an entire large production table.

## Step 3: configure home navigation

Select `btnNewRequest` and set `OnSelect`:

```powerfx
Navigate(scrRequestNew, ScreenTransition.Cover)
```

Add a second button called `btnViewRequests` and set `OnSelect`:

```powerfx
Navigate(scrRequestList, ScreenTransition.Fade)
```

## Step 4: build the list screen

1. Open `scrRequestList`.
2. Select **Insert > Gallery** and choose a vertical gallery.
3. Rename it `galRequests`.
4. Set `Items` to:

```powerfx
SortByColumns(colRequests, "RequestedOn", SortOrder.Descending)
```

5. Choose a gallery layout containing a title and subtitle, or insert labels inside its template.
6. Set the title label to `ThisItem.Title`.
7. Set another label to:

```powerfx
ThisItem.Status & " • " & Text(ThisItem.RequestedOn, "mmm d, yyyy")
```

8. Add a button above the gallery named `btnListNew` with:

```powerfx
Navigate(scrRequestNew, ScreenTransition.Cover)
```

9. Add a back icon named `icoListBack` with:

```powerfx
Back()
```

`ThisItem` means the record currently being rendered by the gallery template.

## Step 5: build the new-request screen

Place the controls in a vertical container and rename them:

| Control | Name | Purpose |
|---|---|---|
| Text input | `txtRequestTitle` | Short request title. |
| Dropdown | `drpRequestType` | Request category. |
| Number input or text input | `txtAmount` | Estimated amount. |
| Button | `btnSaveRequest` | Validate and create. |
| Button or icon | `btnCancelRequest` | Return without saving. |
| Label | `lblValidation` | Accessible error summary. |

Set the request-type control items:

```powerfx
["Equipment", "Training", "Travel", "Other"]
```

For modern controls, read input using properties such as `Value` or `Selected.Value`. For classic text inputs, use `Text`. The formulas below assume `txtRequestTitle.Value`, `txtAmount.Value`, and `drpRequestType.Selected.Value`; adjust only the property names if your control variant differs.

Set `btnSaveRequest.OnSelect`:

```powerfx
If(
    IsBlank(Trim(txtRequestTitle.Value)),
    Notify("Enter a request title.", NotificationType.Error),
    !IsNumeric(txtAmount.Value) || Value(txtAmount.Value) < 0,
    Notify("Amount must be zero or a positive number.", NotificationType.Error),
    Collect(
        colRequests,
        {
            RequestId: Max(colRequests, RequestId) + 1,
            Title: Trim(txtRequestTitle.Value),
            RequestType: drpRequestType.Selected.Value,
            Status: "Draft",
            Amount: Value(txtAmount.Value),
            RequestedOn: Today()
        }
    );
    Reset(txtRequestTitle);
    Reset(drpRequestType);
    Reset(txtAmount);
    Notify("Draft request created.", NotificationType.Success);
    Navigate(scrRequestList, ScreenTransition.UnCover)
)
```

Read it as:

1. If the title is blank, show an error.
2. Otherwise, if amount is invalid, show an error.
3. Otherwise, collect a record, reset inputs, notify, and navigate.

The semicolon chains behavior actions. Commas separate function arguments in locales that use the English formula separators. Your tenant's authoring locale can change separators.

Set `btnCancelRequest.OnSelect`:

```powerfx
Reset(txtRequestTitle);
Reset(drpRequestType);
Reset(txtAmount);
Back()
```

## Step 6: make validation visible and accessible

Set `lblValidation.Text`:

```powerfx
If(
    IsBlank(Trim(txtRequestTitle.Value)),
    "Title is required.",
    !IsNumeric(txtAmount.Value) || Value(txtAmount.Value) < 0,
    "Enter a valid non-negative amount.",
    ""
)
```

Set `btnSaveRequest.DisplayMode`:

```powerfx
If(
    IsBlank(Trim(txtRequestTitle.Value)) ||
    !IsNumeric(txtAmount.Value) ||
    Value(txtAmount.Value) < 0,
    DisplayMode.Disabled,
    DisplayMode.Edit
)
```

Do not rely only on disabled button color. The label explains what the user must correct.

## Step 7: test systematically

Test these cases:

| Test | Expected result |
|---|---|
| Blank title | Error message; no record. |
| Spaces-only title | Treated as blank. |
| Negative amount | Error message; no record. |
| Non-numeric amount | Error message; no record. |
| Valid request | New record appears first or according to sort. |
| Cancel | Inputs reset and no record added. |
| Browser refresh | Local prototype returns to `OnStart` sample data. |

That last behavior is expected: collections are in-memory app state, not durable storage.

## Step 8: save and publish

1. Select **Save**.
2. Add version notes such as `Completed local request prototype` if prompted.
3. Select **Publish** and confirm publication.
4. Return to the Apps page and play the app.

## Common mistakes

- Expecting `Collect` into a local collection to persist after the session.
- Using `Max(...)+1` as a production identifier generator. Concurrent users can produce duplicates; use Dataverse's unique identifier or autonumber.
- Navigating before the data action completes.
- Validating only through color.
- Mixing control variants without checking whether the input property is `Text` or `Value`.
- Placing data initialization on every screen's `OnVisible`, causing unwanted resets.

## Interview questions

**What is the difference between `Navigate` and `Back`?**  
`Navigate` targets a specific screen and can pass context; `Back` returns to the previous screen in navigation history.

**What is a collection?**  
An in-memory table that the app can create and modify. It is useful for temporary state, staged data, and small reference caches, not automatically persistent storage.

**Why is `Max()+1` unsafe for a production ID?**  
Two users can read the same maximum and generate the same next value. Let the database generate a unique or sequential key.

## Challenge

Add a status filter above the gallery with `All`, `Draft`, `Submitted`, and `Approved`.

### Answer

Set a dropdown named `drpStatusFilter.Items`:

```powerfx
["All", "Draft", "Submitted", "Approved"]
```

Set `galRequests.Items`:

```powerfx
SortByColumns(
    Filter(
        colRequests,
        drpStatusFilter.Selected.Value = "All" ||
        Status = drpStatusFilter.Selected.Value
    ),
    "RequestedOn",
    SortOrder.Descending
)
```

## Primary references

- [Add and navigate screens](https://learn.microsoft.com/power-apps/maker/canvas-apps/add-screen-context-variables)
- [Create and update a collection](https://learn.microsoft.com/power-platform/power-fx/reference/function-clear-collect-clearcollect)
- [Notify function](https://learn.microsoft.com/power-platform/power-fx/reference/function-showerror)

