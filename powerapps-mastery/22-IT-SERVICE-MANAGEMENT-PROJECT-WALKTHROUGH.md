# 22 — IT Service Management & Workflow Automation Project

> A complete beginner-friendly build and interview-defense guide for the project described in the CV

Technology: **Power Apps + Power Automate + SharePoint**

## 1. What the project actually does

The system is an internal IT help desk. Instead of employees sending random emails or WhatsApp messages to the IT team, everyone uses one application.

An employee can:

- Submit an IT request.
- Select a category and priority.
- Attach a screenshot.
- See the request number, assigned agent, and current status.
- Follow the request until it is resolved.

An IT agent can:

- See the support queue.
- Filter requests by category, priority, status, or assignee.
- Assign or reassign a request.
- Update status.
- Add a resolution.
- Close the request.

Power Automate can:

- Generate a request number.
- Assign the request based on its category.
- Calculate a due date from the category SLA.
- Send creation, assignment, approval, status, and resolution notifications.
- Request approval for selected categories.
- Send scheduled overdue reminders.
- Write important changes to a history list.

## 2. The CV bullets translated into technical work

### “Built a Power Apps solution for submitting, categorizing, assigning, and tracking requests”

This means a canvas app was created with employee and IT-agent screens. The app reads and updates SharePoint records using galleries, forms, filters, `SubmitForm`, and `Patch`.

### “Used SharePoint Lists as the data source”

This means SharePoint stores the real records after the app closes. A main list stores requests; smaller configuration/history lists store categories, support agents, and important events.

### “Developed Power Automate cloud flows”

This means the app is not responsible for every background process. Flows react when a SharePoint item is created or modified, start approvals, send notifications, and perform scheduled reminders.

### “Documented application and workflow logic”

This means the solution has a data dictionary, screen list, formula/flow explanation, security assumptions, error behavior, test cases, ownership, and troubleshooting notes. This file is an example of that documentation.

## 3. The system in one picture

```text
Employee
   |
   v
Power Apps canvas app -------------------------+
   |                                           |
   | create/read/update                        | agent queue/update
   v                                           v
SharePoint: IT Service Requests <--------- IT Support Agent
   |             |              |
   |             |              +--> IT Request History
   |             +-----------------> IT Categories / IT Support Agents
   |
   v
Power Automate
   ├── New-request assignment and notification
   ├── Conditional approval
   ├── Status-change notification/history
   └── Scheduled overdue follow-up
```

The most important design idea is separation of responsibility:

- **Power Apps:** interactive user interface.
- **SharePoint:** persistent data.
- **Power Automate:** background process and notifications.

## 4. Terms you must understand first

| Term | Simple meaning in this project |
|---|---|
| Canvas app | The screens and buttons the employee/agent uses. |
| SharePoint list | A cloud table containing rows called items and fields called columns. |
| Data source | The list connected to the app. |
| Connector | The Power Platform interface to SharePoint, Outlook, Approvals, and other services. |
| Gallery | A Power Apps control that repeats a design for every request record. |
| Form | A Power Apps control used to display, create, or edit one request. |
| Power Fx | The formula language used in control properties. |
| Flow | A Power Automate process triggered by an event or schedule. |
| Trigger | The event that starts a flow. |
| Action | A step inside a flow, such as Send an email or Update item. |
| Choice column | A controlled list of values such as Low, Medium, High. |
| Person column | A SharePoint column that stores a Microsoft 365 user. |
| SLA | The target time allowed for the IT team to respond or resolve. |
| Delegation | Letting SharePoint evaluate a filter/sort over the complete list. |

## 5. Requirements used for the build

### User roles

1. **Requester:** employee who submits and tracks personal requests.
2. **IT Agent:** person who handles requests.
3. **IT Manager:** can review all work, approve selected requests, reassign, and view metrics.

### Request lifecycle

```text
New
  -> Pending Approval (only when required)
  -> Assigned
  -> In Progress
  -> Waiting for User
  -> Resolved
  -> Closed

Alternative endings: Rejected or Cancelled
```

### Example business rules

- Title, description, category, and priority are required.
- New requests receive a unique number such as `SR-000125`.
- Category decides default IT agent and SLA hours.
- Access and Purchase requests require approval.
- A request cannot be resolved without a resolution description.
- Requesters see their own requests; IT agents see the support queue according to permissions.
- Overdue active requests generate reminders.

## 6. Phase 1 — Create the SharePoint data model

Use a dedicated SharePoint site such as `IT Service Desk - Training`. Do not build the demo in an important production site.

### 6.1 Create the main list

1. Open the SharePoint site.
2. Select **Site contents**.
3. Select **New > List**.
4. Select **Blank list**.
5. Name it `IT Service Requests`.
6. Select **Create**.

