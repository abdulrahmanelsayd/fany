# IT Service Management Project — Smart Interview Walkthrough

> Power Apps + Power Automate + SharePoint

## 1. Understand the project in one minute

The project replaces IT requests sent through emails or messages with one central application.

- Employees submit and track IT requests in a **Power Apps canvas app**.
- **SharePoint Lists** permanently store requests and configuration.
- IT agents use a queue to assign, update, and resolve requests.
- **Power Automate** assigns agents, sends notifications, starts approvals, and reminds agents about overdue requests.

```text
Employee -> Power Apps -> SharePoint Lists <- IT Agent
                           |
                           v
                    Power Automate
              assignment / approval / email
```

Do not say “Power Apps stores the data.” Power Apps is the interface; SharePoint is the data source.

## 2. The four important terms

| Term | Meaning here |
|---|---|
| Gallery | Displays multiple requests. |
| Form | Creates or edits one request. |
| Power Fx | Formulas controlling the app. |
| Flow | Background automation triggered by a data change or schedule. |

## 3. SharePoint data design

### Main list: `IT Service Requests`

Create it through **SharePoint > Site contents > New > List > Blank list**.

| Column | Type | Why |
|---|---|---|
| Title | Text | Short issue summary. |
| RequestNumber | Text | Generated value such as `SR-000125`. |
| Description | Multiple lines | Full problem. |
| Category | Choice | Hardware, Software, Network, Access, Purchase, Other. |
| Priority | Choice | Low, Medium, High, Critical. |
| Status | Choice | New, Pending Approval, Assigned, In Progress, Resolved, Closed, Rejected. |
| RequesterEmail | Text | Current user's normalized email. |
| AssignedTo | Person | IT agent. |
| DueDate | Date/time | SLA deadline. |
| Resolution | Multiple lines | Agent's solution. |
| ApprovalRequired | Yes/No | Whether approval is needed. |
| ApprovalStarted | Yes/No | Prevents duplicate approval runs. |
| ResolvedOn | Date/time | Resolution timestamp. |

SharePoint automatically supplies `ID`, `Created`, `Modified`, and `Created By`.

### Configuration list: `IT Categories`

| Column | Purpose |
|---|---|
| Title | Category name. |
| DefaultAssignee | Agent for the category. |
| SLAHours | Allowed completion time. |
| NeedsApproval | Whether the category requires approval. |

Example: Network → network agent → 4 hours → no approval.

### History list: `IT Request History`

Stores Request ID, event type, old/new status, comment, and performer. It provides a readable workflow history instead of keeping only the latest state.

An optional `IT Support Agents` list contains agent email, role, and active status for app navigation. It does not replace real SharePoint permissions.

## 4. Build the canvas app

### Create and connect

1. Open `make.powerapps.com`.
2. Select **Create > Blank canvas app**.
3. Name it `IT Service Desk`.
4. Open **Data > Add data > SharePoint**.
5. Select the site and connect the three lists.

### Screens

| Screen | Purpose |
|---|---|
| `scrHome` | Navigation. |
| `scrMyRequests` | Employee's request gallery. |
| `scrNewRequest` | New-request form. |
| `scrRequestDetail` | One request and its history. |
| `scrAgentQueue` | Requests handled by IT. |
| `scrAgentUpdate` | Assignment, status, and resolution. |

Use auto-layout containers so screens resize cleanly. A screen contains controls; each control has properties containing Power Fx formulas.

## 5. The six formulas you need to understand

Modern text inputs commonly use `.Value`; classic inputs commonly use `.Text`.

### A. Store the current user

`App.OnStart`:

```powerfx
Set(varUserEmail, Lower(User().Email))
```

### B. Show only the employee's requests

`galMyRequests.Items`:

```powerfx
SortByColumns(
    Filter(
        'IT Service Requests',
        RequesterEmail = varUserEmail &&
        (IsBlank(txtSearch.Value) || StartsWith(Title, txtSearch.Value))
    ),
    "Created",
    SortOrder.Descending
)
```

`Filter` returns matching records. `StartsWith` provides prefix search. `SortByColumns` shows newest requests first.

### C. Open a request

Gallery item `OnSelect`:

```powerfx
Set(varSelectedRequestId, ThisItem.ID);
Navigate(scrRequestDetail)
```

