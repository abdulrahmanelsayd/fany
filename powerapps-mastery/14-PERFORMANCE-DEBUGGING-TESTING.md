# 14 — Performance, Debugging, and Testing

## Goal

Diagnose canvas apps methodically, reduce startup and network cost, handle errors, and verify behavior with repeatable tests.

## Performance model

User-perceived time comes from:

```text
startup + connector latency + server query + data transfer
+ client calculation + rendering + repeated/unnecessary work
```

Optimize after measuring, but design for delegation and narrow data transfer from the beginning.

## Diagnose in layers

1. Reproduce with exact user, environment, device, and record.
2. Check Studio formula errors/warnings.
3. Use App Checker.
4. Inspect Live Monitor events.
5. Check browser/network/service health where allowed.
6. Inspect flow run history and Dataverse/plugin traces when relevant.
7. Compare administrator versus real-user behavior to isolate security.

## App Checker

Open the app in Studio and select the **App checker** icon/command. Review:

- Formula errors.
- Warnings and delegation.
- Accessibility findings.
- Performance suggestions.

Do not blindly silence warnings. Understand whether each finding affects correctness, accessibility, performance, or maintainability.

## Live Monitor

Typical authoring path:

1. Open app in Studio.
2. Select **Advanced tools** or the current Monitor command.
3. Select **Open live monitor**.
4. In the connected app session, reproduce one scenario.
5. Filter events and inspect duration, data source, response, formula, and errors.

Live Monitor can diagnose the published app through supported launch paths. Publishing source expressions for monitoring can affect app performance and expose formula details to appropriately privileged viewers; enable only when needed and disable afterward.

Look for:

- Repeated connector calls.
- Large response sizes.
- Slow queries.
- Delegation-related behavior.
- 401/403 permission failures.
- 429 throttling.
- Flow invocation duration/errors.
- Control formulas recalculating excessively.

## Error handling

### IfError

```powerfx
IfError(
    Patch(Requests, varRequest, {Title: Trim(txtTitle.Value)}),
    Notify("Save failed. Try again.", NotificationType.Error),
    Notify("Saved.", NotificationType.Success)
)
```

### Errors

After a data operation, `Errors(DataSource)` can expose data-source errors for supported operations. Map technical errors to safe user messages and capture diagnostic details for support.

### App.OnError

Use `App.OnError` for unhandled-error observation/logging and a final safe notification. It does not replace local `IfError` because it cannot turn a failed operation into a successful value/path.

Conceptual pattern:

```powerfx
Trace(
    "Unhandled app error",
    TraceSeverity.Error,
    {
        Message: FirstError.Message,
        Source: FirstError.Source
    }
);
Notify("Something went wrong. Please try again.", NotificationType.Error)
```

Review telemetry privacy. Do not log secrets or unnecessary personal/business data.

## Trace

Use `Trace` to add useful diagnostic events visible in Monitor/telemetry-supported paths:

```powerfx
Trace(
    "Request submit started",
    TraceSeverity.Information,
    {RequestId: Text(varSelectedRequestId)}
)
```

Include correlation identifiers and stage names, not entire sensitive records.

## Startup optimization

Avoid a large sequential `App.OnStart` that downloads reference and transaction tables.

Use:

- `App.StartScreen` for start selection.
- Named formulas for calculated configuration.
- Direct delegable gallery queries.
- Screen-specific loading.
- `Concurrent` only for independent calls.

```powerfx
Concurrent(
    ClearCollect(colRequestTypes, Filter(RequestTypes, Active = true)),
    ClearCollect(colDepartments, Filter(Departments, Active = true))
)
```

Do not use `Concurrent` when one call depends on the result of another, and do not cache large sources without measurement.

## Reduce data and calls

- Filter at the server.
- Request only necessary rows and, where supported, columns.
- Avoid connector calls inside gallery templates.
- Avoid repeated `LookUp` per visible row when relationships/query shape can provide the data efficiently.
- Reuse a returned `Patch`/`LastSubmit` record.
- Avoid unconditional `Refresh`.
- Keep media optimized.
- Reduce excessive controls and deep nested layout where it affects rendering.

### N+1 query smell

