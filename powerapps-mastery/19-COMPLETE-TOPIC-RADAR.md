# 19 — Complete Power Apps Topic Radar

## Purpose

This file prevents the “I have never heard that term” problem. It covers the broad Power Apps surface at interview depth.

Legend:

- **A:** explain and apply tomorrow.
- **B:** explain accurately and recognize the design decision.
- **C:** awareness; do not pretend implementation experience.
- **High/Medium/Low:** rough frequency in general junior Power Apps interviews. The job description can change the priority completely.

## 1. Platform and product map

| Topic | Level | Likelihood | What you should say |
|---|---:|---:|---|
| Power Platform | A | High | Connected low-code products: Power Apps for UI, Automate for workflow, Dataverse for business data, BI for analytics, Copilot Studio for agents, Pages for external sites. |
| Canvas app | A | High | UI-first app with precise layout, Power Fx, and many data-source connectors. |
| Model-driven app | A | High | Dataverse-first application generated from tables, relationships, forms, views, and navigation. |
| Custom page | B | Medium | Canvas-style page hosted in a model-driven app for focused custom interaction. |
| Power Apps cards | C | Low | Lightweight interactive card experiences in supported hosts; check current product support before proposing them. |
| Power Pages | B | Medium | External business websites using Dataverse, web roles/table permissions, identity, and separate licensing. |
| Power Automate | A | High | Workflow, approvals, schedules, integration, and background processing. |
| Power BI integration | B | Medium | Analytics/report embedding and Dataverse reporting; operational charts are not a replacement for an analytics model. |
| Copilot Studio | C | Medium | Platform for agents/conversational experiences that can connect to business data/actions. |

## 2. Tenants, environments, makers, and lifecycle

| Topic | Level | Likelihood | Safe explanation |
|---|---:|---:|---|
| Tenant | B | Medium | Organization-level Microsoft cloud boundary containing environments, identity, policies, and licenses. |
| Environment | A | High | Container/boundary for apps, flows, connections, Dataverse, security, capacity, and policies. |
| Environment types | B | Medium | Production, sandbox, trial, developer, default, and special-purpose types have different lifecycle/use; choose based on risk and purpose. |
| Default environment | B | Medium | Shared tenant environment intended for broad productivity; not automatically the correct place for governed production solutions. |
| Developer environment | B | Medium | Personal development space for building/testing, not a production entitlement or multi-user production architecture. |
| Managed Environments | B | Medium | Premium governance capabilities for environments, with licensing implications for active users. |
| Maker portal | A | High | `make.powerapps.com` for apps, tables, flows, solutions, connections, and environment selection. |
| Admin center | B | Medium | Power Platform admin center for environments, analytics, capacity, policies, backups, settings, and governance. |
| Solution | A | High | ALM container for apps, flows, tables, roles, references, and dependencies. |
| Publisher/prefix | B | Medium | Defines schema prefix/ownership identity for solution components; choose deliberately before schema grows. |

## 3. Canvas Studio and user interface

| Topic | Level | Likelihood | Safe explanation |
|---|---:|---:|---|
| Screen | A | High | App page containing controls and behavior such as `OnVisible`. |
| Control | A | High | UI element whose properties are Power Fx formulas. |
| Property | A | High | Appearance, layout, data, or behavior formula such as `Items`, `Text`, `Visible`, or `OnSelect`. |
| Tree View | A | High | Control hierarchy and reliable naming/selection/order. |
| Formula bar | A | High | Where the selected property formula is authored. |
| Gallery | A | High | Repeats a template over a table of records. |
| Display/Edit form | A | High | Displays or edits one record through data cards and form modes. |
| Data card | A | Medium | Form child mapped to a source field with default, input, update, validation, and error properties. |
| Modern controls/themes | B | Medium | Newer Fluent-based controls and centralized themes; validate current feature support/properties. |
| Containers | A | High | Layout hierarchy controlling alignment, gap, sizing, wrapping, and responsiveness. |
| Responsive design | A | High | Disable scale-to-fit, use auto-layout and relative sizing/breakpoints, then test multiple widths and zoom. |
| Accessibility | A | Medium | Keyboard use, focus, accessible labels, contrast, semantic structure, and text—not color alone—for status/errors. |
| Components | B | Medium | Reusable in-app control groups with explicit inputs/outputs/events. |
| Component library | B | Medium | Versioned reusable components shared across apps and governed by an owner/team. |
| Media | C | Low | Images/audio/video add payload and accessibility obligations; optimize and provide alternative text. |

