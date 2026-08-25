# 15 — Components and Advanced Patterns

## Goal

Turn a working app into a maintainable product by introducing reusable components, a clear architecture, consistent naming, and carefully chosen extensibility.

## Maintainability starts with boundaries

Organize responsibility:

```text
App
├── Named formulas/configuration
├── Navigation and global error policy
├── Screens (user tasks)
│   ├── Containers (layout)
│   ├── Components (reusable UI/behavior)
│   ├── Forms/galleries (data interaction)
│   └── Local context state
├── Flows (automation/integration)
└── Dataverse (authoritative data/security/rules)
```

A screen should not become a thousand-control monolith with every concern mixed together.

## Canvas components

A component packages controls and exposes custom properties.

Good component candidates:

- Application header.
- Navigation rail.
- Confirmation dialog.
- Status badge.
- Empty state.
- Standard error banner.
- Pagination control for an intentionally paged API.

Poor candidates:

- A one-off group with no stable interface.
- A component that depends on many unexplained global variables.
- A wrapper around every single button.

## Create a component

Typical Studio path:

1. Open **Tree View**.
2. Switch to **Components** or select **New component**.
3. Name it `cmpPageHeader`.
4. Set its dimensions responsively.
5. Insert title label, optional subtitle, and action area.
6. Create custom properties through the component properties pane.

Suggested custom properties:

| Property | Direction/type | Purpose |
|---|---|---|
| `Title` | Input, text | Page heading. |
| `Subtitle` | Input, text | Supporting text. |
| `ShowBack` | Input, Boolean | Show back action. |
| `BackLabel` | Input, text | Accessible action label. |
| `OnBack` | Event/behavior property where supported | Let host decide what back means. |

Inside the component, bind the title label to `cmpPageHeader.Title`. Bind back visibility to `cmpPageHeader.ShowBack`.

Avoid direct `Navigate(scrSomeScreen)` inside a reusable component. Expose an event so each host screen controls navigation.

## Input versus output versus event

- **Input:** host passes data/configuration into component.
- **Output:** component exposes a calculated value.
- **Event/behavior:** host supplies behavior that component invokes.
- **Action/function capabilities:** availability and syntax depend on current component-property features.

Prefer a small stable interface. Twenty custom properties can make a component harder to use than the original controls.

## Component libraries

An in-app component is reusable within one app. A component library supports reuse across apps with version/update behavior.

Use a component library when:

- Multiple apps share a design system.
- A team owns and versions common components.
- Updates are tested and communicated.
- Dependencies and release compatibility are managed.

Do not use one giant global library with unrelated components and no owner.

## Named formulas and configuration

Named formulas calculate values based on dependencies and avoid procedural initialization.

Conceptual App `Formulas` entries:

```powerfx
nfCurrentUserEmail = Lower(User().Email);
nfPrimaryColor = ColorValue("#0F6CBD");
nfIsPhone = App.Width < 600;
```

Use environment variables for deploy-time configuration such as service base URL or operations mailbox. Use named formulas to consume/shape app-level values when appropriate. Do not hard-code production configuration in controls.

## Role-aware UI pattern

First, security must exist in Dataverse/service. Then tailor UI.

Prefer retrieving user access or role mapping through an authorized server-side model. Example conceptual query:

```powerfx
Set(
    varUserProfile,
    LookUp(AppUserProfiles, UserEmailNormalized = Lower(User().Email))
)
```

Then:

```powerfx
btnApprove.Visible = varUserProfile.CanApprove
```

The approval operation must still validate authorization server-side. Avoid hard-coded email lists and avoid enumerating complex system-role internals from the client for every control.

## Dialog pattern

```text
Screen
├── Normal page content
└── Dialog overlay (Visible = locShowDeleteDialog)
    ├── Modal surface
    ├── Heading
    ├── Explanation
    ├── Cancel
    └── Confirm
```

Requirements:

- Trap/manage focus as supported.
- Provide accessible title and buttons.
- Prevent background interaction.
- Return focus on close.
- Keep destructive action explicit.
- Do not delete until confirmation.

## Unsaved-changes pattern

Forms expose `Unsaved` in supported form controls:

```powerfx
If(
    frmRequestEdit.Unsaved,
    UpdateContext({locShowDiscardDialog: true}),
    Back()
)
```

For custom controls, compare a structured draft with original values or track changes deliberately. Do not set `locDirty=true` on every `OnChange` and forget to reset it after save.

