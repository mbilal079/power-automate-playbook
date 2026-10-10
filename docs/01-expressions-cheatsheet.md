# Expressions Cheat Sheet

The expressions you reach for every day, grouped by type. Paste them into the **Expression** tab of the dynamic
content panel, or wrap them in `@{ }` inside text fields.

> **Tip:** always use the safe-navigation operator `?[...]`, as in `outputs('Get_item')?['body/Title']`.
> It returns `null` instead of failing when a property is missing.

---

## Text

| Goal | Expression | Result |
|---|---|---|
| Join text | `concat('INV-', variables('Number'))` | `INV-1042` |
| Upper / lower case | `toUpper('abc')` · `toLower('ABC')` | `ABC` · `abc` |
| Trim spaces | `trim('  hello  ')` | `hello` |
| Length | `length('Power')` | `5` |
| Part of a string | `substring('Power Automate', 6, 8)` | `Automate` |
| Part by start/end index | `slice('Power Automate', 0, 5)` | `Power` |
| Find position | `indexOf('a@b.com', '@')` | `1` |
| Text after a character | `last(split('john@contoso.com', '@'))` | `contoso.com` |
| Text before a character | `first(split('john@contoso.com', '@'))` | `john` |
| Replace | `replace('2026/10/10', '/', '-')` | `2026-10-10` |
| Contains / starts / ends | `contains(x,'urgent')` · `startsWith(x,'RE:')` · `endsWith(x,'.pdf')` | `true`/`false` |
| New line | `decodeUriComponent('%0A')` | line break |
| Escape a single quote | `'It''s done'` | `It's done` |
| Remove all spaces | `replace(x, ' ', '')` | |
| Pad a number | `formatNumber(7, '0000')` | `0007` |
| Format currency | `formatNumber(1234.5, 'C2', 'en-US')` | `$1,234.50` |
| Thousands separator | `formatNumber(1234567, 'N0')` | `1,234,567` |

> A text field that starts with `@` is treated as an expression. To start text with a literal `@`, write `@@`.

## Dates & times

All Power Automate times are **UTC**. Convert at the edge, when you display or compare local dates.

| Goal | Expression |
|---|---|
| Now (UTC) | `utcNow()` |
| Today as `yyyy-MM-dd` | `formatDateTime(utcNow(), 'yyyy-MM-dd')` |
| Friendly format | `formatDateTime(utcNow(), 'dddd, MMMM d, yyyy')` → `Saturday, October 10, 2026` |
| Convert to local time | `convertTimeZone(utcNow(), 'UTC', 'Pakistan Standard Time', 'dd/MM/yyyy HH:mm')` |
| Add / subtract days | `addDays(utcNow(), 7)` · `addDays(utcNow(), -30, 'yyyy-MM-dd')` |
| Add hours / minutes | `addHours(utcNow(), 2)` · `addMinutes(utcNow(), 15)` |
| Add months | `addToTime(utcNow(), 1, 'Month')` |
| Start of day / month | `startOfDay(utcNow())` · `startOfMonth(utcNow())` |
| Day of week (0 = Sunday) | `dayOfWeek(utcNow())` |
| Is it a weekend? | `or(equals(dayOfWeek(utcNow()), 0), equals(dayOfWeek(utcNow()), 6))` |
| Days between two dates | `div(sub(ticks(variables('End')), ticks(variables('Start'))), 864000000000)` |
| Minutes between | `div(sub(ticks(variables('End')), ticks(variables('Start'))), 600000000)` |
| Excel serial date → date | `addDays('1899-12-30', int(item()?['Date']), 'yyyy-MM-dd')` |
| Unix timestamp → date | `addSeconds('1970-01-01T00:00:00Z', int(variables('Epoch')))` |
| Date → Unix timestamp | `div(sub(ticks(utcNow()), ticks('1970-01-01T00:00:00Z')), 10000000)` |

> `ticks()` counts 100-nanosecond intervals, so one day = `864000000000` ticks.

## Numbers & math

| Goal | Expression |
|---|---|
| Add / subtract / multiply / divide | `add(a,b)` · `sub(a,b)` · `mul(a,b)` · `div(a,b)` |
| Remainder | `mod(10, 3)` → `1` |
| Text → number | `int('42')` · `float('42.75')` |
| Round to 2 decimals | `float(formatNumber(variables('Amount'), 'F2'))` (there is no `round()` function) |
| Min / max | `min(3, 7, 1)` · `max(3, 7, 1)` |
| Random number | `rand(1, 100)` |
| Is it a number? | `isFloat('12.5')` · `isInt('12')` |

## Arrays & collections

| Goal | Expression |
|---|---|
| Count items | `length(body('Get_items')?['value'])` |
| Is it empty? | `empty(body('List_rows')?['value'])` |
| First / last item | `first(...)` · `last(...)` |
| First N / skip N | `take(arr, 10)` · `skip(arr, 10)` |
| Item by index | `body('List_rows')?['value']?[0]` |
| Join into text | `join(body('Select_emails'), '; ')` |
| Text → array | `split('a,b,c', ',')` |
| Array contains value | `contains(variables('Ids'), 'X-12')` |
| Remove duplicates | `union(variables('Arr'), variables('Arr'))` |
| Items in both arrays | `intersection(variables('A'), variables('B'))` |
| Number sequence | `range(1, 5)` → `[1,2,3,4,5]` |
| Reverse | `reverse(variables('Arr'))` |
| Sort by property | `sort(body('List_rows')?['value'], 'createdon')` |
| Split into batches | `chunk(variables('Arr'), 100)` |
| Build an array inline | `createArray('a', 'b', 'c')` |

## Logic & null handling

| Goal | Expression |
|---|---|
| If / else | `if(equals(x, 'Approved'), 'Yes', 'No')` |
| And / or / not | `and(a, b)` · `or(a, b)` · `not(a)` |
| Compare | `equals` · `greater` · `greaterOrEquals` · `less` · `lessOrEquals` |
| First non-null value | `coalesce(item()?['Nickname'], item()?['FirstName'], 'there')` |
| Null or empty check | `empty(x)` (works on text, arrays and objects) |
| Default for missing text | `if(empty(triggerBody()?['Notes']), 'No notes', triggerBody()?['Notes'])` |

## Conversion & encoding

| Goal | Expression |
|---|---|
| Text → JSON object | `json('{"a":1}')` |
| Object → text | `string(body('HTTP'))` |
| To boolean | `bool('true')` |
| Base64 encode / decode | `base64('hello')` · `base64ToString('aGVsbG8=')` |
| Base64 → binary (for files) | `base64ToBinary(variables('FileB64'))` |
| URL encode / decode | `uriComponent('a b&c')` · `decodeUriComponent('a%20b')` |
| New GUID | `guid()` |
| XML → query with XPath | `xpath(xml(body('Get_file')), '//Invoice/Total/text()')` |

## Flow & run information

| Goal | Expression |
|---|---|
| Flow display name | `workflow()?['tags']?['flowDisplayName']` |
| Environment ID | `workflow()?['tags']?['environmentName']` |
| Current run ID | `workflow()?['run']?['name']` |
| **Link to this run** | `concat('https://make.powerautomate.com/environments/', workflow()?['tags']?['environmentName'], '/flows/', workflow()?['name'], '/runs/', workflow()?['run']?['name'])` |
| Trigger body | `triggerBody()` · `triggerOutputs()?['body/...']` |
| Current loop item | `item()` · `items('Apply_to_each')` |
| Action status (inside a scope) | `result('Try')` |
