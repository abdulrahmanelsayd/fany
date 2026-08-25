# 16 — Solutions, ALM, and Deployment

## Goal

Package the Employee Request Hub, move it safely from development to test and production, and explain solution layers, configuration, pipelines, and source control.

## ALM mental model

Application lifecycle management covers:

```text
Plan -> Develop -> Validate -> Build -> Test -> Deploy
  ^                                               |
  |------------ Operate, monitor, learn ----------|
```

ALM is not just exporting a ZIP file. It includes requirements, versioning, testing, security, deployment, monitoring, rollback/recovery, and ownership.

## Environment strategy

Minimum professional pattern:

| Environment | Purpose | Solution type |
|---|---|---|
| Development | Makers create/change components | Unmanaged |
| Test/UAT | Integration/security/business validation | Managed deployment |
| Production | Real users/data | Managed deployment |

Larger programs may add build, integration, training, hotfix, and regional environments. Environment count should follow isolation, risk, team, capacity, and release needs.

## Managed versus unmanaged

| Unmanaged | Managed |
|---|---|
| Source development/customization layer | Packaged downstream deployment |
| Components can be directly edited | Controlled through managed properties/layers |
| Export as managed or unmanaged | Update/upgrade through solution packages |
| Keep in development | Use in test/production in standard practice |

Do not import the same solution as unmanaged into production merely because it is easier to edit there. Fix in development and deploy.

## Solution contents checklist

For Employee Request Hub include:

- Canvas app.
- Model-driven app.
- Request, Department, Request Comment, Approval Event tables.
- Required columns, relationships, choices, forms, views, charts.
- Cloud flows.
- Connection references.
- Environment variables.
- Custom security roles.
- Component library/custom connector if used.
- Business process flow and business rules if used.

Use **Add required objects** carefully and review what it includes. Missing dependencies cause import/runtime failures; including unrelated components creates coupling.

## Versioning

Use a consistent four-part version, for example:

```text
Major.Minor.Build.Revision
1.3.0.0
```

Example policy:

- Major: breaking schema/process release.
- Minor: backward-compatible feature release.
- Build/revision: fixes and deployment iterations.

Increment before export and map the version to a release record/source commit.

## Environment variables

Use for environment-specific configuration:

- Operations mailbox.
- Approval timeout.
- API base URL.
- SharePoint site/list identifiers where appropriate.
- Feature/configuration values.

Create inside solution:

1. **New > More > Environment variable** or current command.
2. Set display/schema name and type.
3. Set a default only if safe and universal.
4. Provide current value in each environment/deployment.

Do not store secrets in ordinary environment variables. Use supported secret/Azure Key Vault-backed patterns where applicable.

## Connection references

A connection reference allows solution-aware components to bind to an environment's actual connection.

Development:

```text
Flow -> SharePoint connection reference -> Developer connection
```

Production:

```text
Same flow -> Same reference -> Production service connection
```

Use organizational/service ownership with least privilege for critical automation where supported. Document credential rotation and owner continuity.

## Pre-export checklist

1. Save and publish all customizations.
2. Run solution checker where available.
3. Resolve app checker/delegation/accessibility issues.
4. Confirm flows are solution-aware and use connection references.
5. Remove hard-coded environment URLs, IDs, and emails.
6. Confirm environment variables and defaults.
7. Confirm dependencies and managed properties.
8. Increment version.
9. Run automated/manual test suite in development.
10. Record release notes and data-migration steps.

## Manual export/import lab

### Export from development

1. Open **Solutions > Employee Request Hub**.
2. Select **Publish all customizations** when appropriate.
3. Select **Export**.
4. Run dependency/problem checks offered by the wizard.
5. Choose **Managed** for test.
6. Export/download the package.

### Import to test

1. Switch to the test environment.
2. Select **Solutions > Import solution**.
3. Upload/select the managed package.
4. Review package details and dependencies.
5. Bind connection references.
6. Supply environment variable values.
7. Import.
8. Review import history/result.
9. Turn on flows only after configuration/security review.
10. Assign roles/app access to test personas.
11. Execute smoke, integration, security, and UAT tests.

UI details vary with release and pipeline use, but the dependency/configuration/security sequence remains.

## Update versus upgrade

- **Update:** adds/changes components but does not generally remove components missing from the new version.
- **Upgrade:** can remove components no longer present and applies upgrade behavior.
- **Stage for upgrade:** supports controlled staging before applying upgrade.

