# KQL Cheatsheet — Practical Notes

Notes for Kusto Query Language as actually used in Microsoft Sentinel detections. Not an exhaustive reference — just the operators and patterns I keep reaching for.

## The pipeline

KQL queries flow left to right through `|`. Each operator transforms the previous result.

```kql
SigninLogs
| where TimeGenerated >= ago(1h)
| where ResultType != 0
| summarize Failures = count() by UserPrincipalName
| sort by Failures desc
| take 10
```

Read as: "from SigninLogs, keep the last hour, keep failures, count failures per user, sort, top 10."

## Time

```kql
| where TimeGenerated >= ago(15m)              // last 15 minutes
| where TimeGenerated between (ago(1d) .. now()) // last 24 hours
| where TimeGenerated >= startofday(now())     // since midnight UTC
```

`ago()` always relative to now. `between` is inclusive both ends. `startofday`, `startofweek`, `startofmonth` for calendar boundaries.

## Filter — `where`

```kql
| where EventID == 4625
| where Account !contains "$"
| where Computer startswith "DC-"
| where IpAddress in ("10.0.0.1", "10.0.0.2")
| where IpAddress !in (KnownGoodIPs)
```

Prefer `==` over `contains` when possible — much faster on indexed columns.

## Project / extend — pick and compute fields

```kql
| project TimeGenerated, UserPrincipalName, IPAddress    // keep only these columns
| extend Country = tostring(LocationDetails.countryOrRegion)  // add a computed field
| project-away ColumnToHide
| project-rename NewName = OldName
```

## Summarize — group and aggregate

```kql
| summarize 
    EventCount = count(),
    UniqueUsers = dcount(Account),
    FirstSeen = min(TimeGenerated),
    Accounts = make_set(Account, 100)   // collects distinct values (max 100)
    by IpAddress
```

Useful aggregates: `count()`, `dcount()` (distinct count), `min`, `max`, `avg`, `sum`, `make_set()`, `make_list()`, `any()` (any one value), `arg_max(timestamp, *)` (latest row per group).

## Joins

```kql
LeftTable
| join kind=inner RightTable on Key
```

`kind=` controls join behavior. Most commonly used: `inner`, `leftouter`, `leftanti` (rows in left **not** in right — useful for "events with no matching response").

## Time-bucketing

```kql
| summarize Count = count() by bin(TimeGenerated, 1h), Computer
| render timechart
```

`bin()` rounds time to a bucket; pairs with `render timechart` for visualisations.

## Lookups against a previous row

Several detections in this repo use `prev()` to compare consecutive events:

```kql
| sort by UserPrincipalName asc, TimeGenerated asc
| extend PrevTime = prev(TimeGenerated, 1)
| extend MinutesGap = datetime_diff('minute', TimeGenerated, PrevTime)
```

Sort first; `prev()` looks at the previous row in the sorted output.

## Set membership

```kql
let PrivilegedGroups = dynamic(["Domain Admins", "Enterprise Admins"]);
| where TargetGroup in (PrivilegedGroups)
| where ParametersLower has_any (SuspiciousKeywords)
```

`dynamic([...])` creates a list. `in` for exact equality, `has_any` for substring matching across any element.

## Strings

```kql
| extend CmdLine = tolower(CommandLine)
| where CmdLine has "encodedcommand"           // word match, faster than contains
| extend Parts = split(Account, "\\")
| extend Domain = tostring(Parts[0])
| extend User = tostring(Parts[1])
| extend Full = strcat(Domain, "/", User)
```

Prefer `has` over `contains` for performance — `has` matches whole tokens (faster), `contains` matches anywhere in the string (slower).

## Boolean math for scoring

A pattern used in several detection rules — convert boolean flags to integers and add them up:

```kql
| extend SuspicionScore = 
    toint(HasEncodedCommand) * 3 +
    toint(HasDownload) * 3 +
    toint(HasHiddenWindow) * 2
| where SuspicionScore >= 4
```

## Geographic distance

KQL has built-in geospatial functions, useful for impossible-travel detection:

```kql
| extend DistanceKm = geo_distance_2points(Longitude1, Latitude1, Longitude2, Latitude2) / 1000.0
```

## Variables — `let`

```kql
let LookbackPeriod = 1h;
let FailureThreshold = 10;
SigninLogs
| where TimeGenerated >= ago(LookbackPeriod)
| summarize Failures = countif(ResultType != 0) by UserPrincipalName
| where Failures >= FailureThreshold
```

Declare constants and reusable subqueries at the top with `let`. Keeps queries tidy and tunable.

## Performance tips

- **Always filter on time first.** `| where TimeGenerated >= ago(...)` should be your first or second line.
- **`has` over `contains`** for indexed text fields.
- **`summarize` early.** Aggregating reduces row count for downstream operators.
- **`project` early** to drop unused columns.
- **Avoid `*`** in joins; specify the columns you actually need.

## References

- [KQL documentation — Microsoft Learn](https://learn.microsoft.com/azure/data-explorer/kusto/query/)
- [KQL quick reference — Microsoft Learn](https://learn.microsoft.com/azure/data-explorer/kql-quick-reference)
- [Advanced KQL for Sentinel workbook](https://github.com/Azure/Azure-Sentinel/tree/master/Workbooks)