## 4. Power Fx language

| Topic | Level | Likelihood | Safe explanation |
|---|---:|---:|---|
| Declarative formulas | A | High | A property recalculates from dependencies, like a spreadsheet. |
| Behavior formulas | A | High | Event properties execute actions separated in sequence. |
| Types | A | High | Text, number, Boolean, date/time, blank, record, table, Choice/record, GUID, color, etc. |
| Record/table scope | A | High | `ThisItem`, `ThisRecord`, and `As` identify the current record. |
| `If`/`Switch` | A | High | Ordered conditions versus comparison of one expression to several cases. |
| `Filter`/`LookUp` | A | High | All matching records as a table versus first matching record/value. |
| `SortByColumns` | A | High | Server-capable sorting when connector/column/expression supports delegation. |
| `With` | A | Medium | Names intermediate values in one formula without global state. |
| `ForAll` | B | Medium | Iterates a table but is often inefficient/nondelegable for large server operations. |
| `Patch` | A | High | Creates/updates a record; source types such as lookup/Choice must be respected. |
| Form functions | A | High | `NewForm`, `EditForm`, `ViewForm`, `SubmitForm`, `ResetForm`; handle result in form events. |
| Error handling | A | High | `IfError`, form `OnFailure`, `Errors`, and `App.OnError`/`Trace` for appropriate layers. |
| Variables | A | High | Global with `Set`, screen context with `UpdateContext`, navigation context via `Navigate`. |
| Collections | A | High | In-memory tables for drafts/selections/small caches, not a universal database/delegation fix. |
| Named formulas | B | Medium | Dependency-driven app-level calculations/configuration that can reduce procedural startup state. |
| User/Param/Launch | B | Medium | Current-user metadata and launch/deep-link parameters; neither grants authorization. |
| SaveData/LoadData | C | Low | Device-local collection persistence in supported clients; security, limits, sync, and conflicts still require design. |

## 5. Data sources and integration

| Topic | Level | Likelihood | Safe explanation |
|---|---:|---:|---|
| Connector/connection | A | High | Connector defines service operations; connection is an authenticated instance. |
| Standard/premium/custom connector | B | High | Licensing/governance categories and bespoke API wrapper; classification must be checked currently. |
| SharePoint | A | High | Common list/document source; understand Choice/Person fields, permissions, thresholds, indexes, and delegation. |
| Excel | A | Medium | Named-table source suitable for learning/small scenarios, weak for large concurrent transactions. |
| SQL | B | Medium | Enterprise relational source via connector; delegation, gateway/network, credentials, views/stored procedures, and licensing matter. |
| Dataverse | A | High | Native Power Platform business data with rich security, relationships, APIs, events, and ALM. |
| On-premises gateway | B | Medium | Secure bridge from cloud services to supported on-premises data; identity, cluster/high availability, updates, and ownership matter. |
| VNet data gateway | C | Low | Managed gateway path for supported Azure data inside virtual networks. |
| Custom connector | B | Medium | OpenAPI-backed connector for an API with authentication, schemas, policies, throttling, DLP, and lifecycle. |
| Dataflow | B | Medium | Power Query-based ingestion/transformation into supported destinations such as Dataverse; good for repeatable data loading, not interactive app writes. |
| Virtual table | B | Medium | External data represented in Dataverse without ordinary data replication; provider capability, security, feature limits, and performance matter. |
| Dataverse Web API | C | Medium for developer roles | OData-based API for programmatic Dataverse operations under proper authentication/security. |