## Deep links

Use `Param` to read launch parameters:

```powerfx
Param("requestId")
```

Safe pattern:

1. Validate that the parameter is present and correctly typed.
2. Query the record through the data source.
3. Let data security deny unauthorized access.
4. Show a neutral not-found/not-authorized experience.
5. Never treat possession of a URL/GUID as authorization.

Use `App.StartScreen` for declarative initial navigation where possible, but avoid referencing state that is unavailable at evaluation time.

## Bulk operations

Bulk changes need deliberate error and partial-success handling. A naive pattern:

```powerfx
ForAll(colSelectedRequests, Patch(Requests, ThisRecord, {Status: "Submitted"}))
```

may create many calls, delegation limitations, throttling, and partial success. For significant bulk work, prefer a server-side batch/API, Dataverse action/plugin, or background flow designed for scale and idempotency. Report individual failures and preserve audit.

## Custom connectors

Use a custom connector when a supported REST API has no suitable standard connector and Power Platform should consume it declaratively.

Design responsibilities:

- OpenAPI definition/operations.
- Authentication (OAuth preferred where appropriate).
- Request/response schemas.
- Policy, pagination, throttling, and error mapping.
- DLP classification.
- Solution packaging and connection references.
- API lifecycle/versioning.

Never place API secrets in canvas formulas.

## PCF controls

Power Apps Component Framework (PCF) is pro-code extensibility for reusable controls built with supported web technologies/tooling.

Consider PCF when:

- Standard/modern controls cannot meet an essential interaction.
- A high-performance specialized visualization/input is required.
- The organization can develop, secure, test, version, and support code.

Do not choose PCF only to change minor styling. It introduces code, dependency, accessibility, security, deployment, and maintenance responsibilities.

## Coding standards

- Meaningful control/variable/collection names.
- One naming convention across the team.
- Use `With` to clarify complex formulas.
- Comments explain business reason, not obvious syntax.
- No personal emails, environment URLs, or record GUIDs hard-coded.
- No unexplained delegation warnings.
- Error and loading paths for every external operation.
- Reusable components have documented interfaces.
- Avoid experimental features in critical production paths unless risk is accepted.

## Documentation block for each screen

Record:

- User story.
- Data sources.
- Security assumptions.
- Context/global state used.
- Navigation in/out.
- External calls.
- Error states.
- Responsive behavior.
- Test cases.

## Architecture review exercise

Refactor Employee Request Hub:

1. Extract page header and confirmation dialog components.
2. Move colors/spacing into a coherent theme/named formulas.
3. Replace hard-coded manager emails with secure data/configuration.
4. Replace repeated navigation logic with component events or clear screen actions.
5. Add deep-link support by request ID with authorization-safe handling.
6. Inventory every global variable and reduce its scope where possible.

## Common mistakes

- Components coupled to screen names/global variables.
- Component library with no version owner.
- Client-side role checks treated as authorization.
- Giant `App.OnStart` configuration script.
- `ForAll` used for large server-side bulk updates.
- PCF selected for cosmetic reasons.
- Custom connector authentication using embedded secrets.

## Interview questions

**When should you create a component?**  
When a stable UI/behavior pattern repeats and can be expressed through a small clear interface.

**Component versus component library?**  
An app component is local to one app. A component library publishes reusable components for multiple apps and needs version governance.

**Custom connector versus PCF?**  
A custom connector integrates an API/data operation. PCF implements a custom UI control. They solve different layers and can be used together.

**Why avoid global variables inside components?**  
They create hidden coupling, make reuse difficult, and obscure the component's contract.

## Challenge

Three apps need the same header, but each uses different navigation. How should the component work?

### Answer

Publish a header in a component library with input properties for text/visibility and an event property for navigation actions. Each host app supplies its own back/menu behavior.

## Primary references

- [Canvas components overview](https://learn.microsoft.com/power-apps/maker/canvas-apps/create-component)
- [Component libraries](https://learn.microsoft.com/power-apps/maker/canvas-apps/component-library)
- [Named formulas](https://learn.microsoft.com/power-apps/maker/canvas-apps/app-formulas)
- [Power Apps Component Framework overview](https://learn.microsoft.com/power-apps/developer/component-framework/overview)
- [Custom connectors overview](https://learn.microsoft.com/connectors/custom-connectors/)

