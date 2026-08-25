# 04 — UI, UX, and Responsive Design

## Goal

Create a canvas app that remains usable on different screen sizes, follows a consistent visual system, and supports keyboard and screen-reader users.

## Responsive design mental model

Responsive design is a hierarchy problem, not a collection of clever X/Y formulas.

```text
Screen
└── conPage (vertical)
    ├── conHeader (horizontal)
    ├── conContent (vertical or horizontal by breakpoint)
    │   ├── conFilters
    │   └── galRequests
    └── conFooter
```

Each parent controls how its children consume available space.

## Step 1: configure responsive behavior

In the app from Lesson 03:

1. Open **Settings**.
2. Open **Display** or the current screen-size settings area.
3. Turn off **Scale to fit**.
4. Turn off **Lock aspect ratio** when it is enabled by scale-to-fit behavior.
5. Allow orientation changes if the target devices require them.
6. Apply the changes.

Turning off scale-to-fit only enables responsive design; it does not make existing absolute layouts responsive automatically.

## Step 2: use auto-layout containers

Navigate to `scrRequestList`.

1. Select **Insert > Layout**.
2. Insert a vertical auto-layout container.
3. Rename it `conPage`.
4. Set width and height to fill the screen:

```powerfx
Width = Parent.Width
Height = Parent.Height
```

5. Move the header and content into the container using Tree View.
6. Add a horizontal container named `conHeader`.
7. Add a flexible content container named `conContent`.

Inside auto-layout containers, use container properties such as gap, padding, alignment, justify, wrap, fill portions, minimum width, and overflow. Avoid fighting the layout engine with X and Y.

## Step 3: define breakpoints

Use named formulas or a consistent formula to classify width. A simple screen formula is:

```powerfx
App.Width < 600
```

Use it to change layout behavior. For example, set a flexible container's direction property to the equivalent of:

```powerfx
If(App.Width < 600, LayoutDirection.Vertical, LayoutDirection.Horizontal)
```

Property names and enum names may vary by container generation. Select the layout property from the property selector and use Studio autocomplete.

Useful conceptual breakpoints:

| Width | Experience |
|---:|---|
| `< 600` | Phone: single column, compact navigation. |
| `600-1023` | Tablet: stacked or two-column where safe. |
| `>= 1024` | Desktop: side filters and wider content. |

Choose breakpoints based on content failure, not a favorite device model.

## Step 4: size relatively

Useful patterns:

```powerfx
Parent.Width
```

```powerfx
Min(720, Parent.Width - 32)
```

```powerfx
Max(280, Parent.Width * 0.3)
```

```powerfx
If(App.Width < 600, false, true)
```

Avoid formulas that reference long chains of siblings. Parent-based sizing and container rules are easier to maintain.

## Step 5: use a design system

Define a small set of design tokens. Depending on Studio capabilities, use named formulas, component properties, or consistent formulas.

Conceptual named formulas:

```powerfx
nfColorPrimary = ColorValue("#0F6CBD")
nfColorSurface = ColorValue("#FFFFFF")
nfColorBackground = ColorValue("#F5F5F5")
nfColorDanger = ColorValue("#C50F1F")
nfSpaceS = 8
nfSpaceM = 16
nfSpaceL = 24
```

The exact named-formula syntax is edited through **App > Formulas** where available. Do not scatter slightly different color literals and spacing values across hundreds of controls.

## Modern controls and themes

Modern controls use the Fluent design direction and centralized themes. In current Studio versions, access commonly appears through the authoring menu's **Themes** area, while feature availability can depend on tenant and release state.

Use modern controls when they meet the functional requirement, but verify:

- Required properties and events exist.
- Accessibility behavior is suitable.
- The control behaves correctly inside containers.
- The feature is supported for production in your tenant.

Do not mix modern and classic controls randomly. If a classic control is necessary, style it intentionally and document why.

## Accessibility checklist

### Text and contrast

- Use meaningful text, not color alone, for status and errors.
- Ensure foreground/background contrast is sufficient.
- Allow readable sizes and spacing.
- Do not encode information only as red/green.

### Keyboard

- Test all interactive controls without a mouse.
- Keep tab order logical.
- Avoid placing non-interactive elements in the tab sequence.
- Provide visible focus indicators.

