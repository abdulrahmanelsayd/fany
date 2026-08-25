# 08 — Galleries, Forms, and CRUD

## Goal

Build a reliable create/read/update/delete experience with forms and `Patch`, including validation, success/failure handling, and optimistic concurrency awareness.

## Gallery versus form

| Control | Responsibility |
|---|---|
| Gallery | Repeats a template for many records. |
| Display form | Shows one record read-only. |
| Edit form | Creates or edits one record using data cards. |

A gallery selects a record; a form displays or edits that record.

## Build the browse-detail-edit pattern

### Browse screen

Set `galRequests.Items` to a delegable query developed in Lesson 09. For now:

```powerfx
SortByColumns(Requests, "RequestedOn", SortOrder.Descending)
```

Set the gallery template `OnSelect` or detail icon `OnSelect`:

```powerfx
Set(varSelectedRequestId, ThisItem.Request);
Navigate(scrRequestDetail, ScreenTransition.Fade)
```

For Dataverse, the primary key may appear under the table's singular name or a schema-specific GUID column. Use Studio autocomplete.

### Detail screen

Insert a display form named `frmRequestView`.

Set:

```powerfx
frmRequestView.DataSource = Requests
```

```powerfx
frmRequestView.Item = LookUp(Requests, Request = varSelectedRequestId)
```

Add an edit button:

```powerfx
EditForm(frmRequestEdit);
Navigate(scrRequestEdit, ScreenTransition.Cover)
```

### New record button

```powerfx
NewForm(frmRequestEdit);
Set(varSelectedRequestId, Blank());
Navigate(scrRequestEdit, ScreenTransition.Cover)
```

### Edit form item

```powerfx
If(
    frmRequestEdit.Mode = FormMode.New,
    Defaults(Requests),
    LookUp(Requests, Request = varSelectedRequestId)
)
```

## Understand data cards

An edit form contains data cards. Each card typically has:

- `DataField`: source column mapping.
- `Default`: current source value.
- An input control.
- `Update`: value the form will submit.
- `Required`: whether the card is required.
- `Valid`: whether current input is valid.
- An error label.

Unlock a card only when you need customization. Once unlocked, you are responsible for maintaining its formulas as the form/schema changes.

## SubmitForm pattern

Set `btnSave.OnSelect`:

```powerfx
If(
    frmRequestEdit.Valid && !locIsSaving,
    UpdateContext({locIsSaving: true});
    SubmitForm(frmRequestEdit),
    Notify("Complete the required fields.", NotificationType.Error)
)
```

Set `frmRequestEdit.OnSuccess`:

```powerfx
UpdateContext({locIsSaving: false});
Set(varSelectedRequestId, frmRequestEdit.LastSubmit.Request);
Notify("Request saved.", NotificationType.Success);
ViewForm(frmRequestView);
Navigate(scrRequestDetail, ScreenTransition.UnCover)
```

Set `frmRequestEdit.OnFailure`:

```powerfx
UpdateContext({locIsSaving: false});
Notify(
    Coalesce(frmRequestEdit.Error, "The request could not be saved."),
    NotificationType.Error
)
```

Set cancel:

```powerfx
ResetForm(frmRequestEdit);
Back()
```

Why `OnSuccess` matters: `SubmitForm` is asynchronous from the UI's perspective. Navigate and use `LastSubmit` after the form reports success, not immediately after calling submit.

## When to use Patch

Use `Patch` when:

- The UI is not a standard edit form.
- You update a small subset of columns.
- You create/update related records in a controlled sequence.
- You need the returned saved record directly.

Create:

```powerfx
With(
    {
        savedRequest:
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
    Set(varSelectedRequestId, savedRequest.Request);
    Notify("Request created.", NotificationType.Success)
)
```

Update:

```powerfx
Patch(
    Requests,
    LookUp(Requests, Request = varSelectedRequestId),
    {Title: Trim(txtTitle.Value)}
)
```

Do not use `Patch` to avoid understanding form validation. With custom inputs, you own validation, required fields, type conversion, errors, accessibility, and reset behavior.

## Choice and lookup fields

