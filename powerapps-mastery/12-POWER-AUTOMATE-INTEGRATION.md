# 12 — Power Automate Integration

## Goal

Trigger reliable automation from a canvas app, pass typed inputs, return a result, and design approval flows that remain secure and supportable.

## Decide what triggers the flow

| Trigger | Best when |
|---|---|
| Power Apps (V2) | A user explicitly requests an immediate action and the app needs a result. |
| Dataverse row added/modified | The process must run regardless of which app/API changed the record. |
| Scheduled | Periodic reminder, cleanup, or reconciliation. |
| Approval response | Continue when an approver decides. |

Important design insight: if every submitted Request must enter approval, triggering on the Dataverse status change is usually more robust than relying only on one canvas button.

## Lab A: call a simple flow from the app

### Create the flow

In the canvas app Studio:

1. Open the **Power Automate** pane on the left.
2. Select **Create new flow**.
3. Choose **Create from blank** or the relevant template.
4. Use the **Power Apps (V2)** trigger.
5. Add text input `RequestId`.
6. Add text input `RequesterEmail` only if the flow truly needs it; do not trust it for authorization.
7. Add a Dataverse **Get a row by ID** action for Request using `RequestId`.
8. Add **Respond to a PowerApp or flow**.
9. Add text output `Result` and text output `Message`.
10. Save with name `ERH - Submit request`.
11. Return to Studio and add/refresh the flow in the Power Automate pane.

The generated callable name appears through Studio autocomplete.

### Call it

Conceptual button formula:

```powerfx
If(
    !locSubmitting,
    UpdateContext({locSubmitting: true});
    IfError(
        Set(
            varFlowResponse,
            'ERH-Submitrequest'.Run(Text(varSelectedRequestId), User().Email)
        );
        If(
            varFlowResponse.result = "Success",
            Notify(varFlowResponse.message, NotificationType.Success),
            Notify(varFlowResponse.message, NotificationType.Error)
        ),
        Notify("The submission service could not be reached.", NotificationType.Error)
    );
    UpdateContext({locSubmitting: false})
)
```

Actual generated flow/function/output names may remove spaces or use different casing. Insert them with autocomplete rather than typing from memory.

## Never trust app-supplied identity or price

Inputs from the app can be manipulated. The flow should:

- Use its trigger/user context and data-layer security where possible.
- Retrieve authoritative Request values by ID.
- Verify the caller may perform the operation.
- Recalculate sensitive totals or approval thresholds server-side.
- Validate current status before changing it.

UI validation improves experience; server validation protects the business process.

## Approval architecture

Robust sequence:

1. User changes Request Status from Draft to Submitted.
2. Dataverse-triggered flow detects the transition.
3. Flow validates required fields and manager.
4. Flow creates an Approval Event/pending record or approval request.
5. Approver responds.
6. Flow updates Request status and decision metadata.
7. Flow creates immutable decision history.
8. Flow notifies requester.
9. Failure path records operational status and alerts support.

## Configure the Dataverse trigger carefully

For a row added/modified trigger, configure:

- Change type.
- Table name.
- Scope.
- Select columns/filter columns where supported.
- Filter expression/trigger condition to avoid unnecessary runs.

Avoid a loop:

```text
Flow triggers on any Request update
-> Flow updates Request
-> Update triggers the same flow again
```

Prevent this using narrow filter columns, trigger conditions, distinct state transitions, or a separate process marker. Design and test retries because cloud flows can retry actions.

## Idempotency

An idempotent process can safely receive the same event more than once without creating duplicate outcomes.

Pattern:

- Store a process/correlation ID.
- Check whether an Approval Event for that submission version already exists.
- Create only when absent.
- Use alternate keys or server constraints where appropriate.

Do not depend only on “the flow normally runs once.”

## Child flows and solution awareness

For reusable automation:

- Create flows inside the solution.
- Use connection references.
- Use environment variables for URLs, mailbox addresses, and configuration.
- Use child flows for shared operations where licensing/support requirements are met.
- Avoid hard-coded environment-specific IDs.

Lesson 16 covers deployment.

## Run-only users and connections

For instant flows, configure run-only behavior deliberately:

- Which connections are supplied by the run-only user?
- Which connections are embedded/owned by a service account?
- What permissions does that identity have?
- What happens if the owner leaves?

Use least privilege and an organizational ownership/support model for critical flows.

## Error handling pattern in flows

Use scopes:

```text
Scope - Try
  Validate
  Update data
  Start approval

Scope - Catch (run after Try failed/timed out)
  Record error/correlation ID
  Notify support
  Return safe error when synchronous

Scope - Finally
  Cleanup/telemetry
```

Do not expose full raw connector errors, tokens, URLs, or personal information to the end user.

## Timeouts and asynchronous work

An app-triggered flow call makes the user wait for a response and is constrained by platform timeouts. For long work:

1. App creates a job/request row.
2. Background flow processes it.
3. App shows `Queued`/`Processing`.
4. User refreshes or the app polls conservatively.
5. Flow records `Succeeded`/`Failed` and a user-safe message.

Do not keep a user staring at a spinner while a multi-minute integration completes.

## Attachments and files

When passing files from Power Apps to a flow:

- Confirm the trigger supports the required file input structure.
- Validate type and size.
- Use an appropriate document store for large/managed documents.
- Scan and apply retention/compliance policies.
- Avoid passing large base64 payloads unnecessarily.

## Flow testing checklist

- Correct app user and connection identity.
- Missing/invalid inputs.
- Unauthorized record ID.
- Record already submitted.
- Duplicate/retried trigger.
- Connector timeout/throttling.
- Approver missing or inactive.
- Approval rejected.
- Owner connection disabled.
- Deployment to test using different connection references/configuration.

## Common mistakes

- Trusting `User().Email` passed by the app as authorization.
- Calling a long-running flow synchronously.
- No response action when the app expects an output.
- Updating the trigger row and creating an infinite loop.
- Personal connections with no support/continuity plan.
- Hard-coded URLs and IDs.
- No idempotency or duplicate-event handling.

## Interview questions

**When should an app call a flow directly?**  
When a user initiates an immediate action that requires automation or a response. Use a data-change trigger when the business process must run regardless of client.

**How do you prevent a flow-trigger loop?**  
Limit trigger columns and conditions, model explicit state transitions, avoid updating triggering fields unnecessarily, and use process markers/correlation data.

**What is a connection reference?**  
A solution component that lets apps/flows refer to a connection abstractly so the actual connection can be bound per environment.

**What is idempotency?**  
Designing an operation so duplicate execution does not create duplicate business effects.

## Challenge

The app passes `Amount=100` to the flow, but the saved Request says `Amount=5000`. Which value determines approval?

### Answer

The flow must retrieve and validate the authoritative saved value from Dataverse. Never use the app-supplied amount for a sensitive threshold.

## Primary references

- [Use the Power Automate pane in Power Apps](https://learn.microsoft.com/power-apps/maker/canvas-apps/working-with-flows)
- [Trigger a flow from a canvas app](https://learn.microsoft.com/power-apps/maker/canvas-apps/how-to/trigger-flow)
- [Dataverse trigger filtering](https://learn.microsoft.com/power-automate/dataverse/create-update-delete-trigger)
- [Power Automate error handling guidance](https://learn.microsoft.com/power-automate/guidance/coding-guidelines/error-handling)