Keep SharePoint's built-in `ID`, `Created`, `Modified`, `Created By`, and `Modified By` columns.

Create these columns using **Add column**:

| Display name | Type | Configuration/purpose |
|---|---|---|
| Title | Single line of text | Short issue summary; required. |
| RequestNumber | Single line of text | Flow writes `SR-000001`; make unique later if governance requires. |
| Description | Multiple lines of text | Full problem description; plain text is simplest. |
| Category | Choice | Hardware, Software, Network, Access, Purchase, Other. |
| Priority | Choice | Low, Medium, High, Critical; default Medium. |
| Status | Choice | New, Pending Approval, Assigned, In Progress, Waiting for User, Resolved, Closed, Rejected, Cancelled; default New. |
| RequesterEmail | Single line of text | Lowercase email written by the app for filtering/integration. |
| AssignedTo | Person | IT agent; allow one person. |
| DueDate | Date and time | Calculated by the assignment flow. |
| ApprovalRequired | Yes/No | Default No. |
| ApprovalStarted | Yes/No | Default No; prevents duplicate approvals. |
| ApprovalOutcome | Single line of text | Approved, Rejected, or blank. |
| Resolution | Multiple lines of text | Required by app logic before Resolved. |
| ResolvedOn | Date and time | Written when status becomes Resolved. |
| LastReminderDate | Date only | Helps prevent repeated same-day reminders. |

Enable attachments:

1. In the list, select **Settings > List settings**.
2. Select **Advanced settings**.
3. Confirm attachments are enabled.

Why keep both `Created By` and `RequesterEmail`? `Created By` is SharePoint's audit identity. `RequesterEmail` is a normalized text value that simplifies app filtering and flow integration. It must not be trusted as the only authorization control.

### 6.2 Create the category configuration list

Create a blank list named `IT Categories`:

| Column | Type | Example |
|---|---|---|
| Title | Single line | Hardware |
| DefaultAssignee | Person | hardware.agent@company.com |
| SLAHours | Number | 24 |
| NeedsApproval | Yes/No | No |
| Active | Yes/No | Yes |

Example rows:

| Title | SLA hours | Needs approval |
|---|---:|---:|
| Hardware | 24 | No |
| Software | 24 | No |
| Network | 4 | No |
| Access | 8 | Yes |
| Purchase | 72 | Yes |
| Other | 48 | No |

The advantage of this list is that assignment and SLA configuration can change without editing the app or flow logic.

### 6.3 Create the history list

Create `IT Request History`:

| Column | Type | Purpose |
|---|---|---|
| Title | Single line | Event summary. |
| Request | Lookup to IT Service Requests | The related request item. |
| RequestNumber | Single line | Easy display/search. |
| EventType | Choice | Created, Assigned, Approval, Status Changed, Reminder, Resolved. |
| FromStatus | Single line | Previous status when available. |
| ToStatus | Single line | New status. |
| Comment | Multiple lines | Event detail. |
| PerformedByEmail | Single line | User/flow identity responsible. |

SharePoint already has Created/Created By, so do not create duplicate event date columns unless the process requires a different timestamp.

### 6.4 Create the support-agent list

Create `IT Support Agents`:

| Column | Type |
|---|---|
| Title | Single line; agent name |
| Email | Single line; lowercase and unique by process |
| Role | Choice: Agent, Manager |
| Active | Yes/No |

The app can use this small list to tailor navigation. It is not the final security boundary.

### 6.5 Internal column names

SharePoint assigns an internal name when a column is created. Renaming its display name later usually does not change the internal name. Power Automate filters and integrations may expose internal names such as `AssignedTo` or encoded names.

Create columns with clean names first and record their internal names in documentation. This prevents mysterious flow filter errors later.

## 7. Phase 2 — Create the canvas app

### 7.1 Create a blank app

1. Open `https://make.powerapps.com`.
2. Confirm the correct development environment in the upper-right selector.
3. Select **Create**.
4. Choose a blank canvas app/start with blank design.
5. Name it `IT Service Desk`.
6. Select **Tablet** for this desktop-oriented demo.
7. Select **Create**.

To support different widths:

1. Open **Settings > Display**.
2. Turn off **Scale to fit**.
3. Turn off **Lock aspect ratio** if required.
4. Use vertical/horizontal auto-layout containers instead of fixed pixel placement.

### 7.2 Connect the data

1. Select **Data** on the left rail.
2. Select **Add data**.
3. Search for **SharePoint**.
4. Select/create the connection.
5. Select the SharePoint site.
6. Select:
   - `IT Service Requests`
   - `IT Categories`
   - `IT Request History`
   - `IT Support Agents`
7. Select **Connect**.

### 7.3 Initialize small user state

Select **App** in Tree View and select `OnStart`:

```powerfx
Set(varUserEmail, Lower(User().Email));
Set(
    varAgentProfile,
    LookUp(
        'IT Support Agents',
        Email = varUserEmail && Active = true
    )
);
Set(varIsAgent, !IsBlank(varAgentProfile));
Set(varIsManager, varAgentProfile.Role.Value = "Manager")
```

Run **App > Run OnStart** while developing if available.

Why this is acceptable here: `IT Support Agents` is a tiny configuration list. Do not download the large request list during startup.

Security warning: these variables only control the experience. SharePoint permissions must still protect the data.

## 8. Phase 3 — Build the screens

Create and rename:

| Screen | User | Purpose |
|---|---|---|
| `scrHome` | All | Welcome, navigation, summary. |
| `scrMyRequests` | Requester | Personal request gallery. |
| `scrNewRequest` | Requester | New request form. |
| `scrRequestDetail` | All authorized users | One request and history. |
| `scrAgentQueue` | Agent/Manager | IT work queue. |
| `scrAgentUpdate` | Agent/Manager | Assignment/status/resolution update. |

Each screen should use:

```text
conPage (vertical)
├── conHeader
├── conContent
└── conFooter (only if needed)
```

### 8.1 Home screen

Add buttons:

`btnNewRequest.OnSelect`:

```powerfx
NewForm(frmNewRequest);
Navigate(scrNewRequest, ScreenTransition.Cover)
```

`btnMyRequests.OnSelect`:

```powerfx
Navigate(scrMyRequests, ScreenTransition.Fade)
```

`btnAgentQueue.Visible`:

```powerfx
varIsAgent
```

`btnAgentQueue.OnSelect`:

```powerfx
Navigate(scrAgentQueue, ScreenTransition.Fade)
```

Add a label:

```powerfx
"Welcome, " & User().FullName
```

For a small demo, summary labels can use `CountRows(Filter(...))`. In a large SharePoint list, counts/aggregates can be nondelegable or slow. Production dashboards may use prepared metrics, Power Automate, a SharePoint view, or Power BI.

### 8.2 My Requests screen

Insert:

- Search text input `txtMySearch`.
- Status dropdown `drpMyStatus`.
- Vertical gallery `galMyRequests`.
- New and Back buttons.

`drpMyStatus.Items`:

```powerfx
["All", "New", "Pending Approval", "Assigned", "In Progress", "Waiting for User", "Resolved", "Closed"]
```

`galMyRequests.Items`:

```powerfx
SortByColumns(
    Filter(
        'IT Service Requests',
        RequesterEmail = varUserEmail &&
        (drpMyStatus.Selected.Value = "All" || Status.Value = drpMyStatus.Selected.Value) &&
        (IsBlank(txtMySearch.Value) || StartsWith(Title, txtMySearch.Value))
    ),
    "Created",
    SortOrder.Descending
)
```

Use `.Text` instead of `.Value` if you inserted a classic text input. Let Studio autocomplete confirm control properties.

Inside the gallery display:

```powerfx
ThisItem.RequestNumber
```

```powerfx
ThisItem.Title
```

```powerfx
ThisItem.Status.Value & " • " & Text(ThisItem.Created, "mmm d, yyyy")
```

Gallery template/detail icon `OnSelect`:

```powerfx
Set(varSelectedRequestId, ThisItem.ID);
Navigate(scrRequestDetail, ScreenTransition.Fade)
```

Delegation interview point: equality and `StartsWith` can be delegable for supported SharePoint columns, but the exact full expression and Choice/Person behavior must be checked against current SharePoint delegation documentation and tested beyond the row limit.

### 8.3 New Request screen

1. Select **Insert > Forms > Edit form**.
2. Rename it `frmNewRequest`.
3. Set `DataSource` to `'IT Service Requests'`.
4. Set `DefaultMode` to `FormMode.New`.
5. Add fields:
   - Title
   - Description
   - Category
   - Priority
   - Attachments
   - RequesterEmail
   - Status
6. Keep RequesterEmail and Status cards hidden but set their values.

Unlock the RequesterEmail card only and set its `Update`:

```powerfx
varUserEmail
```

Unlock Status card only and set `Update` to the typed SharePoint Choice record:

```powerfx
{Value: "New"}
```

`btnSubmitRequest.OnSelect`:

```powerfx
If(
    locIsSubmitting,
    false,
    !frmNewRequest.Valid,
    Notify("Complete all required fields.", NotificationType.Error),
    UpdateContext({locIsSubmitting: true});
    SubmitForm(frmNewRequest)
)
```

`frmNewRequest.OnSuccess`:

```powerfx
UpdateContext({locIsSubmitting: false});
Set(varSelectedRequestId, frmNewRequest.LastSubmit.ID);
Notify("Request submitted successfully.", NotificationType.Success);
ResetForm(frmNewRequest);
Navigate(scrRequestDetail, ScreenTransition.UnCover)
```

