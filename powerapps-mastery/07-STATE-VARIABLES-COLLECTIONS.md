# 07 — State, Variables, and Collections

## Goal

Choose the smallest appropriate state mechanism and prevent stale, hidden, or unnecessarily global app state.

## First principle: prefer formulas over stored state

If a value can be derived reliably, calculate it.

Prefer:

```powerfx
Sum(colDraftLines, Quantity * UnitPrice)
```

over repeatedly maintaining:

```powerfx
Set(varTotal, varTotal + newAmount)
```

Derived formulas stay consistent with their inputs. Stored state must be updated on every possible path.

## State options

| Mechanism | Scope | Created/changed with | Best use |
|---|---|---|---|
| Global variable | Entire app | `Set` | Current user profile, selected record ID, feature state needed across screens. |
| Context variable | One screen | `UpdateContext` or navigation context | Modal state, local edit mode, screen-specific flags. |
| Collection | Entire app session | `Collect`, `ClearCollect`, `Patch`, `Remove` | Temporary table, draft lines, small reference cache. |
| Named formula | Entire app, calculated | App formulas | Derived constants/configuration and dependency-driven values. |
| Control property | Local dependency | Direct formula | UI state that can be calculated from controls/data. |
| Persistent data | Across sessions/users | Data source write | Actual business records and durable preferences. |

## Global variables with Set

```powerfx
Set(varSelectedRequest, galRequests.Selected)
```

```powerfx
Set(varIsSaving, true)
```

Clear a global variable with:

```powerfx
Set(varSelectedRequest, Blank())
```

Use a global variable only when multiple screens genuinely need the value.

## Context variables

```powerfx
UpdateContext({locShowDeleteDialog: true})
```

```powerfx
UpdateContext({locShowDeleteDialog: false, locDeleteReason: Blank()})
```

Pass navigation context:

```powerfx
Navigate(
    scrRequestDetail,
    ScreenTransition.Fade,
    {locRequest: ThisItem}
)
```

This makes `locRequest` available on the target screen. For long-lived editing, consider passing/storing a stable ID and re-querying the authoritative record to reduce staleness.

## Collections

Create/replace a collection:

```powerfx
ClearCollect(
    colRequestTypes,
    {Key: "Equipment", Label: "Equipment"},
    {Key: "Training", Label: "Training"},
    {Key: "Travel", Label: "Travel"}
)
```

Add a record:

```powerfx
Collect(
    colDraftLines,
    {
        LocalId: GUID(),
        Description: Trim(txtLineDescription.Value),
        Quantity: Value(txtQuantity.Value),
        UnitPrice: Value(txtUnitPrice.Value)
    }
)
```

Update a local record:

```powerfx
Patch(
    colDraftLines,
    LookUp(colDraftLines, LocalId = varEditingLineId),
    {Quantity: Value(txtQuantity.Value)}
)
```

Remove:

```powerfx
Remove(colDraftLines, ThisItem)
```

Collections are untyped initially in some creation paths and their schema emerges from records. Create consistent fields and types.

## Loading state correctly

```powerfx
UpdateContext({locIsLoading: true, locLoadError: Blank()});
IfError(
    ClearCollect(
        colMyRecentRequests,
        FirstN(
            SortByColumns(
                Filter(Requests, RequestedByEmail = User().Email),
                "RequestedOn",
                SortOrder.Descending
            ),
            20
        )
    ),
    UpdateContext({locLoadError: "Requests could not be loaded."})
);
UpdateContext({locIsLoading: false})
```

Do not cache an entire large table. Query a narrow, delegable subset. `FirstN` behavior and delegation depend on the source; verify the query plan and warnings.

## Save button double-submit guard

```powerfx
If(
    !locIsSaving,
    UpdateContext({locIsSaving: true});
    IfError(
        SubmitForm(frmRequest),
        Notify("Save failed.", NotificationType.Error);
        UpdateContext({locIsSaving: false})
    )
)
```

Set button display mode:

```powerfx
If(locIsSaving, DisplayMode.Disabled, DisplayMode.Edit)
```

Set `frmRequest.OnSuccess` to reset the flag and navigate. Set `OnFailure` to reset it and show the form error. The final architecture is in Lesson 08.

## Avoid state initialized in App.OnStart when unnecessary

`App.OnStart` is behavior-based and can delay startup if it performs network calls. Alternatives:

- `App.StartScreen` for declarative start-screen choice.
- Named formulas for calculated configuration.
- `Screen.OnVisible` for screen-specific, justified loading.
- Direct delegable gallery queries.

Do not create the same collection every time a user revisits a screen unless that reset is intentional.

## Offline pattern overview

Offline capability is a design project, not a single `SaveData` call. You must decide:

- Which data can be stored on the device.
- Sensitivity and device policy.
- Maximum dataset size.
- Change tracking and conflict resolution.
- Retry behavior and user feedback.
- Connector/client support.

For basic supported clients, `SaveData` and `LoadData` can persist local collections, but browser behavior and platform restrictions differ. Dataverse also offers dedicated mobile offline capabilities in supported scenarios. Validate current platform documentation before committing to an architecture.

## State inventory exercise

For each state item, record owner, scope, initialization, update paths, reset, and persistence.

| State | Scope | Initialized | Reset | Persistent? |
|---|---|---|---|---|
| `locShowDeleteDialog` | Detail screen | Default false | After cancel/delete | No |
| `varSelectedRequestId` | App | When opening detail | After close/sign-out | No |
| `colDraftLines` | App session | New/edit start | Save/cancel | No |
| Request record | Dataverse | User save | Retention process | Yes |

If you cannot explain when state resets, the design is incomplete.

## Common mistakes

- Using global variables for every visual condition.
- Copying a source table into a collection and forgetting changes made by other users.
- Storing a complete record for hours instead of refreshing by ID before a sensitive update.
- Using collections as a security boundary.
- Updating a derived total manually on some but not all paths.
- Leaving saving/loading flags true after an error.

## Interview questions

**What is the difference between `Set` and `UpdateContext`?**  
`Set` creates/updates an app-wide global variable. `UpdateContext` creates/updates screen-scoped context variables.

**When would you use a collection?**  
For temporary tabular state such as draft line items, user selections, or a small justified cache. Not as an automatic replacement for querying the source.

**Why prefer named formulas or direct formulas?**  
They are dependency-driven and reduce manual synchronization and startup behavior.

## Challenge

A dialog is shown only on `scrRequestDetail`. Should its visibility be global, context, or calculated?

### Answer

Usually a context variable such as `locShowDeleteDialog`, because its scope is one screen and its value changes through local user actions. If visibility can be derived directly from another state, calculate it instead.

## Primary references

- [Variables in canvas apps](https://learn.microsoft.com/power-apps/maker/canvas-apps/working-with-variables)
- [Set function](https://learn.microsoft.com/power-platform/power-fx/reference/function-set)
- [UpdateContext and Navigate](https://learn.microsoft.com/power-platform/power-fx/reference/function-navigate)
- [Collect and ClearCollect](https://learn.microsoft.com/power-platform/power-fx/reference/function-clear-collect-clearcollect)