## 6. Delegation and performance

| Topic | Level | Likelihood | Safe explanation |
|---|---:|---:|---|
| Delegation | A | Very high | Server evaluates supported query over all rows; nondelegable local evaluation can be incomplete. |
| Row limit | A | High | Development safeguard/subset limit, commonly configurable within documented bounds; raising it is not a scale solution. |
| Delegation warning | A | High | Potential correctness problem, not merely performance advice. |
| `StartsWith` search | A | Medium | Common delegable prefix-search pattern on supported sources; verify connector rules. |
| Source indexes | B | Medium | Delegation still requires efficient source-side filtering/sorting; SharePoint and databases need suitable indexes/design. |
| N+1 calls | A | High | Per-gallery-row lookup pattern causing many requests; use relationships/query shaping/small reference cache. |
| App Checker | A | High | Detects formula, delegation, accessibility, and performance findings. |
| Live Monitor | A | High | Runtime event/call/error/timing inspection in authoring and supported published scenarios. |
| `Concurrent` | A | Medium | Runs independent calls together; not for dependencies and not a cure for excessive calls. |
| Delayed search | B | Medium | Debounces input or uses explicit search to reduce connector calls. |

## 7. Dataverse data modeling and logic

| Topic | Level | Likelihood | Safe explanation |
|---|---:|---:|---|
| Standard/custom tables | A | High | Microsoft-provided versus organization-defined Dataverse tables. |
| Activity table | B | Low | Special activity behavior for communications/tasks/timeline scenarios. |
| Virtual table | B | Medium | Dataverse metadata over external rows with provider/feature limitations. |
| Elastic table | C | Low | Dataverse table option designed for specific high-scale patterns; consistency/features differ, so validate fit. |
| Columns/data types | A | High | Choose typed fields, formats, required level, precision, time-zone behavior, and length deliberately. |
| Choice | A | High | Controlled option set; local or reusable global choice. |
| Lookup | A | High | Related-record reference creating N:1 relationship. |
| 1:N/N:N | A | High | Relationship cardinality; use junction table when relationship has attributes. |
| Primary GUID | A | Medium | System-generated unique row identifier. |
| Autonumber | A | Medium | Human-readable business reference, not a replacement for GUID. |
| Alternate key | B | Medium | Unique business/integration key supporting identity/upsert. |
| User/team ownership | A | High | Row owner enables owner/business-unit security depths. |
| Organization ownership | A | Medium | Organization-wide ownership suited to common/reference data. |
| Formula/calculated column | B | Medium | Server/platform-derived row value using supported expressions. |
| Rollup column | B | Medium | Periodic aggregate over related rows; not necessarily real-time. |
| Business rule | A | High | Configured validation/default/visibility/required logic at defined scope; critical invariants still need nonbypassable enforcement. |
| Business process flow | A | High | Stage-based user guidance for model-driven processes. |
| Auditing | B | Medium | Tracks row/field changes when enabled and subject to capacity/retention; not the same as business event history. |
| Duplicate detection | C | Low | Rules/jobs/UI behaviors for possible duplicates; unique keys/server design are stronger for strict uniqueness. |

## 8. Model-driven app configuration

| Topic | Level | Likelihood | Safe explanation |
|---|---:|---:|---|
| App designer/navigation | A | High | Adds table/custom pages and organizes task-focused navigation. |
| Main/quick create/quick view forms | A | High | Full record, compact creation, and embedded related read-only data. |
| Views | A | High | Columns, order, width, sort, and filter for record lists. |
| System/public/personal views | B | Medium | Platform-defined, maker-shared, and user-owned views. |
| Charts/dashboards | B | Medium | Operational visual summaries respecting view/data privileges; Power BI for deeper analytics. |
| Subgrid | A | Medium | Related-record list on a form. |
| Timeline | B | Low | Displays configured activity/note records for a row. |
| Command bar | B | Medium | Model-driven actions; can be customized with supported designer/Power Fx patterns. |
| Business process flow | A | High | Visible stages/steps; separate from background automation. |
| Custom page | B | Medium | Canvas flexibility inside model-driven shell. |
| Client scripting | C | Medium for developer roles | JavaScript/web resources for supported model-driven client extensions; prefer supported APIs and minimize brittle scripts. |

