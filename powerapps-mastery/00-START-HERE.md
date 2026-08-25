# Power Apps Mastery: Start Here

> A beginner-to-interview curriculum for Microsoft Power Apps

Last curriculum review: **August 25, 2026**

## Interview tomorrow? Do not start with the eight-week plan

If you currently know almost nothing and the interview is tomorrow, start here:

1. `18-TOMORROW-INTERVIEW-PLAN.md` — choose the 3-hour, 6-hour, or 12-hour route.
2. `19-COMPLETE-TOPIC-RADAR.md` — see every major topic at least once and learn the safe interview-level explanation.
3. `20-POWER-FX-CRASH-SHEET.md` — learn the formulas and patterns most likely to be tested.
4. `21-SCENARIO-ANSWER-PLAYBOOK.md` — practice answering design and troubleshooting questions professionally.
5. `17-INTERVIEW-MASTER-FILE.md` — use the numbered bank for active recall, not cover-to-cover reading.

The emergency route does **not** simplify the technical level. It changes the order: broad map first, high-frequency depth second, advanced-topic recognition third. That gives you something accurate to say even when the interviewer mentions a topic you have not implemented yet.

## What you will be able to do

By the end of this course, you should be able to:

- Decide whether a business problem needs a canvas app, a model-driven app, or neither.
- Build responsive canvas apps without relying on fragile pixel-by-pixel layouts.
- Connect to SharePoint, Excel, SQL, and Microsoft Dataverse and explain the trade-offs.
- Write, read, and debug practical Power Fx formulas.
- Build forms, galleries, search, filtering, validation, navigation, and role-aware experiences.
- Design a relational Dataverse model and secure it correctly.
- Trigger Power Automate flows from an app and handle results and failures.
- Diagnose delegation, performance, connector, sharing, and permission problems.
- Package a solution and move it through development, test, and production.
- Answer both knowledge and scenario-based Power Apps interview questions.

## How this course works

The course uses one continuous case study: **Employee Request Hub**. Employees submit requests, managers review them, and operations staff report on them.

You will build it in layers:

1. A small local prototype.
2. A canvas app connected to real data.
3. A Dataverse data model.
4. A model-driven back-office app.
5. Approval and notification automation.
6. Security roles and governance.
7. A deployable solution.

Each lesson contains:

- A plain-English mental model.
- Exact UI click paths.
- A guided lab.
- Formulas with explanations.
- Common mistakes.
- Interview questions.
- A short challenge and answer.

## Files and recommended order

| Order | File | Outcome |
|---:|---|---|
| 0 | `00-START-HERE.md` | Prepare your environment and study plan. |
| 1 | `01-POWER-PLATFORM-MAP.md` | Understand the platform and choose the right app type. |
| 2 | `02-POWER-APPS-UI-TOUR.md` | Become comfortable in the maker portal and Studio. |
| 3 | `03-FIRST-CANVAS-APP.md` | Build and publish a small working app. |
| 4 | `04-UI-UX-RESPONSIVE-DESIGN.md` | Build accessible, responsive interfaces. |
| 5 | `05-DATA-SOURCES-CONNECTORS.md` | Connect data and understand connector behavior. |
| 6 | `06-POWER-FX-FUNDAMENTALS.md` | Learn the Power Fx language. |
| 7 | `07-STATE-VARIABLES-COLLECTIONS.md` | Manage state deliberately. |
| 8 | `08-GALLERIES-FORMS-CRUD.md` | Build complete create, read, update, delete experiences. |
| 9 | `09-SEARCH-FILTER-DELEGATION.md` | Query large data sources safely. |
| 10 | `10-DATAVERSE-DEEP-DIVE.md` | Design business data correctly. |
| 11 | `11-MODEL-DRIVEN-APPS.md` | Build a back-office app. |
| 12 | `12-POWER-AUTOMATE-INTEGRATION.md` | Add approvals and notifications. |
| 13 | `13-SECURITY-SHARING-GOVERNANCE.md` | Secure and govern the solution. |
| 14 | `14-PERFORMANCE-DEBUGGING-TESTING.md` | Diagnose and optimize apps. |
| 15 | `15-COMPONENTS-ADVANCED-PATTERNS.md` | Create maintainable, reusable app architecture. |
| 16 | `16-SOLUTIONS-ALM-DEPLOYMENT.md` | Move the solution safely between environments. |
| 17 | `17-INTERVIEW-MASTER-FILE.md` | Prepare for technical and scenario interviews. |
| 18 | `18-TOMORROW-INTERVIEW-PLAN.md` | Execute a time-boxed plan for an interview within 24 hours. |
| 19 | `19-COMPLETE-TOPIC-RADAR.md` | Cover the full Power Apps topic surface without blind spots. |
| 20 | `20-POWER-FX-CRASH-SHEET.md` | Memorize and understand essential Power Fx patterns. |
| 21 | `21-SCENARIO-ANSWER-PLAYBOOK.md` | Structure strong scenario, troubleshooting, and architecture answers. |
| 22 | `22-IT-SERVICE-MANAGEMENT-PROJECT-WALKTHROUGH.md` | Build and explain the exact Power Apps + SharePoint + Power Automate CV project. |

