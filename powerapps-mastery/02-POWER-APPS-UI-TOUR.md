# 02 — Power Apps UI Tour

## Goal

Navigate the maker portal and Power Apps Studio confidently, know where formulas live, and understand the save/publish lifecycle.

UI labels move as Microsoft updates the authoring experience. Use the concepts and nearby icons if a label differs slightly in your tenant.

## Part A: the maker portal

Open `https://make.powerapps.com` and sign in.

### The environment selector

Look at the upper-right area and confirm the active environment before doing anything.

Why it matters:

- Apps and flows belong to an environment.
- Each environment can have at most one Dataverse database.
- Permissions, DLP policies, connections, and solution contents vary by environment.

Say the environment name aloud before creating or deleting anything. This simple habit prevents production mistakes.

### Left navigation

Depending on your permissions and current UI, you will see entries such as:

| Area | Use it for |
|---|---|
| Home | Starting points and recent work. |
| Create | Create an app from data, a design, or a blank canvas. |
| Learn | Guided Microsoft learning content. |
| Apps | Open, edit, play, share, export, or inspect apps. |
| Tables / Dataverse | Create and configure Dataverse tables. |
| Flows | View or create Power Automate cloud flows. |
| Solutions | Package components and manage ALM. |
| More | Find areas hidden from the pinned navigation. |

Pin frequently used areas when the UI provides a pin option.

### Apps page commands

Navigate: **Power Apps maker portal > Apps**.

Select an app or open its context menu. Common actions include:

- **Play:** run the published version.
- **Edit:** open the editable draft in Studio.
- **Details:** inspect identifiers, owner, dates, and links.
- **Share:** grant users or groups access.
- **Monitor:** diagnose runtime behavior.
- **Delete:** remove the app; use cautiously.

## Part B: create a blank canvas app

The exact landing path can vary. A common route is:

1. Select **Create**.
2. Choose a canvas app starting option.
3. Choose **Blank canvas** or **Start with a blank design**.
4. Enter `Employee Request Hub - Training`.
5. Choose a phone or tablet format if prompted.
6. Select **Create**.

If Microsoft offers AI-assisted or data-first starting paths, deliberately choose blank for this lesson so you learn Studio.

## Part C: Studio anatomy

### 1. Command bar

The top command area commonly provides:

- Back to the maker portal.
- Undo and redo.
- Insert.
- Add data.
- New screen.
- Settings.
- App checker.
- Preview/play.
- Save and publish.

Keyboard preview shortcut is often **F5**. Use the on-screen play button if the browser or operating system intercepts it.

### 2. Left rail

Common panels:

- **Tree view:** screen/control hierarchy.
- **Insert:** controls, layouts, icons, media, and components.
- **Data:** connected data sources.
- **Variables:** global and context variables and collections observed by Studio.
- **Media:** uploaded images, video, and audio.
- **Power Automate:** add or create flows callable by the app.
- **Advanced tools:** features such as Live Monitor, depending on the current UI.

### 3. Canvas

The center is the design surface. Select a control to edit it. A blue or highlighted boundary shows selection. Controls inside auto-layout containers are positioned by layout rules, not arbitrary X/Y dragging.

### 4. Property selector and formula bar

This is the most important area in canvas development.

Every control has properties. Some describe appearance (`Color`, `Fill`, `FontSize`), some layout (`Width`, `Height`), some data (`Items`, `Item`), and some behavior (`OnSelect`, `OnChange`).

Workflow:

1. Select a control.
2. Choose a property from the property selector near the formula bar.
3. Enter a Power Fx formula.
4. Read any error/warning markers.

Example: select a button, choose `OnSelect`, and enter:

```powerfx
Notify("Saved successfully", NotificationType.Success)
```

### 5. Properties pane

The right pane offers common configuration without writing every formula manually. It may expose:

- Text or value.
- Data source and fields.
- Display mode.
- Size, alignment, and styling.
- Accessibility labels.
- Advanced properties.