Use upgrade for intentional component removal and test data/dependency consequences. Never delete a column/table without retention, migration, integration, and rollback analysis.

## Solution layers

Multiple solutions can customize the same component. Dataverse evaluates layers to determine effective behavior.

When production differs from development:

1. Inspect solution layers.
2. Identify active/unmanaged layers.
3. Check which managed solution/version introduced each layer.
4. Avoid direct production customizations that create an unmanaged active layer.
5. Resolve through planned source/deployment, not random edits.

## Pipelines

Power Platform pipelines can deploy solutions between configured environments with governance and deployment stages. Azure DevOps Build Tools and GitHub Actions support more customized CI/CD.

A pipeline should automate repeatable work such as:

- Export/unpack/build/check.
- Versioning.
- Static/solution analysis.
- Managed artifact creation.
- Deployment and configuration.
- Approval gates.
- Test invocation and evidence.

Automation does not remove the need for environment strategy, tests, change approval, or recovery planning.

## Source control

Store unpacked solution source in version control using supported Power Platform CLI/tooling. Principles:

- Repository is the collaboration/audit source for solution artifacts.
- Use small reviewed changes.
- Avoid two makers editing the same complex artifact simultaneously.
- Build a managed deployable artifact from controlled source.
- Tag releases.
- Never commit secrets or downloaded connection credentials.

Canvas app source representation/tooling evolves. Use the supported current CLI/source format rather than manipulating internal files casually.

## Team development

Complex artifacts such as a canvas app, form, or flow can conflict when edited simultaneously. Coordinate ownership:

- Split work by components/screens/apps where possible.
- Communicate check-out/edit windows.
- Commit/export frequently.
- Review solution diffs.
- Integrate and test in a shared environment.

More environments can reduce editing collisions but increase merge/dependency discipline requirements.

## Data is not ordinary solution metadata

Solutions primarily move configuration/components, not transactional business rows. Plan separately for:

- Reference/configuration data migration.
- Test data generation.
- Dataflows/import tools/APIs.
- Alternate keys and upsert.
- Sequence/order and relationship resolution.
- Sensitive data masking.

Do not copy production personal data into development without authorization and protection.

## Rollback and recovery

Before production deployment define:

- What artifact/version can be redeployed?
- Can the previous managed solution be restored safely?
- Are schema/data changes backward compatible?
- Is environment backup/restore appropriate and within policy?
- How are flows disabled and queued events handled?
- What is the communication/escalation path?

An app rollback cannot automatically undo data changes already made by a flow or schema migration.

## Release runbook

```text
Release: ERH 1.3.0.0
Owner:
Approved change:
Artifact checksum/location:
Dependencies:
Environment variables:
Connection bindings:
Pre-deployment backup/checks:
Import/deploy steps:
Flow activation order:
Security role assignment:
Smoke tests:
Monitoring window:
Rollback decision and steps:
Sign-off:
```

## Common mistakes

- Development in the default/production environment.
- Unmanaged solution in production.
- Hard-coded environment configuration.
- Missing connection references.
- Exporting without publishing customizations.
- Assuming solutions contain transactional data.
- Direct production edits creating unmanaged layers.
- No rollback for destructive schema/flow changes.

## Interview questions

**Why use solutions?**  
They package Power Platform components, dependencies, configuration references, versions, and managed deployment behavior for ALM.

**Managed versus unmanaged?**  
Unmanaged is the editable source layer used in development; managed is the controlled package normally deployed downstream.

**Environment variable versus connection reference?**  
An environment variable supplies environment-specific configuration values. A connection reference binds a component to an authenticated connector connection in that environment.

**Why might a production component differ after import?**  
Solution layering, an unmanaged active layer, missing dependencies/configuration, app/flow publication state, or an older/newer solution layer can change effective behavior.

## Challenge

A test flow sends email to the production operations mailbox after import. What design error likely occurred?

### Answer

The mailbox was hard-coded or the environment variable current value was bound incorrectly. Move it to an environment variable, validate deployment settings, and add a test assertion before flow activation.

## Primary references

- [ALM with Power Platform](https://learn.microsoft.com/power-platform/alm/)
- [Solution concepts](https://learn.microsoft.com/power-platform/alm/solution-concepts-alm)
- [Environment variables](https://learn.microsoft.com/power-apps/maker/data-platform/environmentvariables)
- [Pipelines in Power Platform](https://learn.microsoft.com/power-platform/alm/pipelines)
- [Power Platform CLI](https://learn.microsoft.com/power-platform/developer/cli/introduction)