`ThisItem` is the current gallery record.

### D. Start a new request

New button `OnSelect`:

```powerfx
NewForm(frmRequest);
Navigate(scrNewRequest)
```

### E. Submit the form

Submit button `OnSelect`:

```powerfx
If(
    frmRequest.Valid,
    SubmitForm(frmRequest),
    Notify("Complete the required fields.", NotificationType.Error)
)
```

Form `OnSuccess`:

```powerfx
Set(varSelectedRequestId, frmRequest.LastSubmit.ID);
Notify("Request submitted.", NotificationType.Success);
Navigate(scrRequestDetail)
```

Navigate in `OnSuccess`, because it confirms SharePoint saved the record.

### F. Resolve a request

Agent button `OnSelect`:

```powerfx
If(
    IsBlank(Trim(txtResolution.Value)),
    Notify("Enter the resolution first.", NotificationType.Error),
    IfError(
        Patch(
            'IT Service Requests',
            LookUp('IT Service Requests', ID = varSelectedRequestId),
            {
                Status: {Value: "Resolved"},
                Resolution: Trim(txtResolution.Value)
            }
        ),
        Notify("Update failed.", NotificationType.Error),
        Notify("Request resolved.", NotificationType.Success)
    )
)
```

`Patch` updates a selected SharePoint item. `IfError` handles connector failure.

## 6. The four Power Automate flows

### Flow 1 — New request assignment

Trigger: **SharePoint — When an item is created**.

Steps:

1. Read the request Category.
2. Get the matching row from `IT Categories`.
3. Generate request number:

```text
concat('SR-', formatNumber(triggerOutputs()?['body/ID'], '000000'))
```

4. Set AssignedTo from DefaultAssignee.
5. Calculate DueDate using SLAHours.
6. Set Status to Assigned or Pending Approval.
7. Update the SharePoint item.
8. Email requester and agent.
9. Create a history item.

### Flow 2 — Approval

Trigger: request modified with:

- ApprovalRequired = Yes.
- ApprovalStarted = No.
- Status = Pending Approval.

Steps:

1. Set ApprovalStarted = Yes.
2. Use **Start and wait for an approval**.
3. If approved, set Status = Assigned.
4. If rejected, set Status = Rejected.
5. Write history and notify requester.

`ApprovalStarted` prevents the flow's own update from starting another approval.

### Flow 3 — Status notification

Trigger: **When an item is created or modified**.

1. Use **Get changes for an item or a file**.
2. Check whether Status changed.
3. Write the transition to history.
4. Email requester.
5. If Resolved, set ResolvedOn and include Resolution.

### Flow 4 — Overdue reminder

Trigger: **Recurrence**, once per day.

1. Get active requests where DueDate has passed.
2. Email AssignedTo and IT manager.
3. Record the reminder date/history.

Use a SharePoint filter query and indexed columns rather than downloading every list item.

## 7. One request from beginning to end

1. Employee enters a Network problem and selects Submit.
2. `SubmitForm` creates SharePoint item ID 125.
3. `OnSuccess` opens the saved request.
4. Flow 1 finds Network configuration.
5. It generates `SR-000125`, assigns the network agent, and calculates a four-hour due date.
6. It updates Status to Assigned and sends notifications.
7. Agent sees the request in `scrAgentQueue`.
8. Agent changes it to In Progress and later Resolved with a resolution.
9. Flow 3 detects the status changes, writes history, and emails the employee.
10. If the due date passes first, Flow 4 sends a reminder.

For an Access request, Flow 1 sets Pending Approval and Flow 2 controls the next step.

## 8. Three important professional points

### UI filters are not security

```powerfx
RequesterEmail = varUserEmail
```

improves the app experience but does not secure the SharePoint list. Real authorization comes from SharePoint permissions and IT security groups. If row security becomes complex, Dataverse is a better platform to evaluate.

### Small-data success does not prove delegation

SharePoint should evaluate large-list filters on the server. Use supported direct comparisons, `StartsWith`, indexed columns, App Checker, Monitor, and realistic data. Loading the entire list into a collection is not a proper fix.

### Flows are asynchronous

Immediately after `SubmitForm`, the SharePoint item exists but RequestNumber or AssignedTo may still be blank until Flow 1 completes. The detail screen should temporarily show New/Processing and refresh deliberately.

