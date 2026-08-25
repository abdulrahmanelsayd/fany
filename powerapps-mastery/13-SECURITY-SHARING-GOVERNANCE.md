# 13 — Security, Sharing, and Governance

## Goal

Explain and implement security from identity through data access, then apply environment and connector governance without confusing UI personalization with authorization.

## Security layers

```text
Identity (Microsoft Entra ID)
  -> Environment access
    -> App access
      -> Connector/flow permissions
        -> Dataverse table privilege
          -> Access depth + ownership/team/sharing
            -> Column security (when configured)
```

All relevant layers must align.

## Authentication versus authorization

- **Authentication:** Who are you?
- **Authorization:** What may you do to which records/columns?

`User().Email` helps personalize UI. It does not grant or restrict Dataverse records by itself.

## Canvas app sharing

Typical path:

1. Maker portal **Apps**.
2. Select the canvas app.
3. Select **Share**.
4. Add individual users or preferably appropriate Microsoft Entra security groups.
5. Choose user versus co-owner only when needed.
6. Review warnings about data sources, connections, flows, and custom connectors.

Users also need:

- Underlying SharePoint/Dataverse/SQL permissions.
- Required connection setup.
- Flow run-only permission/configuration.
- Correct license.

Do not give co-owner merely to make an app run. Co-owner is a maker/management capability.

## Model-driven app sharing

Model-driven apps rely heavily on Dataverse roles:

1. Define custom roles.
2. Give each role required table privileges and access depths.
3. Associate appropriate roles/users/teams with the app.
4. Assign roles to users or teams.
5. Test with nonadministrator accounts.

App visibility plus no table privileges produces an unusable app. Table privileges plus no app availability may expose data through other allowed clients but not that app.

## Dataverse privileges

Core table privileges:

- Create
- Read
- Write
- Delete
- Append
- Append To
- Assign
- Share

`Append` and `Append To` are frequently misunderstood:

- **Append:** this row can be attached to another row.
- **Append To:** other rows can be attached to this row.

Creating/associating a lookup can require both sides.

## Access depths

For user/team-owned tables, common depths are:

| Depth | Meaning |
|---|---|
| None | No privilege. |
| User/Basic | Rows owned by user, assigned team, or shared appropriately. |
| Business Unit/Local | Rows in the user's business unit. |
| Parent: Child Business Units/Deep | Business unit and children. |
| Organization/Global | All rows in environment for that table. |

The exact labels/icons in the modern role editor can vary. Understand the scope, not only the icon color.

## Ownership, teams, and sharing

Use:

- **Owner teams** when a team owns rows and receives roles.
- **Access teams** for flexible record collaboration without transferring ownership, in configured scenarios.
- **Microsoft Entra group teams** to manage membership from directory groups.
- **Row sharing** for exceptions, not as the only strategy for a massive predictable access model.

Employee Request Hub example:

- Requester role: create Request, read/write own Draft rows, read own submitted rows, append comments as allowed.
- Manager role/team: read requests within intended scope, update decision fields/process records.
- Operations role: organization read and controlled update on operational fields.
- Configuration administrator: manage Department/reference data.

Exact implementation depends on organization/business units and manager relationships; test the model before promising it.

## Field/column security

Column security profiles can restrict supported secured columns beyond row access. Use for sensitive fields such as confidential assessment or salary-related values.

Hiding a data card or field on a form is not column security. A user with table/column access may retrieve it through another client/API.

## Least privilege role design

1. Start with job tasks.
2. List required operations and tables.
3. Set the smallest access depth.
4. Include related/reference tables needed for lookups.
5. Include Append/Append To deliberately.
6. Test create, read, update, delete, assign, and share paths.
7. Avoid cloning a powerful administrator role and forgetting excess privileges.

Use separate maker/admin roles from end-user roles.

## Business units

Business units participate in the Dataverse security model and role scope. They are not automatically identical to HR departments or the organizational chart. Design them around security boundaries and ownership needs; reorganizing later can be significant.

## Data Loss Prevention (DLP)

DLP policies classify connectors into groups such as business, non-business, or blocked. They control which connectors can be used together in an app/flow and can restrict connectors.

DLP does not inspect your app's intent and does not replace data classification, least privilege, or security review.

Example risk: combining a corporate Dataverse connector with a consumer data-transfer connector. A DLP policy can prevent the combination.

## Environment governance

Governance decisions include:

- Environment purpose and ownership.
- Who can create apps/flows.
- Managed Environments capabilities where licensed.
- DLP and connector policies.
- Capacity monitoring.
- Tenant isolation and network controls where applicable.
- Application inventory and support ownership.
- Orphaned resource process.
- ALM requirements.
- Logging, audit, retention, backup, and recovery.

A Center of Excellence can provide standards and visibility, but governance should enable safe delivery, not just create paperwork.

## Secrets and configuration

- Never embed passwords or client secrets in Power Fx.
- Use connection authentication and approved secret-management patterns.
- Put environment-specific nonsecret configuration in environment variables.
- Use service principals/application users only with deliberate least privilege and supported connectors/APIs.
- Rotate credentials and plan owner departure.

## Security test matrix

Test with real nonadmin personas:

| Test | Requester | Manager | Operations |
|---|---:|---:|---:|
| Create own request | Allow | Optional | Optional |
| Read own request | Allow | Allow if in scope | Allow |
| Read unrelated request | Deny | Only intended scope | Allow if job requires |
| Edit submitted business fields | Deny/limited | Limited | Controlled |
| Approve | Deny | Allow in scope | According to process |
| Delete | Usually deny | Usually deny | Restricted |
| Manage departments | Deny | Deny | Admin role only |

Run tests through canvas, model-driven app, flow, and any API/integration path.

## Common mistakes

- `Visible = User().Email = "admin@..."` as security.
- Testing only while logged in as System Administrator.
- Sharing the app but not the data.
- Giving organization-level privileges to fix one missing lookup permission.
- Forgetting Append/Append To.
- Personal flow owner with broad permissions and no continuity plan.
- Treating DLP as a complete security control.

## Interview questions

**Are security roles additive?**  
Generally, a user's effective Dataverse privileges combine across assigned roles/teams; a restrictive role does not subtract a privilege granted by another role.

**How do you secure rows in Dataverse?**  
Through table ownership, security roles and access depths, teams, business units, hierarchy/security features, and explicit sharing as appropriate.

**Why is `Visible=false` not security?**  
It only changes one UI. It does not remove underlying data/API privileges.

**What does DLP do?**  
It governs connector use and combinations across Power Platform to reduce inappropriate data movement; it is not row-level access control.

## Challenge

A manager can open the app but receives an error when selecting a Department lookup. What do you check?

### Answer

Check the manager's Read privilege/depth on Department plus Append on Request and Append To on Department for association, record availability/ownership, app/table inclusion, and any column security. Do not solve it immediately with System Administrator.

## Primary references

- [Security in Dataverse](https://learn.microsoft.com/power-platform/admin/wp-security)
- [Security roles and privileges](https://learn.microsoft.com/power-platform/admin/security-roles-privileges)
- [Share a canvas app](https://learn.microsoft.com/power-apps/maker/canvas-apps/share-app)
- [Data policies](https://learn.microsoft.com/power-platform/admin/wp-data-loss-prevention)