`frmNewRequest.OnFailure`:

```powerfx
UpdateContext({locIsSubmitting: false});
Notify(
    Coalesce(frmNewRequest.Error, "The request could not be submitted."),
    NotificationType.Error
)
```

`btnCancel.OnSelect`:

```powerfx
ResetForm(frmNewRequest);
Back()
```

Why use a form instead of `Patch`? Attachments and standard field validation are easier in a supported SharePoint form. `Patch` is useful for targeted/custom updates, but you own more validation and type mapping.

### 8.4 Request Detail screen

Insert a Display Form named `frmRequestDetail`:

```powerfx
DataSource = 'IT Service Requests'
```

```powerfx
Item = LookUp('IT Service Requests', ID = varSelectedRequestId)
```

Insert a history gallery `galHistory`:

```powerfx
SortByColumns(
    Filter('IT Request History', Request.Id = varSelectedRequestId),
    "Created",
    SortOrder.Descending
)
```

Whether the SharePoint lookup appears as `Request.Id`, `Request.ID`, or another generated name should be confirmed with Studio autocomplete.

Requesters should not edit a request after workflow submission unless the process explicitly allows it. A Cancel button can be visible only for selected states:

```powerfx
frmRequestDetail.Item.RequesterEmail = varUserEmail &&
frmRequestDetail.Item.Status.Value in ["New", "Assigned"]
```

But cancellation must also be permitted by SharePoint permissions/process logic; UI visibility is not security.

### 8.5 Agent Queue screen

Add:

- `txtAgentSearch`
- `drpAgentStatus`
- `drpPriority`
- `tglOnlyMine`
- `galAgentQueue`

Example `galAgentQueue.Items`:

```powerfx
SortByColumns(
    Filter(
        'IT Service Requests',
        (drpAgentStatus.Selected.Value = "All" || Status.Value = drpAgentStatus.Selected.Value) &&
        (drpPriority.Selected.Value = "All" || Priority.Value = drpPriority.Selected.Value) &&
        (!tglOnlyMine.Checked || AssignedTo.Email = varUserEmail) &&
        (IsBlank(txtAgentSearch.Value) || StartsWith(Title, txtAgentSearch.Value))
    ),
    "DueDate",
    SortOrder.Ascending
)
```

Person-field filters have connector-specific delegation behavior. For a large list, consider a normalized `AssignedToEmail` text column maintained by the flow, suitable indexes, and a query verified with Monitor/real volume.

Color overdue records only as reinforcement:

```powerfx
If(
    ThisItem.DueDate < Now() && !(ThisItem.Status.Value in ["Resolved", "Closed", "Rejected", "Cancelled"]),
    ColorValue("#C50F1F"),
    ColorValue("#242424")
)
```

Always show visible text such as `Overdue`; do not communicate only with red color.

### 8.6 Agent Update screen

Use either an Edit Form or deliberate custom controls. A form is easier for a beginner.

Set `frmAgentUpdate.Item`:

```powerfx
LookUp('IT Service Requests', ID = varSelectedRequestId)
```

Include AssignedTo, Status, and Resolution.

Save validation concept:

```powerfx
If(
    cmbStatus.Selected.Value = "Resolved" && IsBlank(Trim(txtResolution.Value)),
    Notify("A resolution is required before resolving the request.", NotificationType.Error),
    IfError(
        Patch(
            'IT Service Requests',
            LookUp('IT Service Requests', ID = varSelectedRequestId),
            {
                AssignedTo: cmbAssignedTo.Selected,
                Status: {Value: cmbStatus.Selected.Value},
                Resolution: Trim(txtResolution.Value)
            }
        ),
        Notify("The update failed.", NotificationType.Error),
        Notify("Request updated.", NotificationType.Success);
        Navigate(scrRequestDetail, ScreenTransition.UnCover)
    )
)
```

Exact Person-control record shape depends on the control's `Items` source. Use SharePoint-generated data card controls or Studio autocomplete instead of guessing.

## 9. Phase 4 — Build the Power Automate flows

Build flows in a development solution when possible. Use clear names:

1. `ITSM - New Request Assignment`
2. `ITSM - Request Approval`
3. `ITSM - Status Change Notification`
4. `ITSM - Overdue Reminder`

### 9.1 Flow 1 — New Request Assignment

#### Purpose

When a request is created:

- Generate request number.
- Retrieve category configuration.
- Set assignee, due date, approval requirement, and initial workflow status.
- Notify requester and agent.
- Create history.

#### Create it step by step

