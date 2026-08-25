# 18 — Interview Tomorrow: Zero-to-Credible Plan

## Your objective

You do **not** have time to become an experienced Power Apps developer overnight. You do have time to become a credible beginner who:

- Knows the complete platform map.
- Gives accurate definitions without bluffing.
- Can explain one end-to-end solution.
- Understands the high-risk concepts interviewers use to separate makers from button-clickers.
- Can reason through an unfamiliar scenario.
- Clearly distinguishes “I understand the design” from “I have implemented this in production.”

That honesty is stronger than pretending to have experience and failing a follow-up question.

## The three knowledge levels

Every topic in `19-COMPLETE-TOPIC-RADAR.md` has a target depth:

| Level | What you must be able to do tomorrow |
|---|---|
| **A — Explain and apply** | Define it, say when to use it, give an example, and name a risk/trade-off. |
| **B — Explain and recognize** | Define it accurately, place it in the architecture, and say when a specialist/extra study is needed. |
| **C — Awareness** | Recognize the term and give a safe one-sentence description without inventing details. |

High-frequency topics such as app types, Power Fx, forms, delegation, Dataverse, security, flows, and solutions are Level A. PCF, plug-ins, virtual tables, gateways, offline, licensing, and AI features are generally Level B/C for a junior interview unless the job description emphasizes them.

## Non-negotiable rules for tonight

1. **Speak aloud.** Silent reading creates false confidence.
2. **Build one tiny app.** Even a two-screen app gives formulas meaning.
3. **Never memorize syntax without the property it belongs to.** Say `Gallery.Items = Filter(...)`, not only `Filter(...)`.
4. **Repeat the six critical boundaries:** UI is not security; collection is not database; app sharing is not data sharing; a working small-data query may still fail delegation; save is not publish; unmanaged development is not production deployment.
5. **Do not drown in rare features.** Learn what they are and when they matter, then return to Level A topics.
6. **Sleep.** A clear explanation tomorrow is worth more than three extra hours of exhausted passive reading.

## Choose your route

### If you have only 3 hours

| Time | Work | Output |
|---:|---|---|
| 0:00-0:25 | Read Topic Radar sections 1-3 | Explain Power Platform, canvas, model-driven, Dataverse, connector, environment, solution. |
| 0:25-1:05 | Read `02`, skim `03`, use Power Fx Crash Sheet sections 1-5 | Explain Studio, screen/control/property, gallery, form, `Navigate`, variables. |
| 1:05-1:35 | Power Fx CRUD + delegation | Explain `Filter`, `LookUp`, `Patch`, `SubmitForm`, `IfError`, delegation. |
| 1:35-2:00 | Dataverse + model-driven | Explain tables, relationships, choices/lookups, ownership, forms/views. |
| 2:00-2:25 | Flow + security | Explain app/data triggers, server validation, roles, UI versus authorization. |
| 2:25-2:40 | ALM + performance | Explain solutions, managed/unmanaged, environment variables, Monitor. |
| 2:40-3:00 | Say project pitch and scenarios aloud | Deliver a two-minute architecture plus three scenario answers. |

Do not attempt all 150 questions. Answer `17` questions 1-20, 56, 66-71, 86-93, 111-135, and 136-150 selectively.

### If you have 6 hours — recommended minimum

| Time | Work | Exact focus |
|---:|---|---|
| 0:00-0:35 | Platform map | `01` plus Radar sections 1-2. |
| 0:35-1:15 | Studio and first app | `02`; build 2 screens, 1 gallery, 2 buttons. |
| 1:15-2:05 | Power Fx | Crash Sheet; type formulas rather than copy/paste. |
| 2:05-2:35 | Forms/CRUD | `08` mental model; know `NewForm`, `EditForm`, `SubmitForm`, `OnSuccess`, `Patch`. |
| 2:35-3:05 | Data/delegation | `05` comparison table and `09` core sections. |
| 3:05-3:45 | Dataverse/model-driven | `10` schema + `11` views/forms/navigation. |
| 3:45-4:20 | Automation/security | `12` trigger choice + `13` privilege model. |
| 4:20-4:50 | Performance/ALM | `14` diagnosis + `16` managed deployment. |
| 4:50-5:15 | Rare-topic sweep | Radar sections 8-12: offline, gateway, PCF, AI, Power Pages, licensing. |
| 5:15-6:00 | Mock interview | Project pitch, 10 rapid questions, 5 scenarios, review weak answers. |

