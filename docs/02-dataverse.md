# Dataverse Tips

Patterns for the **Microsoft Dataverse** connector: *List rows*, *Get a row*, *Add / Update a row* and the
*When a row is added, modified or deleted* trigger.

---

## Get a lookup's text value without "Get a row"

**List rows** returns lookup columns as a GUID (`_<column>_value`). To get the **display text** without an
extra *Get a row* call per record, append the OData formatted-value annotation to the column name:

| Expression | Returns |
|---|---|
| `item()?['_ms_teacher_value']` | Teacher's **GUID** |
| `item()?['_ms_teacher_value@OData.Community.Display.V1.FormattedValue']` | Teacher's **name** |

The same works for **choice**, **yes/no**, **currency** and **date** columns. It returns the label or formatted
text instead of the raw value:

```text
item()?['statuscode@OData.Community.Display.V1.FormattedValue']        → "Active"
item()?['ms_amount@OData.Community.Display.V1.FormattedValue']         → "$1,250.00"
```

## Get the first row of List rows (no Apply to each)

When you expect one row, skip the loop entirely:

```text
first(outputs('List_Academic_Session')?['body/value'])?['mb_sessionname']
```

Set **Row count = 1** on the action so Dataverse returns a single row, and guard against "no rows found":

```text
if(empty(outputs('List_rows')?['body/value']), 'Not found', first(outputs('List_rows')?['body/value'])?['name'])
```

## Set a lookup in Add / Update a row

Lookups need the **entity set name** (plural) and the record GUID:

```text
/contacts(00000000-0000-0000-0000-000000000000)
/ms_teachers(@{triggerOutputs()?['body/ms_teacherid']})
```

To **clear** a lookup, set the field to `null` in an expression. Leaving the field empty does not clear it.

## Filter rows (OData) cheat sheet

| Goal | Filter rows |
|---|---|
| Equals text | `name eq 'Contoso'` |
| Choice value | `statuscode eq 1` |
| Lookup by GUID | `_parentcustomerid_value eq @{variables('AccountId')}` |
| Null / not null | `emailaddress1 eq null` · `emailaddress1 ne null` |
| Contains / starts with | `contains(name,'Ltd')` · `startswith(name,'A')` |
| Date range | `createdon ge @{addDays(utcNow(),-7)}` |
| Combine | `statecode eq 0 and (revenue gt 100000 or numberofemployees gt 50)` |

> Always fill in **Select columns** (for example `name,emailaddress1`). Returning every column is slow and makes run history noisy.

## FetchXML for aggregates and joins

*List rows → Fetch Xml Query* handles what OData can't, such as totals and grouping in a single call:

```xml
<fetch aggregate="true">
  <entity name="opportunity">
    <attribute name="estimatedvalue" alias="total" aggregate="sum" />
    <attribute name="ownerid" alias="owner" groupby="true" />
    <filter><condition attribute="statecode" operator="eq" value="0" /></filter>
  </entity>
</fetch>
```

## Trigger best practices

- **Change type = Modified** plus **Select columns** (for example `statuscode`) fires only when those columns change.
- Add a **trigger condition** so the flow only runs for rows it cares about. When the condition is false, no run is created:
  ```text
  @equals(triggerOutputs()?['body/statuscode'], 2)
  ```
- **Prevent infinite loops** when the flow updates the same table. Skip changes made by the flow's own account:
  ```text
  @not(equals(triggerOutputs()?['body/_modifiedby_value'], 'GUID-of-the-flow-connection-user'))
  ```
- After you turn a flow off and on (or import it), webhook triggers can take **a few minutes** to start receiving events.

## Pagination

*List rows* returns 5,000 rows per page by default. For more, open **Settings → Pagination** and set a threshold
(up to 100,000), or loop with the `@odata.nextLink` / skip token.

## Files and images

Use **Download a file or an image** to get a file column's binary content. Pass `body('Download_a_file_or_an_image')`
directly to an email attachment or a SharePoint *Create file* action, with no Base64 conversion needed.
