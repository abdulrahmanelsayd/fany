# 11 — Model-Driven Apps

## Goal

Build a responsive operations app from the Dataverse model using views, forms, navigation, charts, and role-based access.

## Mental model

```text
Dataverse metadata
  ├── Tables and relationships
  ├── Views: which records/columns appear in lists
  ├── Forms: how one record is displayed/edited
  ├── Charts/dashboards: summaries
  └── Business process: guided stages
              |
              v
        Model-driven app
```

Model-driven design gives less pixel-level control than canvas design but rapidly produces a consistent, accessible, responsive data experience.

## Prerequisites

- An environment with Dataverse.
- Appropriate maker/customizer permissions.
- The `Employee Request Hub` solution and tables from Lesson 10.
- At least several sample Request and Department records.

## Step 1: create the app inside the solution

1. Open `https://make.powerapps.com`.
2. Confirm the development environment.
3. Select **Solutions**.
4. Open `Employee Request Hub`.
5. Select **New > App > Model-driven app**.
6. Name it `Employee Request Operations`.
7. Add an optional description.
8. Select **Create**.

The modern app designer opens.

## Step 2: add pages/tables

In the app designer:

1. Select **Add page**.
2. Choose a Dataverse table page.
3. Add `Request`.
4. Add `Department`.
5. Add `Approval Event` and `Request Comment` if operations users need direct navigation.
6. Reorder pages and create navigation groups such as **Work** and **Configuration**.

Use navigation to express user tasks, not your database's entire schema. Hide technical/reference tables that users should reach only through relationships.

## Step 3: create useful views

A view defines columns, ordering, sorting, and filters for a list of records.

Create these public views for Request:

- My Active Requests
- Submitted Requests
- Requests Awaiting Decision
- Approved Requests
- Overdue Requests
- All Active Requests

Typical path from the solution:

1. Open the Request table.
2. Select **Views > New view**.
3. Enter the name.
4. Add only useful columns: Request Number, Request, Status, Requested By, Department, Amount, Needed By.
5. Configure filters and default sort.
6. Save and publish.

View principles:

- Place identifying/actionable columns first.
- Do not show every column.
- Filter to a meaningful work queue.
- Use stable server-side data, not UI-only assumptions.
- Configure Quick Find/search columns deliberately.

Return to app designer, select the table page, and choose which views are **In this app**. Remove irrelevant views rather than exposing a confusing selector.

## Step 4: design the main form

1. Open Request table **Forms**.
2. Create or edit a main form.
3. Use sections/tabs such as:
   - Summary
   - Request details
   - Decision
   - Related information
   - Audit/administration
4. Add a subgrid for Request Comments.
5. Add a subgrid for Approval Events.
6. Add a quick view for Department information if useful.
7. Set labels, required fields, visibility, and controls.
8. Save and publish.

Common form types:

| Form | Purpose |
|---|---|
| Main | Full record experience. |
| Quick create | Fast minimal creation. |
| Quick view | Read-only related-record information embedded on a form. |
| Card | Compact record display in supported experiences. |

Form visibility can be associated with security roles, but table/column privileges remain the actual data security boundary.

## Step 5: business process flow concept

A business process flow (BPF) guides users through stages such as:

```text
Draft -> Manager Review -> Operations -> Fulfilled
```

Use it when users need a consistent stage-based process visible on the record. Do not use a BPF merely as decoration; define stage entry/exit rules, ownership, automation, and exceptional paths.

Typical solution path:

1. **New > Automation > Process > Business process flow** or current equivalent.
2. Select the primary table.
3. Add stages and data steps.
4. Validate, save, and activate.
5. Add the process to the app if selection is required.

Feature labels and creation surfaces can change; use the solution's automation/process area.

## Step 6: charts and dashboards

Create a chart such as Requests by Status:

1. Open the Request table's chart area or app designer chart configuration.
2. Create a system chart.
3. Category: Request Status.
4. Series: Count of Requests.
5. Save and publish.
6. Include the chart in the app.

Charts respect the view/query and user data privileges. For enterprise analytics, use Power BI when modeling, scale, and reporting needs go beyond operational charts.

## Step 7: save, validate, publish, play

1. Select **Save**.
2. Resolve app designer validation messages.
3. Select **Publish**.
4. Select **Play**.
5. Test navigation, each included view, create/edit, related subgrids, search, charts, and small screen behavior.

## Step 8: share through roles

Typical path:

1. Return to **Apps**.
2. Open the app context menu.
3. Select **Share**.
4. Associate appropriate security roles/users/teams.
5. Copy the web link.

Users also need correct Dataverse privileges and licenses. App availability does not override table security.

## Canvas page inside a model-driven app

Use a custom page when a focused area needs canvas-style layout while the surrounding application benefits from model-driven navigation/data. Good examples include a visual scheduling board or guided intake page.

Do not rebuild every standard form as a custom page. Use the native model-driven experience unless custom interaction adds real value.

## Common mistakes

- Building forms before designing the data model.
- Adding every table/view to navigation.
- Creating one form containing every field with no information hierarchy.
- Assuming form hiding equals column security.
- Forgetting to publish table components and then publishing only the app.
- Giving users System Administrator because a custom role is incomplete.
- Using a BPF for a process that has no stable stages or owners.

## Interview questions

**Why are model-driven apps responsive?**  
The UI is generated from Dataverse metadata and standardized application components rather than fixed-position controls.

**What is a view?**  
A definition of the columns, order, width, default sort, and filters used to display a list of table records.

**What is the difference between a business process flow and a cloud flow?**  
A BPF guides users through visible record stages. A cloud flow automates actions and integrations when triggered. They can work together.

**Can you add custom UI to a model-driven app?**  
Yes, through custom pages and other extensibility options such as PCF controls, when justified.

## Challenge

Operations users need all submitted requests; employees should see only their own. Should you create separate apps?

### Answer

Not automatically. First design Dataverse roles/ownership and views. A single app can show different records and capabilities based on security. Separate apps may still improve task-focused navigation or reduce complexity, but do not use separate UI as the security boundary.

## Primary references

- [Create a model-driven app](https://learn.microsoft.com/power-apps/maker/model-driven-apps/create-edit-app)
- [Understand model-driven views](https://learn.microsoft.com/power-apps/maker/model-driven-apps/create-edit-views)
- [Create and edit model-driven forms](https://learn.microsoft.com/power-apps/maker/model-driven-apps/create-design-forms)
- [Share a model-driven app](https://learn.microsoft.com/power-apps/maker/model-driven-apps/share-model-driven-app)

