# Tricks & Hidden Gems

Small techniques that save actions, API calls or a lot of head-scratching.

---

## Sum an array without a loop

There is no `sum()` function, but XPath has one. First **Select** the numbers into a plain array (text mode,
Map `item()?['amount']`), then:

```text
xpath(
  xml(json(concat('{"root":{"v":', string(body('Select_amounts')), '}}'))),
  'sum(/root/v)'
)
```

`count(/root/v)`, `max` and `min` work the same way.

## Remove duplicates

`union()` with the same array twice returns each distinct value once:

```text
union(variables('Emails'), variables('Emails'))
```

## Look up a value by key, without a Switch

Store a mapping as JSON in a **Compose** (or an environment variable) and index into it:

```text
Compose "Region managers":  { "North": "anna@contoso.com", "South": "raj@contoso.com" }

outputs('Region_managers')?[triggerBody()?['Region']]
```

## Line breaks in text and emails

| Where | Use |
|---|---|
| Plain text, Teams, `join()` | `decodeUriComponent('%0A')` |
| HTML email body | `<br>` |
| Convert text newlines for an HTML email | `replace(x, decodeUriComponent('%0A'), '<br>')` |

## Escape user text before putting it in HTML

```text
replace(replace(replace(coalesce(x, ''), '&', '&amp;'), '<', '&lt;'), '>', '&gt;')
```

This stops a comment containing `<` or `&` from breaking your email layout.

## Style the "Create HTML table" output

The action outputs a bare `<table>`. Add inline styles with `replace()` before you send it:

```text
replace(
  replace(body('Create_HTML_table'), '<table>', '<table style="border-collapse:collapse;font-family:Segoe UI">'),
  '<td>', '<td style="border:1px solid #ddd;padding:6px 10px">'
)
```

## Email attachments from several files

Build an array variable of `{ "Name": ..., "ContentBytes": ... }` objects and pass the whole variable to the
**Attachments** field of *Send an email (V2)*, after switching the field to array input:

```json
{
  "Name": "@{outputs('Get_file_metadata')?['body/Name']}",
  "ContentBytes": "@body('Get_file_content')"
}
```

> [!NOTE]
> Pass the file action's `body()` straight through. That's the binary content the email action expects.

## Generate a PDF from HTML, with no premium connector

1. **OneDrive for Business → Create file**: name it `temp.html`, content = your HTML (built with Compose).
2. **Convert file**: File = the created file's Id, Target type = `PDF`.
3. **Delete file**: remove the temp HTML.

The converter keeps fonts, tables, borders and text colours, but **drops background colours**, so design the
PDF with borders and coloured text rather than filled boxes.

## Trigger conditions worth knowing

```text
@equals(triggerOutputs()?['body/Status'], 'Approved')                    only when approved
@not(empty(triggerOutputs()?['body/Email']))                             field is filled in
@contains(toLower(triggerOutputs()?['body/Subject']), 'invoice')         subject keyword
@not(equals(triggerOutputs()?['body/Editor/Email'], 'flow@contoso.com')) skip the flow's own edits
```

Several conditions on one trigger combine with **AND**. For OR, put `or(...)` inside a single condition.

## Work-day maths: add N business days

There's no built-in function. Loop with **Do until** over `addDays()` and skip days where
`dayOfWeek()` is `0` or `6`, plus any dates from a holidays list. Better still, put this in a
[child flow](04-child-flows.md) and reuse it everywhere.

## Parse JSON is optional

You don't need **Parse JSON** just to read properties. `body('HTTP')?['customer']?['name']` works directly.
Parse JSON only helps when you want dynamic-content tokens in the designer. Generate its schema from a
real sample and **remove `required`** entries, so null fields don't fail the action.

## Quick debugging

- Drop in a **Compose** that outputs the expression you're unsure about, run the flow, and read the result in run history.
- **Peek code** (… on any action) shows the exact JSON and expression the action uses.
- **Resubmit** a failed run after a fix, so you don't need to recreate the trigger event.
- *Test → Automatically → with a recently used trigger* replays real trigger data.
