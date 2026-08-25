# 05 — Data Sources and Connectors

## Goal

Connect an app to external data, understand identity and permissions, and select an appropriate storage option.

## Four terms that must not be confused

| Term | Meaning | Example |
|---|---|---|
| Data source | The data you read or change | A SharePoint list named `Requests`. |
| Connector | Power Platform's interface to a service/API | SharePoint connector. |
| Connection | An authenticated connector instance | Alex's SharePoint connection. |
| Connection reference | Solution-aware pointer used by apps/flows | `cr_SharedSharePoint`. |

## Compare common data sources

| Capability | Excel | SharePoint Lists | Dataverse | SQL |
|---|---|---|---|---|
| Good for | Personal/small simple data | Team lists and document-centric work | Business applications | Existing enterprise relational systems |
| Relationships | Weak | Lookup-based, limited | Native rich relationships | Native relational model |
| Security | File permissions | Site/list/item controls | Role, row, team, column hierarchy | Database/service security |
| Multi-user transactions | Poor choice | Moderate scenarios | Strong | Strong |
| Delegation | Limited | Connector-specific | Strong for supported operations | Strong for supported operations |
| Model-driven apps | No | No | Yes | No, unless data is integrated/virtualized into Dataverse |
| Licensing | Often standard Microsoft 365 context | Often standard Microsoft 365 context | License-dependent | Premium in many Power Apps scenarios |

Always verify current licensing and connector classification for the tenant.

## Add a data source in Studio

Typical path:

1. Open the canvas app in edit mode.
2. Select **Data** on the left rail.
3. Select **Add data**.
4. Search for the service, such as SharePoint or Dataverse.
5. Select or create a connection.
6. Select the site/database and table/list.
7. Confirm that the data source appears in the Data panel.

If the data source has spaces, Power Fx normally uses single quotes:

```powerfx
'Employee Requests'
```

## Option A lab: Excel in OneDrive

Use the workbook prepared in Lesson 00.

1. In Studio, select **Data > Add data**.
2. Search for **Excel Online (Business)** or the current OneDrive/Excel connector path.
3. Select the connection.
4. Browse to the workbook.
5. Select the table `RequestsTable`.
6. Set `galRequests.Items` to:

```powerfx
RequestsTable
```

Excel limitations to remember:

- Data must be formatted as a named table.
- Workbook schema and file location must remain stable.
- Concurrent editing and transactional behavior are not database-grade.
- Delegation is limited; large-data behavior needs careful testing.
- A generated ID column should be unique if updates depend on it.

## Option B lab: SharePoint list

Create a list named `Employee Requests` with columns:

| Column | SharePoint type |
|---|---|
| Title | Single line of text |
| RequestType | Choice |
| Status | Choice |
| Amount | Currency |
| NeededBy | Date only |
| Description | Multiple lines of text |
| RequesterEmail | Single line of text |
| ManagerEmail | Single line of text |

Then connect it:

1. **Data > Add data > SharePoint**.
2. Choose the site.
3. Select `Employee Requests`.
4. Set the gallery `Items` property:

```powerfx
SortByColumns(
    'Employee Requests',
    "Created",
    SortOrder.Descending
)
```

Choice values are records. A SharePoint choice may be displayed through:

```powerfx
ThisItem.Status.Value
```

The precise record shape depends on the column and connector. Inspect a record with Studio, a temporary label, or the Collections/Variables view rather than guessing.

## CRUD concepts

CRUD means create, read, update, and delete.

| Operation | Common Power Fx / control path |
|---|---|
| Create | `SubmitForm` in New mode or `Patch(DataSource, Defaults(DataSource), ...)` |
| Read | Gallery/form using data source records |
| Update | `SubmitForm` in Edit mode or `Patch(DataSource, ExistingRecord, ...)` |
| Delete | `Remove(DataSource, ExistingRecord)` |

Lesson 08 implements these carefully.

## Refresh and stale data

Use:

```powerfx
Refresh('Employee Requests')
```

`Refresh` requests the latest data. Do not call it after every small interaction. It adds network work and can create a slow, flickering app.

Data written through `Patch` or `SubmitForm` returns a record/result you can often use immediately rather than refreshing the entire source.

## Connection identity and sharing

Ask these questions for every connector:

1. Does the app use each user's connection, a shared connection, or a service principal-supported pattern?
2. Does the user have permission to the underlying data?
3. Does a called flow run with the owner's connections or require run-only user connections?
4. Can the operation expose more data than the UI displays?

Never depend on filtering a gallery to secure records. A malicious or alternative client may reach the underlying data. Enforce security at the data layer.

## Data types matter

Common avoidable bugs come from comparing incompatible types:

```powerfx
Amount = "100"        // number compared with text: problematic
```

Prefer explicit conversion at system boundaries:

```powerfx
Amount = Value(txtAmount.Value)
```

```powerfx
Text(NeededBy, "yyyy-mm-dd")
```

Do not convert every database column to text just to make formulas easy. Preserve strong types.

## Error-aware writes

Basic pattern:

```powerfx
IfError(
    Patch(
        'Employee Requests',
        Defaults('Employee Requests'),
        {
            Title: Trim(txtRequestTitle.Value),
            Amount: Value(txtAmount.Value)
        }
    ),
    Notify("The request could not be saved. Try again.", NotificationType.Error),
    Notify("Request saved.", NotificationType.Success)
)
```

For production, store or log useful diagnostic context without showing raw sensitive connector messages to end users.

## Common mistakes

- Connecting to an Excel range instead of a table.
- Treating a SharePoint Choice or Person value as plain text without inspecting its schema.
- Sharing only the app.
- Using personal owner connections for a business-critical automation with no continuity plan.
- Refreshing repeatedly.
- Pulling all rows into a collection to avoid learning delegation.
- Assuming a successful UI filter is authorization.

## Interview questions

**How do you choose between SharePoint and Dataverse?**  
Consider data relationships, row and field security, transaction volume, business rules, model-driven requirements, auditing, ALM, integration, governance, and licensing. SharePoint is effective for list/document scenarios; Dataverse is designed as a business application data platform.

**Does sharing an app give a user access to SharePoint or Dataverse data?**  
No. App access and data access are separate concerns.

**When should you use `Refresh`?**  
When the app needs to request changes made outside its current data interaction. Avoid automatic repeated refreshes without a user or business reason.

## Challenge

Explain why this is dangerous in a large production app:

```powerfx
ClearCollect(colAllRequests, 'Employee Requests')
```

### Answer

It can retrieve only a limited nondelegable subset, increase startup time and memory use, become stale, and bypass efficient server querying. Query only the required rows and columns using delegable operations where possible.

## Primary references

- [Add data connections to canvas apps](https://learn.microsoft.com/power-apps/maker/canvas-apps/add-data-connection)
- [Connect to SharePoint from a canvas app](https://learn.microsoft.com/power-apps/maker/canvas-apps/connections/connection-sharepoint-online)
- [Connect to cloud-storage Excel](https://learn.microsoft.com/power-apps/maker/canvas-apps/connections/connection-cloud-storage-blob-connections)
- [Dataverse overview](https://learn.microsoft.com/power-apps/maker/data-platform/data-platform-intro)