## 9. Automation and business logic

| Topic | Level | Likelihood | Safe explanation |
|---|---:|---:|---|
| Power Apps (V2) trigger | A | High | Canvas app invokes flow with typed inputs and can receive response. |
| Dataverse trigger | A | High | Process runs on row change regardless of client; narrow columns/conditions and avoid loops. |
| Scheduled flow | A | Medium | Recurrence-based reminders/reconciliation/batch process. |
| Approval | A | High | Start/wait or other approval pattern; store business history and handle timeout/retry/duplicates. |
| Trigger conditions | B | Medium | Prevent unnecessary runs before flow actions start. |
| Run-after scopes | A | Medium | Try/catch/finally-style failure, timeout, and cleanup handling. |
| Idempotency | A | High | Duplicate trigger/run produces no duplicate business effect. |
| Child flow | B | Medium | Reusable solution-aware flow operation with connection/licensing considerations. |
| Connection reference | A | High | Solution abstraction bound to an actual environment connection. |
| Low-code plug-in | C | Low/Medium | Dataverse server-side reusable business logic authored with low-code capabilities; check current support/limitations. |
| Plug-in/custom API | C | Medium for developer roles | Pro-code Dataverse server extensions for transactional logic/custom operations, requiring engineering and ALM. |
| Classic workflow | C | Low | Legacy Dataverse automation technology; know it may exist, but prefer current supported architecture for new work. |

## 10. Security, governance, and administration

| Topic | Level | Likelihood | Safe explanation |
|---|---:|---:|---|
| Authentication/authorization | A | High | Identity versus permitted operation/data scope. |
| Security roles | A | Very high | Additive table privileges plus access depths assigned to users/teams. |
| Privileges | A | High | Create, Read, Write, Delete, Append, Append To, Assign, Share. |
| Access depths | A | High | User/basic, business unit/local, parent-child/deep, organization/global. |
| Business units | B | Medium | Dataverse security/ownership hierarchy, not automatically the HR chart. |
| Owner teams | B | Medium | Teams that can own rows and have roles. |
| Access teams | B | Medium | Flexible row collaboration without owning rows in configured patterns. |
| Entra group teams | B | Medium | Directory-managed membership mapped into Dataverse team access. |
| Row sharing | B | Medium | Exception-based direct access; can become difficult at large predictable scale. |
| Column security | B | Medium | Security profiles restrict supported sensitive columns beyond row privilege. |
| DLP/data policies | A | High | Connector grouping/blocking to govern data movement; not row security. |
| Tenant isolation | C | Low | Admin control over cross-tenant connector communication in supported scenarios. |
| Capacity | B | Medium | Dataverse database/file/log and service capacity must be monitored and licensed. |
| Backups/copy/reset | C | Low/Medium | Admin environment lifecycle operations with restrictions, retention, and production risk. |
| CoE Starter Kit | C | Medium | Microsoft-provided starter solutions/components for adoption, inventory, governance, and nurturing—not automatic governance. |
| Managed Environments | B | Medium | Premium governance/operations features with active-user licensing requirements. |

## 11. ALM and professional delivery

| Topic | Level | Likelihood | Safe explanation |
|---|---:|---:|---|
| Unmanaged solution | A | High | Editable development source layer. |
| Managed solution | A | High | Controlled downstream deployment package. |
| Solution layers | B | High | Ordered managed/unmanaged customizations producing effective behavior. |
| Dependencies | A | Medium | Required related components must be included/available for import/runtime. |
| Environment variables | A | High | Environment-specific nonsecret configuration. |
| Connection references | A | High | Environment binding for connector connections. |
| Update/upgrade | B | Medium | Update changes/adds; upgrade can remove missing components and applies upgrade behavior. |
| Pipelines | B | High | Repeatable staged solution deployment with governance; built-in or external CI/CD tooling. |
| Source control | B | Medium | Store supported unpacked solution source, review changes, tag releases, never secrets. |
| Solution checker | B | Medium | Static analysis/recommendations for solution components. |
| Deployment settings | C | Medium | Supply environment variable values and connection bindings during automated deployment. |
| Data migration | B | Medium | Solutions mainly move metadata, so reference/transaction data needs separate secure migration/upsert plan. |
| Rollback | B | Medium | Previous app package alone cannot undo data/schema/process effects; define recovery before release. |