## 9. How it was tested

- Required fields cannot be blank.
- Hardware request assigns the correct agent.
- Access request creates exactly one approval.
- Approval and rejection update the correct status.
- Resolution is required before Resolved.
- Requesters cannot access other employees' records through SharePoint permissions.
- Agent and requester notifications arrive.
- Overdue requests receive one planned reminder.
- Flow failure and missing category configuration are logged/handled.
- Gallery remains correct beyond the Power Apps local row limit.

## 10. Your 30-second answer

> I built an internal IT service desk using a Power Apps canvas app backed by SharePoint. Employees submit and track requests, while IT agents use a filtered queue to assign, update, and resolve them. A category configuration list controls the default agent, SLA, and approval requirement. Power Automate generates the request number, handles assignment and conditional approval, sends status notifications, and runs overdue reminders. I also separated UI filtering from SharePoint permissions and tested delegation, errors, and duplicate flow triggers.

## 11. Your two-minute answer

> I started with three roles: requester, IT agent, and manager, then defined the request lifecycle from New to Assigned, In Progress, Resolved, and Closed, with a Pending Approval path.
>
> SharePoint stores the data. The main list contains request details, category, priority, status, assignee, SLA due date, and resolution. A category list stores routing configuration, and a history list stores important workflow events.
>
> In the canvas app, requesters use a form to submit and galleries to track their requests. Agents have a queue and an update screen. I used `SubmitForm` with `OnSuccess` and `OnFailure`, delegable gallery filters, and `Patch` for targeted status/resolution updates.
>
> Four flows handle the background work: new-request assignment, conditional approval, status-change notification, and scheduled overdue reminders. The approval flow sets an ApprovalStarted flag before creating the approval to prevent loops and duplicates.
>
> SharePoint permissions protect the actual records; app filters only improve the experience. I tested with requester and agent accounts, checked delegation with realistic volume, and documented the data model, formulas, flows, security assumptions, and known SharePoint limits.

## 12. Likely follow-up questions

### Why SharePoint instead of Dataverse?

> It was a relatively simple internal Microsoft 365 request process with existing SharePoint access. If it required complex relationships, team/manager row security, column security, or larger enterprise scale, I would evaluate Dataverse.

### Why use a configuration list?

> Agent, SLA, and approval routing can change without editing and republishing the app.

### How did you prevent duplicate approvals?

> The flow starts only when approval is required, status is Pending Approval, and ApprovalStarted is false. It sets ApprovalStarted true before creating the approval.

### What was the difficult part?

> Coordinating the saved SharePoint item with asynchronous flows, avoiding approval loops, and keeping search/filter formulas delegable while separating UI filtering from actual permissions.

### `Patch` or `SubmitForm`?

> I used `SubmitForm` for request creation because forms handle validation and attachments. I used `Patch` for focused agent updates where I controlled validation and errors.

### What would you improve?

> Stronger business-hours SLA calculation, Power BI reporting, better telemetry/idempotency, automated provisioning, and Dataverse migration if security or scale became more complex.

## 13. What you must draw on paper

```text
Requester screens        SharePoint             Agent screens
New / My / Detail  <-->  Requests list   <-->  Queue / Update
                           |       |
                    Categories   History
                           |
                    Power Automate
              Assign / Approve / Notify / Remind
```

Explain one Network request from submission to resolution. If you can do that without reading, you understand the project.

## 14. Honest final sentence

If you built only the training demo, say:

> I built and tested the complete demo and understand its data, formulas, and flow architecture. Before calling it enterprise production-ready, I would validate real list volume, permissions, licensing, monitoring, ownership, and support requirements with the platform administrator.

## References

- [Power Apps and SharePoint](https://learn.microsoft.com/power-apps/maker/canvas-apps/connections/connection-sharepoint-online)
- [Form controls](https://learn.microsoft.com/power-apps/maker/canvas-apps/controls/control-form-detail)
- [SharePoint delegation](https://learn.microsoft.com/power-apps/maker/canvas-apps/connections/connection-sharepoint-online#power-apps-delegable-functions-and-operations-for-sharepoint)
- [Power Automate approvals](https://learn.microsoft.com/power-automate/get-started-approvals)

