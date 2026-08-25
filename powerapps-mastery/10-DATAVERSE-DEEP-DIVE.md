# 10 — Microsoft Dataverse Deep Dive

## Goal

Design a relational, secure, maintainable business data model and build it inside a solution.

## What Dataverse provides

Dataverse is more than cloud table storage. It provides:

- Typed tables, columns, relationships, and metadata.
- Role-based, row-level, team, hierarchy, and column security capabilities.
- Auditing and change tracking capabilities.
- Business rules, calculated/rollup behavior, APIs, events, and automation integration.
- Native support for model-driven apps.
- Solution-aware application lifecycle management.

## Start from business facts, not screens

Bad design starts with “I need 40 fields on this form.” Good design starts with business entities and relationships.

Employee Request Hub facts:

- A department has many requests.
- A request belongs to one department.
- A request has many comments.
- A request can have many approval events.
- A user creates/owns or is associated with requests.

## Proposed schema

### Request table

| Display name | Suggested type | Notes |
|---|---|---|
| Request Title | Primary name text | Human-readable summary/title; formulas in this course shorten the example field to `Title`. |
| Request Number | Autonumber | Example `REQ-{SEQNUM:6}`. |
| Request Type | Choice | Equipment, Training, Travel, Other. |
| Request Status | Choice | Draft, Submitted, In Review, Approved, Rejected, Fulfilled, Cancelled. |
| Amount | Currency | Use the organization's currency strategy. |
| Description | Multiline text | Business justification. |
| Needed By | Date only | Avoid time-zone ambiguity when time is irrelevant. |
| Department | Lookup to Department | N:1 relationship. |
| Requested By Email | Text | Useful for display/integration; not the only identity/security mechanism. |
| Manager | Lookup to User | Approver reference where appropriate. |
| Submitted On | Date and time | Separate from system-created time. |
| Decision Comment | Multiline text | Consider a separate Approval table for full history. |

### Department table

| Column | Type |
|---|---|
| Department Name | Primary name text |
| Department Code | Text, alternate-key candidate |
| Cost Center | Text |
| Active | Yes/No |

### Request Comment table

| Column | Type |
|---|---|
| Comment | Primary name or multiline text pattern |
| Request | Lookup to Request |
| Comment Type | Choice |
| Visible to Requester | Yes/No |

### Approval Event table

Use when audit/history needs more than current-state columns:

| Column | Type |
|---|---|
| Approval Event | Primary name/autonumber |
| Request | Lookup |
| Approver | Lookup to User |
| Outcome | Choice |
| Comment | Multiline text |
| Decision Time | Date and time |

## Create the solution first

Typical path:

1. Go to `https://make.powerapps.com`.
2. Confirm the development environment.
3. Select **Solutions > New solution**.
4. Display name: `Employee Request Hub`.
5. Create or select a publisher owned by your organization.
6. Use a meaningful prefix such as `erh`, not the default publisher for real work.
7. Version: `1.0.0.0`.
8. Select **Create**.

Publisher prefix affects schema names and is hard to change later. Choose it deliberately.

## Create a table

Inside the solution:

1. Select **New > Table > Table (advanced properties)** or the current table creation command.
2. Display name: `Request`.
3. Plural name: `Requests`.
4. Primary column display name: `Request Title` (this remains distinct from the generated Request GUID column).
5. Choose ownership type.
6. Enable attachments, auditing, activities, or other features only when required.
7. Select **Save**.

### Ownership choice

| Ownership | Use |
|---|---|
| User or team owned | Record access varies by owner/team; common for operational records. |
| Organization owned | Records have organization-wide ownership; privileges do not use owner depth in the same way. |

Ownership type is a foundational decision and generally cannot be casually changed later. Request is often user/team owned; reference data such as Department is often organization owned.

## Add columns

Open the table and select **Columns > New column**.

For each column:

1. Set display name.
2. Confirm schema/logical name and publisher prefix.
3. Choose data type.
4. Set required level deliberately.
5. Configure length, format, minimum/maximum, precision, or choice values.
6. Save.

Required levels:

- **Optional:** may be blank.
- **Business recommended:** UI guidance, not hard enforcement.
- **Business required:** platform validation for supported operations.

Do not mark every field required. Ask whether the value is known at creation and required for every process state.

## Choices: local or global

- A **local choice** belongs to one column.
- A **global choice** can be reused by multiple columns/tables.

Use a global choice when values and meaning truly need to remain consistent across the solution. Do not reuse merely because labels happen to match today.