## Prerequisites

You need:

- A work or school Microsoft account with Power Apps access.
- Permission to create apps in at least one environment.
- For the Dataverse lessons, an environment containing a Dataverse database and the Environment Maker role.
- For Power Automate, permission to create cloud flows.

Licensing changes regularly. Treat the tenant's license assignment and the current Microsoft licensing guide as the source of truth. Do not promise a client that a connector or feature is included before checking its current license classification.

## Prepare a safe learning environment

Prefer a developer or sandbox environment instead of experimenting in production.

1. Open `https://make.powerapps.com`.
2. Sign in.
3. Check the environment selector in the upper-right corner.
4. Select a developer or sandbox environment provided by your administrator.
5. Confirm that **Dataverse > Tables** is available in the left navigation.

If you cannot see **Tables**, you may be in an environment without Dataverse or lack permission. You can still complete early canvas lessons using the supplied CSV data, but ask your administrator for a safe Dataverse environment before Lesson 10.

## Import the starter data

The file `sample-data/requests.csv` contains practice records.

For a quick Excel exercise:

1. Open the CSV in Excel.
2. Select the data and choose **Insert > Table**.
3. Confirm **My table has headers**.
4. On **Table Design**, rename the table to `RequestsTable`.
5. Save the workbook to OneDrive for Business.

Important: Power Apps connects to an Excel **table**, not an arbitrary cell range. Excel is acceptable for learning and small personal scenarios, but it is not a substitute for a transactional multi-user database.

## The professional build habit

For every feature, use this sequence:

1. State the user story.
2. Identify the data and security boundary.
3. Choose the app type and data source.
4. Build the smallest working path.
5. Add validation and error handling.
6. Test with realistic accounts and data volume.
7. Document assumptions and deployment dependencies.

## Eight-week study plan

| Week | Lessons | Deliverable |
|---:|---|---|
| 1 | 00-03 | A published local-data canvas app. |
| 2 | 04-06 | Responsive UI and basic formulas. |
| 3 | 07-09 | Data-backed CRUD app with delegable search. |
| 4 | 10 | Dataverse schema and security design. |
| 5 | 11-12 | Model-driven app plus approval flow. |
| 6 | 13-14 | Secure, tested, optimized app. |
| 7 | 15-16 | Reusable components and deployment package. |
| 8 | 17 | Mock interviews and final project explanation. |

Recommended daily rhythm: 25 minutes reading, 50 minutes building, 15 minutes explaining the concept aloud without notes.

## How to know you really understand a lesson

Do not mark a lesson complete merely because the supplied formula works. You should be able to:

- Rebuild the feature in a blank app.
- Explain why the formula belongs on that property.
- Predict what happens when the data source has 100,000 rows.
- Explain which user identity and permissions are used.
- Describe at least one alternative and its trade-off.
- Diagnose one deliberately introduced error.

## Conventions used in formulas

| Prefix | Meaning | Example |
|---|---|---|
| `scr` | Screen | `scrRequestList` |
| `con` | Container | `conPage` |
| `gal` | Gallery | `galRequests` |
| `frm` | Form | `frmRequest` |
| `txt` | Text input | `txtSearch` |
| `lbl` | Label | `lblTitle` |
| `btn` | Button | `btnSave` |
| `ico` | Icon | `icoBack` |
| `cmb` | Combo box | `cmbStatus` |
| `var` | Variable | `varSelectedRequest` |
| `col` | Collection | `colDraftLines` |

Control properties can differ between modern and classic controls. For example, a modern text input commonly exposes `Value`, while a classic text input commonly exposes `Text`. Read the property list in your Studio version and adapt the sample deliberately.

## Primary references

- [Power Apps maker documentation](https://learn.microsoft.com/power-apps/maker/)
- [Power Fx overview](https://learn.microsoft.com/power-platform/power-fx/overview)
- [Power Platform learning paths](https://learn.microsoft.com/training/powerplatform/)
- [Power Platform licensing](https://www.microsoft.com/licensing/docs/view/Power-Platform)

## First checkpoint

Before opening Lesson 01, answer these aloud:

1. Which environment will you use?
2. Can you create a canvas app there?
3. Can you see Dataverse tables?
4. Where will you store the exercise workbook?
5. What will you show as your final portfolio project?