If a gallery shows 100 requests and each template performs a separate lookup for Department, you may generate many calls. Prefer Dataverse relationships/direct related values, a server query, or a justified small reference cache.

## Delay or explicitly submit search

Use delayed text output where supported or a Search button to avoid querying on every keystroke. Monitor actual calls to confirm behavior.

## Loading experience

Track operations that the app starts:

```powerfx
UpdateContext({locIsLoading: true});
IfError(
    Refresh(Requests),
    Notify("Refresh failed.", NotificationType.Error)
);
UpdateContext({locIsLoading: false})
```

Ensure every success and failure path resets the flag. For built-in connector loading state, the app also exposes data-source/loading signals in some contexts, but explicit operation state gives precise UX control.

## Testing pyramid

| Level | Test |
|---|---|
| Formula/component | Input-output behavior and edge cases. |
| App path | Create/edit/search/navigation/error flows. |
| Integration | Dataverse, connectors, flows, approvals. |
| Security | Each persona and access boundary. |
| Performance | Realistic volume, network, device. |
| UAT | Business acceptance in test environment. |

## Test cases for Employee Request Hub

### Functional

- Create a valid request.
- Required title missing.
- Needed By in past.
- Edit draft.
- Attempt edit after submission.
- Cancel/delete according to policy.
- Search and filter combinations.

### Security

- Requester tries unrelated GUID.
- Manager outside scope tries request link.
- Operations user accesses configuration.
- Flow is called with manipulated email/amount.

### Reliability

- Network disconnect during save.
- Duplicate Submit selection.
- Flow timeout.
- Approval response repeated.
- Record changed by another user.

### Scale

- Source exceeds delegation limit.
- Gallery contains enough rows/images to expose rendering issues.
- Search executed rapidly.
- Concurrent users submit requests.

## Test Studio and automated tests

Power Apps testing capabilities evolve. Where the current tenant provides Test Studio or test authoring, create suites for stable critical paths, but keep integration and security tests outside a purely mocked UI test. Use accessible selectors/control names and avoid timing assumptions.

At minimum, maintain a Markdown test matrix with steps, data, persona, expected result, actual result, evidence, and defect ID.

## Performance review checklist

- No unexplained delegation warnings.
- Startup does not download large business tables.
- No connector calls repeated per gallery row.
- Search is delayed/explicit and delegable.
- Independent initialization calls are concurrent only when useful.
- Images/media are appropriately sized.
- Forms/galleries contain only necessary controls.
- Monitor shows expected call count and payload.
- Test uses realistic user privileges and data volume.

## Common mistakes

- Optimizing formulas without measuring network calls.
- Raising the data row limit to “fix” delegation.
- Using `Concurrent` on dependent operations.
- Hiding errors and navigating as though save succeeded.
- Logging raw records or sensitive error bodies.
- Testing only as administrator on fast office Wi-Fi.
- Publishing source expressions for Monitor and leaving the performance-impacting setting enabled.

## Interview questions

**How do you troubleshoot a slow canvas app?**  
Reproduce consistently, inspect App Checker and delegation, use Live Monitor to locate slow/repeated calls or rendering events, then reduce server work, payload, network round trips, and client controls based on evidence.

**When should you use `Concurrent`?**  
For independent operations whose results are not required by each other. It reduces sequential waiting but does not make the backend faster or fix excessive calls.

**`IfError` versus `App.OnError`?**  
`IfError` handles a specific expression and can provide an alternative path/value. `App.OnError` observes unhandled errors for notification/logging but cannot replace the failed operation's result.

## Challenge

Monitor shows 80 Department lookups when opening a gallery with 80 requests. What is the likely problem?

### Answer

An N+1 query pattern—each gallery row performs a separate lookup. Use the relationship's available related value, reshape the server query, or cache a genuinely small Department reference table and perform local lookups.

## Primary references

- [Live Monitor for canvas apps](https://learn.microsoft.com/power-apps/maker/monitor-canvasapps)
- [Error handling guidance](https://learn.microsoft.com/power-apps/guidance/coding-guidelines/error-handling)
- [Canvas app performance optimization](https://learn.microsoft.com/power-apps/maker/canvas-apps/app-performance-considerations)
- [Concurrent function](https://learn.microsoft.com/power-platform/power-fx/reference/function-concurrent)