1. Open `https://make.powerautomate.com` or select **Flows** in the maker portal.
2. Select **Create > Automated cloud flow**.
3. Name: `ITSM - New Request Assignment`.
4. Trigger: **SharePoint — When an item is created**.
5. Select the Site Address and `IT Service Requests` list.
6. Add **SharePoint — Get items**.
7. Site: same site.
8. List: `IT Categories`.
9. Filter Query concept: find active category whose Title equals the trigger Category value.
10. Set **Top Count** to `1`.
11. Add a **Condition** checking whether one configuration record was returned.

Use a Compose or expression for request number:

```text
concat('SR-', formatNumber(triggerOutputs()?['body/ID'], '000000'))
```

Due date concept using the returned SLA hours:

```text
addHours(utcNow(), int(first(body('Get_items')?['value'])?['SLAHours']))
```

Use dynamic content/expression autocomplete because action names and internal column names affect the exact expression.

12. Add **SharePoint — Update item**.
13. Set ID to the trigger ID.
14. Preserve required fields such as Title using trigger dynamic content.
15. Set:
    - RequestNumber = Compose output.
    - AssignedTo = DefaultAssignee Email.
    - DueDate = calculated value.
    - ApprovalRequired = NeedsApproval.
    - Status = `Pending Approval` when NeedsApproval is true, otherwise `Assigned`.
16. Add **Send an email (V2)** to requester.
17. Add a second email/Teams notification to the assigned agent when appropriate.
18. Add **Create item** in `IT Request History` with EventType `Created/Assigned`.
19. Save and test with one request from each category type.

If no category configuration is found, do not silently fail. Update or log a safe error/status and notify the support owner.

### 9.2 Flow 2 — Conditional Approval

#### Purpose

Start approval once for requests whose category requires it.

#### Trigger design

1. Create an automated flow.
2. Trigger: **SharePoint — When an item is created or modified**.
3. Add trigger conditions or immediate conditions requiring:
   - ApprovalRequired = true.
   - ApprovalStarted = false.
   - Status = Pending Approval.

Why `ApprovalStarted` exists: the flow updates the SharePoint item, which triggers the flow again. Setting it true before starting approval prevents a duplicate approval loop.

#### Actions

1. **Update item:** set ApprovalStarted = Yes.
2. **Start and wait for an approval**.
3. Approval type: typically Approve/Reject — First to respond for one approver, or choose the correct business rule.
4. Assigned To: IT Manager or category approver from configuration.
5. Title: include RequestNumber and request Title.
6. Details: include requester, priority, description, and link to the item/app.
7. Add a **Condition** on the approval Outcome.

If approved:

- Update ApprovalOutcome = Approved.
- Update Status = Assigned.
- Create Approval history item.
- Notify requester and assigned agent.

If rejected:

- Update ApprovalOutcome = Rejected.
- Update Status = Rejected.
- Save approval comments in history.
- Notify requester.

Production considerations:

- Approver missing/inactive.
- Timeout and escalation.
- Retry/duplicate events.
- Approver authorization.
- Immutable decision history.
- Connection owner continuity.

### 9.3 Flow 3 — Status Change Notification

#### Purpose

Notify only when Status actually changes and record the transition.

#### Steps

1. Trigger: **When an item is created or modified**.
2. Add **Get changes for an item or a file (properties only)**.
3. ID = trigger ID.
4. Since = **Trigger Window Start Token**.
5. Add Condition: **Has Column Changed: Status** equals true.
6. If true:
   - Create a history item.
   - Send a status email to requester.
   - Notify assigned agent where relevant.
7. If new Status is Resolved:
   - Update ResolvedOn with `utcNow()`.
   - Include Resolution in the requester email.

Updating `ResolvedOn` triggers the flow again, but on the second run Status did not change, so the status-change branch should not repeat.

To store exact FromStatus, you may need version/history retrieval or explicitly write transitions from the app/flow. Do not claim `Get changes` automatically returns the previous value; it primarily tells whether a column changed.

### 9.4 Flow 4 — Scheduled Overdue Reminder

#### Purpose

Send a daily reminder for active requests whose DueDate has passed.

#### Steps

1. Select **Create > Scheduled cloud flow**.
2. Name: `ITSM - Overdue Reminder`.
3. Recurrence: once per day at an agreed time zone.
4. Add **Get items** for `IT Service Requests`.
5. Use a server-side Filter Query for active statuses and overdue DueDate when possible.
6. Add a Condition to ensure LastReminderDate is not today.
7. Send email/Teams reminder to AssignedTo and optionally IT Manager.
8. Update LastReminderDate.
9. Create a Reminder history item.

Avoid retrieving every list row daily and filtering only inside the flow. Use SharePoint-supported OData filters and indexed columns, then test with expected volume and pagination settings.

## 10. How assignment works in plain English

Example:

1. Employee submits Category = Network.
2. SharePoint creates item ID 125 with Status New.
3. New Request flow reads the Network row from `IT Categories`.
4. It finds default agent and SLA = 4 hours.
5. It creates RequestNumber `SR-000125`.
6. It sets AssignedTo to the network agent.
7. It sets DueDate to current time + 4 hours.
8. Because Network does not require approval, it sets Status Assigned.
9. It emails requester and agent.
10. It writes the Created/Assigned event to history.

For Category = Access, the same flow sets Pending Approval and the Approval flow starts once.

## 11. Security design — do not confuse it with filters

### Demo behavior

The requester gallery filters:

```powerfx
RequesterEmail = varUserEmail
```

This gives the correct user experience but does not prevent a user with broad SharePoint permission from opening the list directly.

### Real security options

For a SharePoint-based internal project:

- Configure list item-level read/edit behavior for creators when it matches the requirement.
- Give the IT support group an appropriate elevated permission level.
- Use Microsoft 365/Entra groups, not individual ad hoc sharing.
- Protect the SharePoint list/site directly.
- Review attachment and history-list permissions.
- Use a carefully designed item-permission flow only if necessary; unique permission scopes can become difficult at scale.

If security relationships become complex—manager hierarchy, teams, departments, column security, large row sharing—Dataverse is usually a stronger platform to evaluate.

### What to say in the interview

> The app filter was for user experience, while authorization was enforced at SharePoint through list/item permissions and IT support group access. I would never describe `Visible` or `Filter` as security. If the access model grew more complex, I would evaluate Dataverse roles, ownership, and teams.

## 12. Delegation and SharePoint performance

Likely high-volume columns to index:

- RequesterEmail
- Status
- AssignedTo or normalized AssignedToEmail
- DueDate
- Category
- Priority

Rules:

- Prefer direct delegable equality/prefix filters.
- Use `StartsWith` instead of an unsupported contains pattern when business search allows it.
- Do not `ClearCollect` the entire request list.
- Do not call `LookUp` separately for every gallery row.
- Delay search input if supported.
- Use Monitor with realistic data.
- Keep views/queries narrow.

SharePoint has thresholds and connector-specific delegation behavior. A formula working with 20 demo items does not prove correctness with 20,000.

## 13. Error handling and reliability

### App

- Disable submit while saving.
- Use Form `OnSuccess` and `OnFailure`.
- Use `IfError` around `Patch`.
- Show useful messages but not raw sensitive connector details.
- Preserve user input when a recoverable failure occurs.

### Flows

Use Scopes:

```text
Try
  Get configuration
  Update request
  Send notifications

Catch (run after failure/timeout)
  Log request ID and failed stage
  Notify support owner
  Set safe process/error status where appropriate

Finally
  Cleanup/telemetry
```

Make automation idempotent. The same trigger/retry should not create two approvals or duplicate business effects.

## 14. Testing plan

### Functional tests

| Test | Expected result |
|---|---|
| Submit Hardware request | Number, assignment, due date, emails, history. |
| Submit Access request | Pending Approval and one approval only. |
| Approve | Status Assigned; outcome/history/notifications. |
| Reject | Status Rejected; comment/history/notification. |
| Resolve without resolution | App blocks and explains error. |
| Resolve with text | Status and ResolvedOn updated; requester notified. |
| Search/filter | Correct records and no delegation warning for intended query. |
| Overdue scheduled run | One reminder per planned interval; LastReminderDate updated. |

### Security tests

- Requester A cannot read Requester B data through SharePoint direct link/API according to design.
- Normal requester cannot open Agent Queue data merely by changing navigation.
- Agent can update only allowed fields/items.
- Approval operation verifies intended approver/process.
- Flow connections use least privilege.

### Reliability tests

- Double-click submit.
- Flow retry.
- Missing category configuration.
- Missing/inactive assignee.
- Outlook/Approvals action failure.
- Concurrent agent update.
- Large attachment.
- List contains records beyond delegation limit.

### Device/accessibility tests

- Desktop and phone/tablet width if supported.
- Keyboard navigation.
- Screen-reader labels for icon buttons.
- Zoom and contrast.
- Errors shown with text, not color alone.

## 15. Documentation deliverables

For the CV statement “documented application and workflow logic,” explain that you prepared:

- Business requirements and roles.
- Architecture diagram.
- SharePoint data dictionary and internal column names.
- Screen/control inventory.
- Important Power Fx formulas.
- Flow trigger/action diagrams.
- Security model and group ownership.
- Test cases and evidence.
- Deployment/configuration instructions.
- Support owner and troubleshooting guide.
- Known limitations and improvement backlog.

## 16. Deployment and ownership

For a professional version:

- Develop in a dedicated development environment/site.
- Put canvas app and flows in a solution when supported.
- Use connection references for flows/connectors.
- Use environment variables for site/list IDs, support mailbox, and other environment configuration where practical.
- Provision SharePoint lists separately through documented/manual or automated site provisioning; ordinary Power Platform solutions do not automatically move the complete SharePoint schema/data.
- Test with non-owner accounts.
- Publish the canvas app and share with groups.
- Configure SharePoint permissions separately.
- Use organizational/service ownership for critical flows where supported.
- Monitor failures and handle owner departure.

## 17. What was technically difficult?

Use a genuine answer such as:

> The difficult part was coordinating app state with background flow updates. The SharePoint item is created before the flow assigns a number and agent, so the app cannot assume those fields are immediately populated. I saved the item first, navigated using `LastSubmit.ID`, showed the current status, and refreshed deliberately when the user needed the flow result. I also used an ApprovalStarted flag and trigger conditions to prevent the approval flow from starting twice.

Other defensible difficulties:

- SharePoint Choice and Person record types in Power Fx.
- Delegation with search and Person fields.
- Avoiding trigger loops.
- Separating UI filters from actual list permissions.
- Handling missing category configuration.
- Designing status transitions and history.

## 18. The 30-second interview answer

> I built an internal IT service desk as a canvas app backed by SharePoint. Employees submit categorized requests and track them, while IT agents use a filtered queue to assign, update, and resolve work. A main SharePoint list stores the request, with category configuration and history lists. Power Automate generates the service number, chooses the default assignee and SLA, handles conditional approvals, sends status notifications, and runs overdue reminders. I separated the UI from data permissions, tested delegation and flow loops, and documented the schema, formulas, workflows, security, and support process.

## 19. The two-minute interview answer

> I started by defining three personas: requester, IT agent, and manager, then defined the request lifecycle from New through Assigned, In Progress, Resolved, and Closed, with Pending Approval and Rejected paths. I used a SharePoint list called IT Service Requests for the transaction data, and smaller lists for category-to-agent/SLA configuration, support agents, and request history.
>
> In Power Apps I created a responsive canvas app. Requesters have Home, New Request, My Requests, and Detail screens. IT agents also get an Agent Queue and Update screen. I used an Edit Form for creation because it handles SharePoint validation and attachments, `SubmitForm` with `OnSuccess`/`OnFailure`, and galleries with delegable filters such as direct requester equality and `StartsWith` search. Targeted agent updates use `Patch` with validation—for example, Resolution is required before setting Resolved.
>
> Power Automate handles background logic. The new-item flow reads category configuration, generates `SR-` plus the SharePoint ID, assigns the agent, calculates the SLA due date, and sends notifications. A separate approval flow starts only when ApprovalRequired is true and ApprovalStarted is false, which prevents trigger loops and duplicate approvals. Another flow detects actual status-column changes and writes history, and a scheduled flow sends overdue reminders.
>
> For security, the app filters improve UX but SharePoint permissions and groups protect the list. I tested with requester and agent accounts, reviewed delegation and flow failures, and documented the architecture, data dictionary, formulas, flows, security assumptions, deployment, and known limits. If the security and relationship requirements grew, I would consider moving the data model to Dataverse.

## 20. The five-minute whiteboard explanation

Draw four boxes:

1. Users: Requester, Agent, Manager.
2. Canvas app screens.
3. SharePoint lists and relationships/configuration.
4. Four Power Automate flows.

Then narrate one Hardware request end to end:

1. User opens New Request.
2. `NewForm` prepares empty defaults.
3. `SubmitForm` validates and creates the SharePoint item.
4. `OnSuccess` captures `LastSubmit.ID` and navigates to detail.
5. SharePoint-created flow receives item ID.
6. Flow reads Hardware category configuration.
7. Flow creates RequestNumber and due date, assigns agent, updates status.
8. Flow sends emails and writes history.
9. Agent sees it through Agent Queue filters.
10. Agent moves to In Progress, then Resolved with a resolution.
11. Status flow detects the change, records it, and notifies requester.
12. Scheduled flow reminds the agent if DueDate passes before resolution.

Finish with three risks and mitigations:

- Delegation/list growth: delegable filters, indexes, realistic testing.
- Security: list permissions/groups, not hidden controls.
- Duplicate automation: trigger conditions, ApprovalStarted/idempotency.

## 21. Follow-up questions and strong answers

### Why did you use SharePoint instead of Dataverse?

> It was an internal Microsoft 365 scenario with relatively simple request records, existing SharePoint access, and document/attachment needs. I still designed for SharePoint's delegation and permission limitations. If we needed richer relational data, manager/team row security, complex audit, or model-driven operations at scale, I would evaluate Dataverse.

### Why use a category configuration list?

> It removes assignee, SLA, and approval decisions from hard-coded formulas. An authorized administrator can change the routing configuration without republishing the app.

### Why separate the flows?

