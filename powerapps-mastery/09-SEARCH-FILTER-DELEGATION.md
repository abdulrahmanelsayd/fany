# 09 — Search, Filter, and Delegation

## Goal

Build search and filtering that stays correct when a data source grows from 50 to 500,000 records.

## What delegation means

Delegation is Power Apps sending an operation to the data source so the server performs it over the full dataset.

```text
Delegable:   App sends query -> server filters/sorts -> matching rows return
Nondelegable: App retrieves limited rows -> client applies operation -> incomplete risk
```

When a formula cannot be delegated, Power Apps evaluates it over only the configured nondelegable row limit (commonly 500 by default and configurable up to a documented maximum such as 2,000). Increasing that limit does not fix correctness for larger sources.

## The blue underline is a correctness warning

A delegation warning does not simply mean “the app might be slower.” It can mean the answer is wrong because qualifying records beyond the local limit are never evaluated.

Always inspect warnings on:

- Gallery `Items`.
- Counts and aggregates.
- Lookups used for authorization or uniqueness.
- Update/delete targeting formulas.

## Delegation depends on three things

1. The connector/data source.
2. The function/operator.
3. The column type and expression shape.

A function delegable for Dataverse may not be delegable for Excel or SharePoint. Read the connector-specific delegation list and test with realistic volume.

## Design a delegable gallery query

Assume Dataverse table `Requests` with text column `Title`, Choice `Status`, date `RequestedOn`, and normalized text `RequestedByEmail`.

Start simple:

```powerfx
Filter(Requests, RequestedByEmail = varUserEmail)
```

Add status:

```powerfx
Filter(
    Requests,
    RequestedByEmail = varUserEmail &&
    (IsBlank(cmbStatus.Selected) || Status = cmbStatus.Selected.Value)
)
```

The exact Choice expression depends on control and source types. Favor direct typed comparisons selected by autocomplete.

Add prefix search where supported:

```powerfx
Filter(
    Requests,
    RequestedByEmail = varUserEmail &&
    (IsBlank(txtSearch.Value) || StartsWith(Title, txtSearch.Value))
)
```

Sort:

```powerfx
SortByColumns(
    Filter(
        Requests,
        RequestedByEmail = varUserEmail &&
        (IsBlank(txtSearch.Value) || StartsWith(Title, txtSearch.Value))
    ),
    "RequestedOn",
    SortOrder.Descending
)
```

Verify every part against the current Dataverse delegation documentation. Do not assume that an expression remains delegable after wrapping a column in a transformation.

## Why contains search is harder

Users like substring search such as “keyboard” anywhere in a description. But `in`, `Search`, and transformations have connector-specific delegation behavior.

Options:

- Use `StartsWith` when acceptable and supported.
- Use a server-side search capability designed for the source.
- Add normalized/search columns maintained at the data layer.
- Use Dataverse search or an appropriate indexed service for richer search.
- Narrow by delegable criteria first, then perform a local transformation only on a guaranteed-small result.

Do not hide a delegation warning by loading a large source into a collection.

## Debounce user input

Searching the server on every keystroke can create excessive queries. Where supported, configure delayed output for a text input or use an explicit search action.

Concept:

```powerfx
txtSearch.DelayOutput = true
```

Then reference the search value in the gallery query. Property availability differs between control generations.

## Sort safely

Allow only known column names rather than creating arbitrary dynamic queries.

```powerfx
Switch(
    locSortColumn,
    "Title", SortByColumns(filteredRequests, "Title", locSortDirection),
    "Amount", SortByColumns(filteredRequests, "Amount", locSortDirection),
    SortByColumns(filteredRequests, "RequestedOn", locSortDirection)
)
```

Complex locally constructed table variables can break delegation. Often repeat or encapsulate the delegable source expression carefully rather than materializing it locally.

## Date ranges

```powerfx
Filter(
    Requests,
    RequestedOn >= dpFrom.SelectedDate &&
    RequestedOn < DateAdd(dpTo.SelectedDate, 1, TimeUnit.Days)
)
```

Using an exclusive upper bound is useful when the stored field contains time, because records late on the selected end date remain included.

## Common nondelegable traps

These depend on connector and version, so treat them as investigation triggers:

- Transforming the source column before comparison: `Lower(SourceColumn) = ...`.
- Complex `AddColumns`/`GroupBy` pipelines over large sources.
- `Distinct` or `ForAll` against a source.
- Row-by-row local membership tests.
- Counting large sets with unsupported aggregates.
- Dynamic column expressions.

Normalize data at write time or use server-supported columns/queries when needed.

## Data row limit test technique

During development, temporarily set the nondelegable data row limit to a very small value such as 1:

1. Open **Settings**.
2. Find the data row limit setting.
3. Set it to `1` in a safe test copy.
4. Test all galleries, lookups, and counts.

Correct delegable queries continue to reach matching server records. Nondelegable assumptions fail dramatically and become easy to detect. Restore the intended setting after the test.

## Indexes and source design

Delegation is necessary but not sufficient. Server query performance also depends on:

- Indexed/filterable columns.
- Selectivity of filters.
- Sort columns.
- Relationship/query structure.
- Source throttling and service limits.

For SharePoint, indexed columns and threshold-aware list design matter. For Dataverse/SQL, work with the platform's indexing and query guidance rather than guessing.

## Security warning

This formula is personalization, not authorization:

```powerfx
Filter(Requests, RequestedByEmail = User().Email)
```

If the user has read access to every source record, another app or API could expose them. Use Dataverse ownership/security roles, SharePoint permissions where appropriate, or server-side authorization.

## Guided lab

1. Create at least 25 sample records with varied titles, dates, users, and statuses.
2. Build filters for current user, status, and date range.
3. Add prefix title search.
4. Add ascending/descending sort.
5. Enable delayed input if available.
6. Reduce the nondelegable limit to 1.
7. Confirm that a matching record outside the first local row still appears.
8. Review every Studio warning.
9. Use Live Monitor in Lesson 14 to inspect requests.

## Interview questions

**What is delegation?**  
The ability to translate a Power Fx operation into a query performed by the data source over the complete dataset.

**How do you fix a delegation warning?**  
Identify the connector, function/operator, and column type causing it; rewrite using supported operations, redesign a normalized/indexed server column, use a server-side search/query capability, or first narrow to a provably small dataset. Raising the row limit is not a general fix.

**Why can a nondelegable formula be dangerous even when it looks correct?**  
Small development data fits inside the local limit. Production data may not, producing silently incomplete results.

**Is `ClearCollect(allRows, Source)` a delegation solution?**  
No. The collection load itself can be limited, consumes memory, becomes stale, and moves work to the client.

## Challenge

A gallery uses:

```powerfx
Filter(Requests, Lower(RequestedByEmail) = Lower(User().Email))
```

It warns for your connector. Propose a redesign.

### Answer

Store a normalized lowercase email column at write time or use an immutable identity/lookup column, then compare directly with a normalized constant. Ensure authorization is enforced through data-layer security rather than the filter.

## Primary references

- [Delegation overview](https://learn.microsoft.com/power-apps/maker/canvas-apps/delegation-overview)
- [Dataverse delegable functions and operations](https://learn.microsoft.com/power-apps/maker/canvas-apps/connections/connection-common-data-service#power-apps-delegable-functions-and-operations-for-dataverse)
- [SharePoint delegable functions and operations](https://learn.microsoft.com/power-apps/maker/canvas-apps/connections/connection-sharepoint-online#power-apps-delegable-functions-and-operations-for-sharepoint)

