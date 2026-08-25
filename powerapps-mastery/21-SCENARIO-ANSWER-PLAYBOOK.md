# 21 — Scenario Answer Playbook

## Why scenario questions matter

Interviewers rarely care only whether you remember a button. They want to see whether you protect data, choose the right layer, recognize scale, and can investigate uncertainty.

## The SMARTER answer framework

Use this order:

1. **S — Scope:** users, devices, process, data volume, latency, external/internal, and licensing constraints.
2. **M — Model:** authoritative data, tables/relationships, ownership, and lifecycle.
3. **A — App choice:** canvas, model-driven, custom page, Pages, or no app—and why.
4. **R — Rules and security:** where validation and authorization are enforced.
5. **T — Transactions/automation:** flows, triggers, idempotency, errors, concurrency.
6. **E — Efficiency:** delegation, indexes, network calls, Monitor, responsive/accessibility.
7. **R — Release:** solutions, environments, configuration, testing, deployment, support.

You will not always say all seven. Thinking through them prevents shallow answers.

## Clarifying questions that make you sound professional

Before choosing architecture, ask two or three:

- Who are the users: employees, guests, customers, anonymous users?
- What is the authoritative source and current data volume/growth?
- Must users work offline or on mobile devices?
- Are record/column access rules different by user, manager, team, or region?
- Is this an immediate interaction or background process?
- What are the expected concurrency and integration rates?
- Which licenses/connectors/environments are already available?
- What audit, retention, data residency, and recovery obligations exist?

## Scenario 1 — Employee request application

**Question:** Design an app where employees submit requests and managers approve them.

**Strong answer:**

> I would clarify volume, approval routing, mobile need, attachment sensitivity, and manager access scope. I would use a responsive canvas app for simple employee intake and a model-driven app for manager/operations work over Dataverse. Dataverse would contain user/team-owned Request rows related to Department, Comments, and Approval Events. Submission changes a validated status; a Dataverse-triggered idempotent flow retrieves authoritative data, creates approval, writes decision history, and notifies the requester. Roles/teams—not canvas visibility—enforce record access. I would package everything in a solution with connection references/environment variables and deploy managed builds through dev, test, and production.

**Likely follow-up:** Why not SharePoint?  
SharePoint could fit a smaller list/document process, but Dataverse better fits relational approval history, row security, model-driven operations, auditing, and ALM. I would confirm licensing and requirements before deciding.

## Scenario 2 — App works for maker only

**Question:** The maker can use the app; other users get an error.

**Answer path:**

1. Reproduce with an affected account; capture exact operation/error.
2. Confirm correct environment, published version, and app sharing.
3. Check user license.
4. Check data-source permissions/security roles/access depth.
5. Check connection prompts and flow run-only connections.
6. Check custom connector/API permissions and DLP.
7. Avoid granting System Administrator as the fix.

## Scenario 3 — Missing records in a gallery

**Question:** A gallery works in development but misses production records.

**Strong answer:**

> I would suspect delegation before assuming corrupt data. I would inspect Studio warnings and the connector-specific support for every function, lower the nondelegable row limit in a test copy, and use Monitor with realistic data. I would rewrite using server-supported filters/sorts, normalize/index searchable columns, or use source-native search. Loading all rows into a collection or raising the row limit would not be my production fix.

## Scenario 4 — `Patch` or form?

**Question:** You need to create and edit a request. Which approach?

**Strong answer:**

> For a standard record form I would start with an Edit Form because data cards, required fields, validity, and `OnSuccess`/`OnFailure` are built in. I would use `Patch` for a highly custom experience or targeted changes, but then I own type mapping, validation, reset, accessibility, and error behavior. Either way, important rules and security are enforced in Dataverse/server logic.

## Scenario 5 — Sensitive button hidden by email

**Question:** Approve button is visible only for a hard-coded manager email. Is that secure?

**Strong answer:**

> No. That is only UI personalization and is brittle. I would model authorization through Dataverse roles, teams, ownership/access depth, and server-side status/approver validation in the flow or operation. The button can reflect effective permissions for UX, but hiding it is never the security boundary.

## Scenario 6 — Duplicate approval emails

**Question:** Two approval requests are sometimes created.

**Answer path:**

- Check trigger retrial, duplicate app submission, and self-trigger loops.
- Introduce a submission/correlation ID and unique pending-approval constraint/check.
- Make the flow idempotent.
- Restrict trigger columns/conditions to a real status transition.
- Disable UI double-submit, but do not rely only on the client flag.
- Store approval event/run correlation for support.

## Scenario 7 — Slow canvas app

**Question:** The app takes 20 seconds to open.

**Strong answer:**

> I would measure first. App Checker shows formula/delegation/accessibility findings; Live Monitor shows call counts, duration, payload, errors, and repeated queries. I would inspect large sequential `App.OnStart`, whole-table collections, N+1 gallery lookups, repeated refresh, heavy media, and too many controls. I would use delegable narrow queries, direct relationships, named formulas/screen loading, and `Concurrent` only for independent calls.

## Scenario 8 — Excel requested as production database

**Question:** The business already has an Excel file and wants 200 employees to update it.