### Screen readers

- Set accessible labels for icon-only buttons.
- Name controls according to purpose, not shape.
- Provide error summaries and required-field guidance.
- Use headings and container hierarchy consistently.

Examples:

```powerfx
icoBack.AccessibleLabel = "Return to request list"
```

```powerfx
btnSaveRequest.AccessibleLabel =
    If(btnSaveRequest.DisplayMode = DisplayMode.Disabled,
       "Save request. Complete required fields first.",
       "Save request")
```

### Images

- Informative images need alternative text.
- Decorative images should not create screen-reader noise.

## Status design pattern

Display both text and visual emphasis:

```powerfx
Switch(
    ThisItem.Status,
    "Approved", ColorValue("#107C10"),
    "Rejected", ColorValue("#C50F1F"),
    "Submitted", ColorValue("#0F6CBD"),
    ColorValue("#605E5C")
)
```

The visible label still says `Approved`, `Rejected`, or `Submitted`; the color is reinforcement.

## Empty, loading, error, and success states

A professional screen designs more than the happy path.

| State | UI response |
|---|---|
| Loading | Progress indicator and clear status. |
| Empty | Explain that no requests match and offer the next action. |
| Error | Human-readable message plus retry path. |
| Permission denied | Explain required access without exposing sensitive details. |
| Success | Confirmation and resulting record/navigation. |

Example empty label visibility:

```powerfx
CountRows(galRequests.AllItems) = 0 && !varIsLoading
```

Example text:

```powerfx
If(
    IsBlank(Trim(txtSearch.Value)),
    "No requests yet. Create your first request.",
    "No requests match your search."
)
```

## Guided lab: responsive request list

1. Turn off scale-to-fit.
2. Create `conPage`, `conHeader`, and `conContent`.
3. Put search/filter controls in `conFilters`.
4. Put the gallery in `conResults`.
5. On phone widths, stack filters above results.
6. On desktop widths, place a 280-pixel filter panel beside flexible results.
7. Set meaningful accessible labels on back, new, and filter actions.
8. Add an empty-state label.
9. Test at approximately 390, 768, and 1440 pixels wide.
10. Test keyboard-only navigation at each size.

## UX review questions

- Can a first-time user identify the primary action within five seconds?
- Is the same term used everywhere: request, case, or ticket—not all three?
- Does validation say how to fix the problem?
- Are destructive actions confirmed or recoverable?
- Does the interface still work at 200% browser zoom?
- Can a user return without losing work unexpectedly?
- Is sensitive data hidden through actual security, not only `Visible=false`?

## Common mistakes

- Assuming scale-to-fit equals responsive design.
- Nesting many unnecessary containers, making layout hard to reason about.
- Hard-coding widths on every child.
- Using `Visible=false` as a security mechanism.
- Overusing icons with no text or accessible label.
- Showing validation only after save and not placing focus near the problem.
- Creating a beautiful desktop screen that becomes unusable on a phone.

## Interview questions

**How do you build a responsive canvas app?**  
Disable scale-to-fit, use a clear hierarchy of auto-layout containers, size relative to parents, introduce content-driven breakpoints, and test different widths, orientations, zoom levels, and input modes.

**What is the difference between responsive and scaled?**  
A scaled app enlarges or shrinks the same layout. A responsive app rearranges and resizes content to use the available space appropriately.

**Is hiding a control a security measure?**  
No. UI visibility improves experience but does not protect the underlying data or operation. Enforce authorization in Dataverse, the data service, connectors, and flows.

## Challenge

Make the new-request button full width on phones and auto-sized on larger screens.

### Suggested approach

Inside an auto-layout container, set the relevant flexible-width or alignment property based on:

```powerfx
App.Width < 600
```

If using manual width:

```powerfx
If(App.Width < 600, Parent.Width, 160)
```

## Primary references

- [Create responsive layouts in canvas apps](https://learn.microsoft.com/power-apps/maker/canvas-apps/create-responsive-layout)
- [Modern controls and themes](https://learn.microsoft.com/power-apps/maker/canvas-apps/controls/modern-controls/overview-modern-controls)
- [Create accessible canvas apps](https://learn.microsoft.com/power-apps/maker/canvas-apps/accessible-apps)