### If you have 12 hours

Use the 6-hour plan once, take a break, then:

| Time | Work |
|---:|---|
| 6:00-7:15 | Actually build the Lesson 03 prototype and explain each property. |
| 7:15-8:00 | Rebuild gallery search/filter and test formula types. |
| 8:00-9:00 | Draw Dataverse schema, relationship cardinality, and security roles. |
| 9:00-9:45 | Draw approval flow with idempotency, errors, and authoritative data retrieval. |
| 9:45-10:30 | Practice 25 randomized questions from `17`. |
| 10:30-11:15 | Practice every scenario in `21` using the answer framework. |
| 11:15-11:40 | Repeat the Topic Radar; mark every topic green/yellow/red. |
| 11:40-12:00 | Final pitch, questions for interviewer, then stop. |

## The one app you should build tonight

Build only this; do not chase visual perfection.

### Screen 1 — `scrRequestList`

- Text input `txtSearch`.
- Gallery `galRequests`.
- Button `btnNew`.
- Gallery shows Title, Status, Amount.

Temporary `App.OnStart`:

```powerfx
ClearCollect(
    colRequests,
    {Id: 1, Title: "Laptop", Status: "Submitted", Amount: 900},
    {Id: 2, Title: "Training", Status: "Approved", Amount: 300}
)
```

Gallery `Items`:

```powerfx
Filter(
    colRequests,
    IsBlank(txtSearch.Value) || StartsWith(Title, txtSearch.Value)
)
```

If using a classic input, replace `.Value` with `.Text`.

New button `OnSelect`:

```powerfx
Navigate(scrRequestEdit, ScreenTransition.Cover)
```

### Screen 2 — `scrRequestEdit`

- Text input `txtTitle`.
- Text/number input `txtAmount`.
- Button `btnSave`.

Save `OnSelect`:

```powerfx
If(
    IsBlank(Trim(txtTitle.Value)),
    Notify("Title is required.", NotificationType.Error),
    !IsNumeric(txtAmount.Value),
    Notify("Amount must be numeric.", NotificationType.Error),
    Collect(
        colRequests,
        {
            Id: Max(colRequests, Id) + 1,
            Title: Trim(txtTitle.Value),
            Status: "Draft",
            Amount: Value(txtAmount.Value)
        }
    );
    Notify("Saved.", NotificationType.Success);
    Back()
)
```

Then explain the production differences:

- Collection becomes SharePoint/Dataverse/SQL.
- `Max()+1` becomes a Dataverse GUID/autonumber.
- Large-source query must be delegable.
- `Patch` or `SubmitForm` persists data.
- Data-layer security controls record access.
- Errors, loading, concurrency, and ALM are added.

This comparison is a strong junior-interview answer because it shows you understand both the prototype and professional design.

## The ten answers you must be able to give

### 1. What is Power Apps?

> Power Apps is a low-code business application platform. Canvas apps give detailed UI control and connect to many sources; model-driven apps generate a data-focused experience from Dataverse. Power Apps normally works with Dataverse or connectors for data and Power Automate for workflow.

### 2. Canvas or model-driven?

> I choose canvas for a tailored task/device experience and model-driven for relationship-heavy Dataverse processes using forms and views. One solution can use both over the same Dataverse model.

### 3. What is Power Fx?

> It is Power Apps' strongly typed, declarative formula language. A control property contains a formula, such as a gallery's `Items = Filter(...)`; behavior properties such as `OnSelect` execute actions.