Choice numeric values are durable internal values. Avoid deleting/recreating options casually in deployed systems.

## Relationships

### One-to-many / many-to-one

Creating a lookup from Request to Department produces:

```text
Department (1) --------< Request (many)
```

Typical UI path:

1. Open the `Request` table.
2. Select **Relationships** or add a new Lookup column.
3. Choose many-to-one.
4. Related table: `Department`.
5. Lookup display name: `Department`.
6. Review relationship behavior.
7. Save.

### Many-to-many

Use only when both sides genuinely have many related records and the relationship needs no additional attributes. If the relationship itself has facts—role, quantity, effective date—create an explicit intersect/junction table.

## Relationship behavior and cascading

Cascading can affect assign, share, unshare, delete, reparent, and merge behaviors. A broad cascade delete can remove large record trees.

Before changing cascade behavior:

1. Map the parent/child lifecycle.
2. Decide whether child history must survive.
3. Test delete, assign, and sharing behavior in a nonproduction environment.
4. Document the business reason.

For Request Comments, deleting the parent might cascade in a disposable training system; in a regulated system, cancellation and retention are usually safer than deletion.

## Calculated, formula, and rollup concepts

- **Formula/calculated column:** derives a value for a row from supported fields/functions.
- **Rollup column:** periodically aggregates related records.
- **Power Fx in app:** calculates for the current experience only.
- **Flow/plugin:** handles procedural or cross-system logic.

Put a rule at the lowest layer that must consistently enforce or expose it. A total required by every app/report should not exist only in one label formula.

## Alternate keys and autonumber

- Primary Dataverse ID is a GUID.
- Autonumber provides a user-friendly business reference.
- Alternate keys enforce/integrate through a unique business identifier where supported.

Do not replace the GUID with `Max()+1` logic in the app.

## Business rules

Business rules can set values, required levels, visibility, recommendations, or validation depending on scope and supported feature. Typical path:

1. Open table in the solution.
2. Select **Business rules > New business rule**.
3. Add condition(s) and action(s).
4. Validate.
5. Save and activate.

Example: when Request Type is `Travel`, make `Needed By` required. Confirm where the rule runs and whether it applies to every intended client/API path. Use server-side enforcement for rules that must never be bypassed.

## Auditing

Auditing must be enabled at appropriate environment/table/column levels and is subject to retention and capacity considerations.

Audit answers “who changed what and when”; it is not the same as business approval history. Use an Approval Event table when the business process needs structured decision records.

## Data model review checklist

- Are repeating groups separated into related tables?
- Are relationships and ownership types correct?
- Are names business-readable and schema names stable?
- Are choice values controlled rather than free text where appropriate?
- Are required fields known at the correct lifecycle stage?
- Are dates `Date only` versus `User local`/time-zone aware intentionally?
- Are money and currency requirements understood?
- Are cascade rules safe?
- Are alternate keys/integration identifiers defined?
- Is security modeled before data import?
- Is auditing enabled only where justified?

## Common mistakes

- One giant table containing repeated comment or approval columns.
- Storing department names as free text on every request.
- Using email text as the only security identity.
- Choosing organization ownership for records that require owner-level access.
- Enabling cascade delete without testing impact.
- Putting all validation only in a canvas app.
- Developing outside a solution.

## Interview questions

**What is the difference between user/team-owned and organization-owned tables?**  
User/team-owned rows have owners and support access depths tied to ownership/business-unit structures. Organization-owned rows are owned at organization scope and fit shared reference-like data.

**When do you create a junction table instead of N:N?**  
When the relationship needs its own attributes, lifecycle, security, or process.

**Choice versus lookup?**  
A Choice is a controlled set of values. A Lookup references a row in another table and supports related data, security, and lifecycle.

## Challenge

The business asks for `Approver1Name`, `Approver1Decision`, `Approver2Name`, and `Approver2Decision` columns. Redesign it.

### Answer

Create an Approval Event table related N:1 to Request, with approver, stage/order, outcome, comment, and decision time. This supports any number of approval steps and preserves history.

## Primary references

- [Create a custom table](https://learn.microsoft.com/power-apps/maker/data-platform/create-custom-entity)
- [Create table relationships](https://learn.microsoft.com/power-apps/maker/data-platform/create-edit-entity-relationships)
- [Why choose Dataverse?](https://learn.microsoft.com/power-apps/maker/data-platform/why-dataverse-overview)
- [Dataverse security](https://learn.microsoft.com/power-platform/admin/wp-security)