Dataverse Choice patching often uses the generated option enum suggested by autocomplete, for example conceptually:

```powerfx
{'Request Status': 'Request Status (Requests)'.Submitted}
```

A Dataverse lookup expects a related table record:

```powerfx
{Department: cmbDepartment.Selected}
```

Do not patch only a display string into a lookup. Let Studio autocomplete reveal the expected type.

## Delete pattern

1. Check whether deletion is allowed by business state and security.
2. Ask for confirmation.
3. Delete the authoritative selected record.
4. Handle dependencies and errors.

```powerfx
IfError(
    Remove(
        Requests,
        LookUp(Requests, Request = varSelectedRequestId)
    ),
    Notify("Delete failed. The record may be in use or you may lack permission.", NotificationType.Error),
    Notify("Request deleted.", NotificationType.Success);
    Set(varSelectedRequestId, Blank());
    Navigate(scrRequestList, ScreenTransition.Fade)
)
```

Many business systems should use status-based cancellation rather than physical deletion for audit and recovery.

## Validation layers

| Layer | Example | Why |
|---|---|---|
| Control | Required label, date constraints | Fast user feedback. |
| Form/formula | Cross-field validation | Prevent invalid submission path. |
| Dataverse/business rule | Required columns and business rules | Applies beyond one canvas screen. |
| Flow/plugin/server | Process/integration rules | Protect cross-system operation. |

Example cross-field rule:

```powerfx
If(
    dpNeededBy.SelectedDate < Today(),
    Notify("Needed-by date cannot be in the past.", NotificationType.Error),
    Value(txtAmount.Value) > 1000 && IsBlank(cmbCostCenter.Selected),
    Notify("Cost center is required above 1,000.", NotificationType.Error),
    SubmitForm(frmRequestEdit)
)
```

## Concurrency

Two users can edit the same record. Decide what should happen:

- Last write wins.
- Detect change and ask the user to refresh/review.
- Lock through a business process.
- Allow updates only in certain statuses.

Power Apps can surface conflicts through errors for supported sources. Design an explicit user experience; do not silently overwrite sensitive approvals.

## Attachments

Attachment controls are commonly used inside supported forms for SharePoint and Dataverse scenarios. Test:

- File size/count limits.
- Mobile selection behavior.
- Malware/compliance policy.
- Save ordering: parent record may need to exist before related file operations.
- User permission to the underlying storage.

## Common mistakes

- Calling `Navigate` immediately after `SubmitForm` instead of in `OnSuccess`.
- Patching text into a Choice or Lookup field.
- Using `Gallery.Selected` after the gallery has re-sorted or filtered; a stable ID is safer.
- Deleting without confirmation or audit consideration.
- Unlocking every data card and creating unnecessary maintenance.
- Assuming form validity covers all cross-field business rules.

## Interview questions

**`SubmitForm` or `Patch`?**  
Use `SubmitForm` for a form-driven create/edit experience with built-in cards, validation, and success/failure events. Use `Patch` for custom or targeted writes when you intentionally own validation and error handling.

**What is `LastSubmit`?**  
The record successfully submitted most recently by the form, including server-generated values. Use it in or after the form's success path.

**How do you patch a lookup?**  
Pass a record from the related table, not merely its display text.

## Challenge

Prevent editing after a request is approved.

### Answer

Enforce it at multiple layers. UI example:

```powerfx
btnEdit.DisplayMode =
    If(
        frmRequestView.Item.Status = 'Request Status'.Approved,
        DisplayMode.Disabled,
        DisplayMode.Edit
    )
```

Also enforce authorization/business state in Dataverse or server-side logic; the UI alone is not security.

## Primary references

- [Edit form and Display form controls](https://learn.microsoft.com/power-apps/maker/canvas-apps/controls/control-form-detail)
- [SubmitForm and form functions](https://learn.microsoft.com/power-platform/power-fx/reference/function-form)
- [Patch function](https://learn.microsoft.com/power-platform/power-fx/reference/function-patch)
- [Errors and error handling](https://learn.microsoft.com/power-platform/power-fx/reference/function-errors)

