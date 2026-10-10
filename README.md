# Power Automate: Expressions, Tips & Best Practices

A practical field guide to **Microsoft Power Automate** cloud flows: the expressions I use every day,
Dataverse shortcuts, error handling, parent/child flows, and tricks that save actions and API calls.

Every snippet is copy-paste ready. Paste it into the **Expression** tab, or wrap it in `@{ }` inside a text field.

---

## Contents

| # | Guide | What's inside |
|---|---|---|
| 1 | [Expressions cheat sheet](docs/01-expressions-cheatsheet.md) | Text, dates and time zones, numbers, arrays, null handling, conversions, run info |
| 2 | [Dataverse tips](docs/02-dataverse.md) | Lookup text without *Get a row*, OData filters, setting lookups, FetchXML, trigger loops |
| 3 | [Error handling](docs/03-error-handling.md) | Try / Catch / Finally scopes, `result()`, run links, Terminate, retry policies |
| 4 | [Parent & child flows](docs/04-child-flows.md) | Requirements, inputs and outputs, fire-and-forget, returning errors |
| 5 | [Performance & best practices](docs/05-performance-best-practices.md) | Replacing loops with Select and Filter, concurrency, naming, ALM |
| 6 | [Tricks & hidden gems](docs/06-tricks.md) | Sum without a loop, de-duplicate, HTML to PDF, attachments, trigger conditions |

---

## The 15 expressions I use most

```text
formatDateTime(utcNow(), 'yyyy-MM-dd')                                   today's date
convertTimeZone(utcNow(), 'UTC', 'Pakistan Standard Time', 'dd/MM/yyyy HH:mm')
addDays(utcNow(), -7)                                                    a week ago
coalesce(item()?['Nickname'], item()?['FirstName'], 'there')             first non-empty value
if(empty(x), 'N/A', x)                                                   default for blanks
first(outputs('List_rows')?['body/value'])?['name']                      first row, no loop
length(body('Filter_array'))                                             count matches
join(body('Select_emails'), ';')                                         array → "a;b;c"
split(x, ',')                                                            "a,b,c" → array
union(variables('Arr'), variables('Arr'))                                remove duplicates
last(split(item()?['Email'], '@'))                                       email domain
decodeUriComponent('%0A')                                                new line
item()?['_ownerid_value@OData.Community.Display.V1.FormattedValue']      lookup display name
formatNumber(1234.5, 'C2', 'en-US')                                      $1,234.50
concat('https://make.powerautomate.com/environments/', workflow()?['tags']?['environmentName'],
       '/flows/', workflow()?['name'], '/runs/', workflow()?['run']?['name'])   link to this run
```

---

## Featured tips

### Get a lookup column's text value in *List rows*

*List rows* returns a lookup as a GUID and its type. Rather than calling *Get a row* for each record, add
**`@OData.Community.Display.V1.FormattedValue`** to the end of the column name. For example, with a *Teacher*
lookup on a *Student* table:

- `_ms_teacher_value` → the teacher's **GUID**
- `_ms_teacher_value@OData.Community.Display.V1.FormattedValue` → the teacher's **name**

[More Dataverse tips →](docs/02-dataverse.md)

### Get the first row without an *Apply to each*

```text
first(outputs('List_Academic_Session')?['body/value'])?['mb_sessionname']
```

### Wrap every production flow in Try / Catch

Put the main actions in a **Try** scope and add a **Catch** scope that runs when Try fails. In Catch, send an
alert with the error message and a link to the run, then **Terminate as Failed**.
[Full pattern →](docs/03-error-handling.md)

---

## Golden rules

1. **Rename actions first.** Expressions reference action names.
2. **Filter at the source** (OData, trigger conditions), not inside the flow.
3. **Select / Filter array instead of loops**, and never set variables in parallel loops.
4. **Solutions, environment variables and connection references** for anything that goes to production.
5. **Try / Catch with Terminate**, so failures are visible, reported and correctly marked.

---

*Maintained by **Muhammad Bilal**, Power Platform developer (Power Apps, Dataverse, Power Automate).
Found a mistake or have a trick to add? Open an issue or a pull request.*