### 4. `Patch` or `SubmitForm`?

> `SubmitForm` is ideal for a form-driven create/edit experience with data cards, validation, `OnSuccess`, and `OnFailure`. `Patch` is better for targeted or custom writes, but I then own validation and error handling.

### 5. What is delegation?

> Delegation means Power Apps sends the query to the data source to evaluate over all rows. A nondelegable formula may inspect only the client row-limit subset and silently miss data; raising the limit is not a real fix.

### 6. Why Dataverse?

> Dataverse provides typed relational business data, choices and lookups, rich role/row security, auditing, APIs, business logic, native model-driven apps, and solution-based ALM.

### 7. How is security implemented?

> Authentication identifies the user; authorization must be enforced in Dataverse or the source through roles, privileges, ownership, teams, and access depth. `Visible=false` and gallery filters are only UI personalization.

### 8. When use Power Automate?

> For workflow, approvals, schedules, and integration. I use an app trigger for an immediate user action, but prefer a Dataverse change trigger when the process must run regardless of the client. Sensitive values are re-read from the authoritative source.

### 9. How do you diagnose a slow app?

> Reproduce with the affected user, inspect App Checker and delegation, then use Live Monitor to find slow or repeated connector calls, large payloads, N+1 lookups, and rendering cost before changing the design.

### 10. How do you deploy?

> I develop components in an unmanaged solution in development, use environment variables and connection references, validate, export/deploy a managed build to test and production, and use pipelines/source control where appropriate.

## How to answer something you only recognize

Use this wording:

> I have not implemented that feature yet, so I would not claim production experience. My understanding is that **[definition]**. I would consider it when **[scenario]**, and before choosing it I would validate **[security/licensing/support/scale concern]**.

Example:

> I have not built a PCF control yet. My understanding is that PCF is the pro-code framework for reusable Power Apps controls. I would consider it when standard controls cannot meet an essential interaction, and I would validate accessibility, security, lifecycle, and support cost before choosing it.

This is honest, technically useful, and much safer than saying only “I do not know.”

## Red-flag sentences to avoid

- “Power Apps is a database.”
- “I fix delegation by setting the limit to 2,000.”
- “I secure it by hiding the button.”
- “Sharing the app gives access to everything.”
- “I always load the table into a collection.”
- “Model-driven apps can use any connector as their native data model.”
- “I edit the managed solution directly in production.”
- “The flow trusts the email and amount sent by the app.”
- “I use Excel for any size because Power Apps supports it.”
- “Copilot generates the solution so no review is needed.”

## Morning-of-interview routine — 35 minutes

1. Read the ten answers above — 8 minutes.
2. Scan the Topic Radar's bold terms — 7 minutes.
3. Write from memory: `Filter`, `LookUp`, `If`, `Set`, `UpdateContext`, `Patch`, `SubmitForm`, `IfError` — 7 minutes.
4. Deliver project pitch — 3 minutes.
5. Answer scenarios 1, 3, 5, 8, and 12 from `21` — 8 minutes.
6. Stop and breathe — 2 minutes.

## Readiness test

You are ready enough for a junior interview if you can, without notes:

- Draw canvas/model-driven/Dataverse/flow relationships.
- Explain gallery, form, screen, control, property, and connector.
- Write a simple `Filter` and a create `Patch` pattern.
- Explain delegation and UI-versus-data security.
- Describe one Dataverse 1:N relationship.
- Describe one approval flow and one error path.
- Explain managed/unmanaged solutions.
- Give the Employee Request Hub pitch in two minutes.

## Primary references

- [Power Apps maker overview](https://learn.microsoft.com/power-apps/maker/)
- [Power Fx overview](https://learn.microsoft.com/power-platform/power-fx/overview)
- [PL-200 functional consultant scope](https://learn.microsoft.com/credentials/certifications/exams/pl-200/)