**Strong answer:**

> I would explain that connector availability does not make Excel a transactional multi-user database. Named table, locking/concurrency, schema stability, delegation, permissions, and growth create risk. I would assess SharePoint for a simple list process, or Dataverse/SQL for relational/security/scale needs, and plan data migration rather than promising Excel will scale.

## Scenario 9 — SharePoint or Dataverse

**Question:** How do you choose?

**Answer dimensions:**

- Relationships and data integrity.
- Row/column security and ownership.
- Volume/delegation/transactions.
- Model-driven requirement.
- Auditing/business rules/APIs.
- Documents and existing SharePoint investment.
- ALM/integration.
- Licensing and admin capacity.

Do not answer “Dataverse is always better” or “SharePoint is free.”

## Scenario 10 — Manager lookup error

**Question:** User can create Request but cannot choose Department.

**Strong answer:**

> I would check Read access to Department and the relationship privileges—typically Append on Request and Append To on Department—plus access depth/row availability, column security, and app inclusion. I would test with the actual role and avoid giving broad organization access before identifying the missing privilege.

## Scenario 11 — Direct flow call with amount

**Question:** Canvas calls a flow with Request ID and Amount to decide approval.

**Strong answer:**

> The flow should treat client Amount as untrusted. It should retrieve the saved Request using the ID, validate the caller or process identity and current status, calculate the threshold from authoritative data, and use least-privilege connections. For a mandatory process, I would consider a Dataverse change trigger so another client cannot bypass it.

## Scenario 12 — Deploy to production

**Question:** Explain your release process.

**Strong answer:**

> I develop in an unmanaged solution in development. Components include apps, tables, flows, roles, environment variables, and connection references. I publish and run app/solution checks, increment version, export/build a managed artifact, deploy to test, bind configuration/connections, activate flows in controlled order, run persona/security/UAT tests, then promote the same artifact to production through a pipeline. I define monitoring and rollback/recovery before release and avoid direct production edits.

## Scenario 13 — Production still shows old behavior

**Question:** A managed solution imported successfully, but behavior did not change.

**Answer path:**

- Confirm correct environment/package/version/import history.
- Publish component/app customizations.
- Inspect solution layers and unmanaged active layer.
- Check missing dependencies and environment variable/connection values.
- Check flow activation and app published version.
- Reproduce in a clean user session only after platform checks.

## Scenario 14 — Offline field app

**Question:** Technicians need to work offline with Dataverse records.

**Strong answer:**

> I would clarify supported mobile client, minimum tables/columns/relationships, sensitive data on device, expected offline duration, attachment needs, conflict policy, and sync volume. I would evaluate Dataverse mobile offline/offline-first support and profiles rather than inventing a collection-only sync engine. I would test unsupported table/control/features and enforce device/governance policies.

## Scenario 15 — On-premises SQL

**Question:** Canvas app needs data from on-prem SQL.

**Strong answer:**

> I would validate connector/licensing and use the on-premises data gateway as the secure bridge. I would plan gateway ownership, cluster/high availability, patching, credentials/least privilege, network/firewall, delegable SQL queries, views/stored procedures where supported, and monitoring. The app should not retrieve an entire table through the gateway.

## Scenario 16 — Complex custom calendar/control

**Question:** Standard controls cannot deliver a required scheduling interaction.

**Strong answer:**

> I would first validate whether modern controls, containers, a canvas custom page, or an existing supported component meets the need. If not, PCF is the pro-code option for a reusable control, but it brings accessibility, security, performance, dependency, test, deployment, and support responsibilities. I would justify it with an essential requirement, not cosmetic preference.

## Scenario 17 — External suppliers

**Question:** Suppliers outside the tenant need to submit requests.

**Strong answer:**

> I would not assume internal Power Apps sharing is the right experience. I would clarify identity, anonymous/authenticated access, volume, documents, licensing, and data isolation, then evaluate Power Pages with Dataverse web roles/table permissions or another approved external architecture. External UI still needs strict server-side permissions and secure file handling.

## Scenario 18 — Bulk update 20,000 records

**Question:** User selects 20,000 requests and changes status.

**Strong answer:**

> I would not use a client `ForAll(Patch(...))` loop without strong evidence. It creates many requests, throttling, partial success, timeout, and audit problems. I would use an asynchronous server-side/batch process such as a Dataverse custom operation/plugin, supported bulk API, or queued flow design with authorization, idempotency, progress, and failure reporting.

## Scenario 19 — Two users overwrite changes

**Question:** Managers edit the same request and one comment disappears.

**Strong answer:**

> I would define a concurrency policy rather than accept silent overwrite. For approval decisions, immutable related Approval Event records are safer than shared comment columns. For editable records, use supported conflict detection/version data, re-read before sensitive transitions, and show a refresh/review path. Server-side status validation prevents invalid simultaneous transitions.

## Scenario 20 — AI-generated app

**Question:** Copilot generated the solution. Is it ready?

**Strong answer:**

> No generated result should skip review. I would validate data types/relationships, security and identity, delegation, connector licensing/DLP, accessibility, errors, concurrency, flow authorization/idempotency, environment configuration, test coverage, and whether any feature is preview. Copilot and Plans accelerate drafting, not accountability.