## 12. Advanced, mobile, AI, and adjacent topics

| Topic | Level | Likelihood | Safe explanation |
|---|---:|---:|---|
| Mobile Power Apps player | B | Medium | Runs supported canvas/model-driven apps with device capabilities and mobile constraints. |
| Mobile offline | B | Medium | Offline-first sync for supported Dataverse/mobile scenarios; configure profiles/data/relationships and handle conflict/limitations. |
| SaveData/LoadData offline | C | Low | Local collection persistence, not a complete enterprise synchronization/security strategy. |
| Responsive versus native device features | B | Medium | Responsive covers layout; camera/barcode/location/signature/device APIs have permissions and platform-specific behavior. |
| PCF | B | Medium | Pro-code reusable UI controls when standard controls cannot meet essential needs. |
| Code components security | C | Medium | Custom code introduces dependency, accessibility, XSS/data, performance, and support review. |
| AI Builder | B | Medium | Prebuilt/custom AI models and document/prediction capabilities integrated with Power Platform, subject to capacity/licensing/governance. |
| Copilot in Power Apps | B | Medium | Natural-language assistance/generation and user experiences; outputs require human security/data/performance review. |
| Plans in Power Apps | C | Low/Medium | AI-powered planning/generation of roles, requirements, data, apps, flows, pages, reports/agents; preview status/capability must be checked. |
| Preview features | A | Medium | Not automatically production-ready; evaluate support, region, licensing, data boundary, and change risk. |
| Licensing | B | High | Depends on app/data/connector/environment/automation/AI usage; verify current official guide and tenant entitlements. |
| Power Apps Developer Plan | C | Low | Individual development entitlement/environment, not a production user-license strategy. |

## Job-description routing

If the job description says:

- **“SharePoint + Canvas”** — prioritize Studio, Power Fx, forms, Person/Choice fields, delegation, indexes, permissions, flows.
- **“Dataverse / Dynamics”** — prioritize schema, security roles, business units/teams, model-driven forms/views, solutions, flows.
- **“Power Platform Developer”** — add PCF, JavaScript/client API, plug-ins, custom APIs, Web API, Azure integration, source control, CI/CD.
- **“Functional Consultant”** — add discovery, process mapping, data model, security, model-driven configuration, Power Pages awareness, UAT/training/ALM.
- **“Administrator / CoE”** — add environments, Managed Environments, DLP, capacity, analytics, tenant settings, CoE, support/ownership/licensing.

## The final blind-spot test

Cover the explanation column and say one sentence for every bold/row topic. Mark:

- **Green:** accurate definition + use case.
- **Yellow:** recognize but cannot explain trade-off.
- **Red:** unfamiliar.

Convert every Level A red/yellow to green. For B/C topics, one accurate sentence is enough tonight.

## Primary references

- [Power Apps maker overview](https://learn.microsoft.com/power-apps/maker/)
- [PL-200 exam/functional consultant scope](https://learn.microsoft.com/credentials/certifications/exams/pl-200/)
- [Mobile offline overview](https://learn.microsoft.com/power-apps/mobile/mobile-offline-works-overview)
- [On-premises gateway](https://learn.microsoft.com/power-apps/maker/canvas-apps/gateway-reference)
- [Copilot in Power Apps](https://learn.microsoft.com/power-apps/maker/canvas-apps/ai-overview)
- [Managed Environments overview](https://learn.microsoft.com/power-platform/admin/managed-environment-overview)