> Each flow has one operational responsibility and can be tested, monitored, retried, and changed independently. The boundaries also make trigger conditions and loop prevention clearer.

### How did you avoid duplicate approvals?

> The trigger requires ApprovalRequired true, Status Pending Approval, and ApprovalStarted false. The flow sets ApprovalStarted true before creating the approval. For a stronger production design I would also use correlation/history and idempotency checks.

### How did you track status history?

> A status-change flow uses SharePoint's Get changes action to detect whether Status changed, then writes a separate history item and notifies the requester. Exact previous-value storage needs explicit version/history or transition capture; I would not claim Get changes returns every old value automatically.

### How did you secure the app?

> App sharing and UI visibility were only the first layer. SharePoint list/item permissions and IT group membership protected the records. The flow used least-privilege connections. I tested with real requester and agent accounts rather than only the owner.

### How did you handle errors?

> The app uses form `OnFailure`, `IfError` for Patch, and saving flags. Flows use Try/Catch/Finally-style scopes, safe logging with request ID/stage, and support notifications. Missing category configuration follows an explicit failure path.

### What is delegation here?

> Power Apps must translate gallery filters to SharePoint so the server searches the complete list. If part of the formula is nondelegable, only the local row-limit subset may be evaluated. I used supported direct comparisons/prefix search, indexed key columns, warnings/Monitor, and realistic-volume tests.

### What would you improve next?

> I would add stronger SLA/business-hours calculation, feedback/knowledge articles, Power BI metrics, automated solution/list provisioning, more complete idempotency/telemetry, and evaluate Dataverse if the security/relationship/scale requirements increased.

### Did the app call every flow directly?

> No. Core lifecycle flows were SharePoint-triggered so the process runs regardless of which allowed client changes the item. A direct Power Apps trigger is useful when the user needs an immediate action/response, but it should not be the only enforcement path for mandatory workflow.

### What happens immediately after save?

> The SharePoint row exists, but background fields such as RequestNumber and AssignedTo may not be populated yet because the flow is asynchronous. The detail screen uses the saved ID and can show Processing/New until a deliberate refresh retrieves the flow update.

## 22. Be honest without losing the interview

If you have only studied/built the demo, say:

> I built the complete demo and can explain the data model, app formulas, and flow logic. I have not yet operated it at enterprise production scale, so for production I would validate permissions, licensing, list volume, gateway/connectors, monitoring, and support ownership with the platform administrator.

Do not invent user counts, production incidents, measured performance numbers, or security approvals. Interviewers often ask follow-ups that reveal fabricated experience. A junior who deeply understands a real demo is stronger than someone who claims unsupported production history.

## 23. Build-this-tonight minimum demo

If the interview is tomorrow, build this subset in order:

1. Create `IT Service Requests` with Title, Description, Category, Priority, Status, RequesterEmail, AssignedTo, DueDate, Resolution.
2. Create `IT Categories` with one or two rows.
3. Build New Request form and My Requests gallery.
4. Build Agent Queue and one status update action.
5. Build New Request Assignment flow.
6. Build either Approval flow or Status Notification flow.
7. Test with at least five items.
8. Draw the full four-flow architecture even if the scheduled flow is not implemented yet.
9. Practice the two-minute answer.

After the interview, build all four flows and security tests before representing the project as production-complete.

## 24. Final recall checklist

Without opening this file, answer:

- What are the four SharePoint lists and why does each exist?
- What happens between `SubmitForm` and `OnSuccess`?
- Why does the approval flow not loop?
- How is request number generated?
- How are agent and due date selected?
- How is status change detected?
- Why is the gallery filter not security?
- What is the delegation risk?
- How are errors handled in app and flow?
- How would you deploy and support it?
- When would you replace SharePoint with Dataverse?

If you can answer all eleven in your own words, you understand the project rather than merely memorizing the CV bullets.

## Primary references

- [Connect Power Apps to SharePoint](https://learn.microsoft.com/power-apps/maker/canvas-apps/connections/connection-sharepoint-online)
- [Edit and display forms](https://learn.microsoft.com/power-apps/maker/canvas-apps/controls/control-form-detail)
- [SharePoint delegation support](https://learn.microsoft.com/power-apps/maker/canvas-apps/connections/connection-sharepoint-online#power-apps-delegable-functions-and-operations-for-sharepoint)
- [Trigger flows from canvas apps](https://learn.microsoft.com/power-apps/maker/canvas-apps/how-to/trigger-flow)
- [SharePoint connector](https://learn.microsoft.com/connectors/sharepointonline/)
- [Power Automate approvals](https://learn.microsoft.com/power-automate/get-started-approvals)
- [Error handling in cloud flows](https://learn.microsoft.com/power-automate/guidance/coding-guidelines/error-handling)