## Scenario 21 — Large attachments

**Question:** Users upload many large files with requests.

**Answer path:**

- Clarify size/count/type, retention, search, sharing, compliance, and malware scanning.
- Decide Dataverse file versus SharePoint/document-management architecture.
- Avoid large unnecessary base64 transfer through canvas/flow.
- Validate connector/action limits and upload experience.
- Separate file permission from record UI visibility.

## Scenario 22 — Need a report

**Question:** Managers want “a dashboard.”

**Strong answer:**

> I would clarify operational versus analytical need. A model-driven chart/dashboard can support current work queues and simple summaries. Power BI is more appropriate for semantic modeling, history, cross-source analytics, and governed reporting. Security, refresh latency, licensing, and embedding requirements affect the choice.

## Scenario 23 — Direct production hotfix

**Question:** A user asks you to edit production immediately.

**Strong answer:**

> I would assess severity and follow the incident/change process. Direct production edits create unmanaged layers and drift. The normal path is fix in development, test a managed artifact, deploy with an expedited approved pipeline, and monitor. If emergency policy permits a production action, I document it and reconcile source/layers immediately afterward.

## Scenario 24 — Licensing question

**Question:** Does every user need Premium?

**Strong answer:**

> I would not answer without the exact app, connectors, Dataverse/Dynamics use rights, environment type, flow process, AI/Pages use, and tenant entitlements. I understand the categories, but licensing changes and contract context matter, so I would verify the current Microsoft licensing guide and admin-center information before giving a commitment.

## Scenario 25 — You do not know the feature

**Question:** Have you implemented virtual tables?

**Strong answer:**

> I have not implemented them yet, so I would not claim production experience. My understanding is that virtual tables surface external data through Dataverse metadata without ordinary replication. I would evaluate provider capabilities, supported operations, security mapping, latency/delegation, offline limitations, and licensing before choosing them over integration or data replication.

## Five formula exercises with answer logic

### A. Search submitted requests

```powerfx
Filter(
    Requests,
    Status = "Submitted" &&
    (IsBlank(txtSearch.Value) || StartsWith(Title, txtSearch.Value))
)
```

Then discuss typed Status/Choice syntax and connector delegation.

### B. Validate and submit a form

```powerfx
If(
    frmRequest.Valid,
    SubmitForm(frmRequest),
    Notify("Complete required fields.", NotificationType.Error)
)
```

Navigate in `OnSuccess`; show `frmRequest.Error` in `OnFailure`.

### C. Patch with error handling

```powerfx
IfError(
    Patch(
        Requests,
        Defaults(Requests),
        {Title: Trim(txtTitle.Value), Amount: Value(txtAmount.Value)}
    ),
    Notify("Save failed.", NotificationType.Error),
    Notify("Saved.", NotificationType.Success)
)
```

### D. Role-aware button

```powerfx
btnApprove.Visible = varUserProfile.CanApprove
```

Then explicitly say the server/data operation also verifies authorization.

### E. Date range including full end date

```powerfx
Filter(
    Requests,
    RequestedOn >= dpFrom.SelectedDate &&
    RequestedOn < DateAdd(dpTo.SelectedDate, 1, TimeUnit.Days)
)
```

Use an exclusive upper bound for date-time values and verify delegation.

## Interview communication traps

### Weak

> I would use Dataverse because it is better.

### Strong

> Dataverse is my initial choice because the requirement has related requests, approval history, role-scoped records, a model-driven operations app, and managed ALM. I would still validate expected volume and licensing against SharePoint/SQL alternatives.

### Weak

> I use Monitor.

### Strong

> I reproduce with the affected persona, use App Checker for static warnings and Live Monitor to inspect call duration/count/payload/errors, then change the measured bottleneck and compare again.

### Weak

> I do not know PCF.

### Strong

> I have not built PCF yet. I understand it is the pro-code control framework used when standard controls cannot meet an essential UI need, with added accessibility, security, ALM, and support responsibility.

## Your final mock interview

Record yourself answering these without notes:

1. Power Apps, canvas, model-driven, and Dataverse.
2. `Patch` versus `SubmitForm`.
3. Delegation and a real fix.
4. Security roles and why hidden controls are not security.
5. SharePoint versus Dataverse.
6. App-triggered versus Dataverse-triggered flow.
7. Diagnose a slow app.
8. Managed deployment from dev to production.
9. Offline or gateway awareness.
10. Employee Request Hub architecture in two minutes.

Score each answer:

- 0: wrong/no answer.
- 1: definition only.
- 2: definition and use case.
- 3: definition, decision, example, risk/trade-off.

Aim for 3 on topics 1-8 and at least 2 on topic 9.

## Primary references

- [Power Apps guidance](https://learn.microsoft.com/power-apps/guidance/)
- [Dataverse security](https://learn.microsoft.com/power-platform/admin/wp-security)
- [Power Automate guidance](https://learn.microsoft.com/power-automate/guidance/)
- [Power Platform ALM](https://learn.microsoft.com/power-platform/alm/)

