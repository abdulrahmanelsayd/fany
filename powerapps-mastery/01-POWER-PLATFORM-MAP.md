# 01 — The Power Platform Map

## Goal

Understand where Power Apps fits, choose between canvas and model-driven apps, and avoid using an app where a simpler tool would work.

## The one-minute mental model

Microsoft Power Platform is a set of connected low-code products:

| Product | Main job | Typical example |
|---|---|---|
| Power Apps | User interface for business work | Employees submit and track requests. |
| Power Automate | Workflow and integration | Route a request for approval and send reminders. |
| Dataverse | Managed business data platform | Store requests, comments, departments, and security. |
| Power BI | Analytics | Show request volume and completion time. |
| Copilot Studio | Conversational agents | Let an employee ask about request status. |
| Power Pages | External-facing business sites | Allow suppliers to submit information. |

Power Apps is not automatically the database, workflow engine, and reporting platform. A good solution assigns each responsibility to the right layer.

## Canvas apps versus model-driven apps

| Question | Canvas app | Model-driven app |
|---|---|---|
| Where do you start? | The user experience | The Dataverse data model |
| Layout control | High | Mostly generated/configured |
| Data sources | Dataverse plus many connectors | Dataverse only |
| Responsive by default | Must be designed | Largely built in |
| Best for | Task-focused, branded, device-specific UI | Data-dense process and back-office work |
| Navigation | You build it | Generated from app navigation and relationships |
| Formula use | Extensive Power Fx | More configuration; formulas/custom pages when needed |
| Build speed | Depends on UI complexity | Fast after the data model is sound |

The Employee Request Hub uses both:

- A **canvas app** for employees because the submission experience should be simple and branded.
- A **model-driven app** for operations staff because they need views, forms, charts, and related records.

This is not duplication. It is two experiences over a shared business model.

## Choose with a decision sequence

Ask in this order:

1. **Who is the user and what decision are they making?**
2. **Where is the authoritative data?**
3. **Is a standard Microsoft list/form already enough?**
4. **Does the UI need precise layout or device behavior?** Choose canvas.
5. **Is the experience primarily forms, views, relationships, and process?** Choose model-driven.
6. **Are external anonymous or licensed external users involved?** Evaluate Power Pages.
7. **Is the requirement primarily an approval or system-to-system process?** Start with Power Automate, not an app.

## Architecture of the course project

```text
Employee
   |
   v
Canvas app ---------> Dataverse: Request
   |                       |  \---- Request Comment
   |                       \------- Department
   v
Power Automate approval/notification
                           ^
                           |
Operations team ---> Model-driven app
                           |
                           v
                       Power BI (optional)
```

## Environments are boundaries

An environment is a container for apps, flows, connections, Dataverse, and governance policies. Common strategy:

- **Development:** makers change unmanaged solutions.
- **Test/UAT:** testers validate a managed build and configuration.
- **Production:** real users and real business data.

Do not treat the default environment as the universal production environment. Organizations use environments to separate teams, data, risk, and application lifecycle stages.

## Solutions are delivery containers

A solution groups components such as apps, tables, flows, environment variables, connection references, and security roles. Development typically happens in an **unmanaged** solution; downstream deployment normally uses a **managed** solution.

You will implement this in Lesson 16. For now, create assets inside a solution whenever the environment and lesson allow it. This builds correct habits early.

## Connectors and connection identity

A connector is a wrapper around a service or API. A connection is an authenticated instance of that connector.

Example:

- Connector: SharePoint
- Connection: `alex@contoso.com` authenticated to SharePoint
- Data source: the `Requests` list on a particular site

Interview trap: sharing an app does not automatically grant permission to its underlying data. Users generally need both app access and data access.

## Copilot and generated plans

AI-assisted creation can produce a starting application or data model. Treat generated output as a draft. A maker remains responsible for:

- Data types and relationships.
- Security boundaries.
- Delegation and performance.
- Validation and error behavior.
- Licensing and connector classification.
- Testing and maintainability.

In an interview, do not describe Copilot as a replacement for solution design. Describe it as an accelerator that still requires review.

## Guided exercise: classify requirements

For each requirement, choose the primary product or app style.

1. Warehouse workers scan items on phones with a custom large-button interface.
2. Case managers review related customers, cases, activities, and dashboards.
3. A manager receives an approval and the requester receives an email.
4. Executives need trend analysis over three years.
5. External vendors submit onboarding documents.

### Suggested answer

1. Canvas app.
2. Model-driven app over Dataverse.
3. Power Automate, possibly initiated by an app or data change.
4. Power BI.
5. Often Power Pages, after identity, licensing, and security analysis.

## Common mistakes

- Building a canvas app only because it looks more customizable, then recreating model-driven views, forms, security, and navigation manually.
- Using Excel as a multi-user transactional database.
- Putting all business rules in screen controls, making them easy to bypass from another app.
- Creating directly in production.
- Sharing the app but forgetting the data source, flow, or custom connector permissions.
- Choosing a premium connector without validating licenses.

## Interview questions

**Why would you choose a model-driven app?**  
When the solution is centered on Dataverse records, relationships, processes, forms, views, and consistent responsive behavior. It reduces UI construction and gives rich data-driven features.

**Can a canvas app use Dataverse?**  
Yes. Canvas apps can connect directly to Dataverse and can also use many other data sources.

**Can a model-driven app use SharePoint as its primary native data source?**  
No. Model-driven apps are built on Dataverse. Integration can still bring external data into the broader solution.

**What is the difference between a connector and a connection?**  
The connector defines how Power Platform communicates with a service; the connection is an authenticated instance used by an app or flow.

## Challenge

Write a five-sentence architecture recommendation for the Employee Request Hub. Include app type, data platform, automation, security, and deployment.

## Primary references

- [Overview of creating apps in Power Apps](https://learn.microsoft.com/power-apps/maker/)
- [What are model-driven apps?](https://learn.microsoft.com/power-apps/maker/model-driven-apps/model-driven-app-overview)
- [What is Dataverse?](https://learn.microsoft.com/power-apps/maker/data-platform/data-platform-intro)
- [Power Platform environments overview](https://learn.microsoft.com/power-platform/admin/environments-overview)