Anything configured visually ultimately maps to properties. Learn to inspect the corresponding formulas.

## Rename controls immediately

Tree View action:

1. Select a control.
2. Open its context menu or press the rename shortcut available in your Studio.
3. Enter a meaningful name such as `btnNewRequest`.

Compare:

```powerfx
Button7.OnSelect
```

with:

```powerfx
btnNewRequest.OnSelect
```

The second is easier to debug, review, and discuss in an interview.

## Screens and app-level objects

Tree View usually contains:

- **App:** app-level properties such as `OnStart`, `StartScreen`, and named formulas where supported.
- **Screens:** pages such as `scrHome` and `scrEditRequest`.
- **Controls:** nested under screens, galleries, forms, containers, and components.

Avoid loading all application data in `App.OnStart`. It can delay startup and create stale copies. Use named formulas, delegable queries, or screen-level loading only where justified.

## Draft, saved, published

Understand the lifecycle:

1. **Edit:** you change the draft in Studio.
2. **Save:** the draft is stored as a version.
3. **Publish:** the selected saved version becomes available to users.
4. **Share:** grants app access, but underlying resources still need permissions.

If you save but do not publish, existing users normally continue to run the last published version.

## Guided lab: build a UI map

1. Create the blank app.
2. Rename the first screen `scrHome`.
3. Insert a vertical container and rename it `conPage`.
4. Insert a label inside it and rename it `lblPageTitle`.
5. Set the label text/value to `"Employee Request Hub"`.
6. Insert a button named `btnNewRequest`.
7. Set its visible caption to `"New request"`.
8. Set `OnSelect` to:

```powerfx
Notify("The request form comes next", NotificationType.Information)
```

9. Preview the app and select the button.
10. Close preview, save, and name the version `UI tour checkpoint` if version notes are offered.

## Formula-bar reading drill

Explain these property/formula pairs:

| Control.Property | Formula | Meaning |
|---|---|---|
| `lblPageTitle.Text` | `"Employee Request Hub"` | Static displayed text. |
| `btnNewRequest.DisplayMode` | `If(varCanCreate, DisplayMode.Edit, DisplayMode.Disabled)` | Button availability depends on state. |
| `conPage.Width` | `Parent.Width` | Container fills its parent width. |
| `scrHome.OnVisible` | `Set(varPageName, "Home")` | Set state when the screen becomes visible. |

## Common mistakes

- Editing the wrong environment.
- Confusing a control's visible caption property with `OnSelect` behavior.
- Typing a formula on the wrong property.
- Leaving controls named `Button1`, `Label14`, and `Gallery3`.
- Saving without publishing and expecting users to see the change.
- Sharing the app without sharing or securing its data.
- Dragging controls after assigning responsive X/Y formulas, which can overwrite the formulas.

## Interview questions

**Where does logic live in a canvas app?**  
In formulas assigned to properties of the app, screens, controls, components, and data interactions. Behavior properties such as `OnSelect` can run actions; declarative properties calculate values.

**What is the difference between Save and Publish?**  
Save stores a draft version. Publish makes a saved version available to users.

**Why is Tree View important?**  
It shows hierarchy, makes selection reliable, controls ordering in containers, and helps maintain meaningful names and structure.

## Challenge and answer

**Challenge:** Make `btnNewRequest` disabled when `varCanCreate` is false.

Set `btnNewRequest.DisplayMode` to:

```powerfx
If(varCanCreate, DisplayMode.Edit, DisplayMode.Disabled)
```

For a quick test, set `App.OnStart` to `Set(varCanCreate, false)`, run **App > Run OnStart** if available, and preview. Remove this temporary initialization after the test.

## Primary references

- [Create a canvas app](https://learn.microsoft.com/power-apps/maker/canvas-apps/create-blank-app)
- [Understand Power Apps Studio](https://learn.microsoft.com/power-apps/maker/canvas-apps/power-apps-studio)
- [Save and publish a canvas app](https://learn.microsoft.com/power-apps/maker/canvas-apps/save-publish-app)

