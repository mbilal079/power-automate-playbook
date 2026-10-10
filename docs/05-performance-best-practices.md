# Performance & Best Practices

Fast, readable flows are easier to support, cheaper in API calls, and less likely to hit throttling limits.

---

## Avoid loops where you can

*Apply to each* is the most common cause of slow flows. These built-in data operations process a whole
array in **one action**:

| Instead of… | Use |
|---|---|
| Loop + *Append to array* to reshape data | **Select**: map each item to a new shape |
| Loop + *Condition* to keep some items | **Filter array** |
| Loop + *Append to string* to build text | **Select** + `join(body('Select'), ', ')` |
| Loop + *Increment variable* to add numbers | the `xpath` sum trick ([Tricks](06-tricks.md#sum-an-array-without-a-loop)) |
| Loop to build an HTML table | **Create HTML table** |
| Loop to count matches | `length(body('Filter_array'))` |

**Example: list of emails, no loop**

- **Select**, *From* `outputs('List_rows')?['body/value']`, switched to text mode, *Map* `item()?['emailaddress1']`
- **Compose**: `join(body('Select'), ';')` → `a@x.com;b@y.com`

## When you do loop

- Turn on **Concurrency control** in *Apply to each* settings (up to 50 in parallel) when the iterations don't depend on each other.
- **Don't set variables inside a parallel loop.** Variables are shared and parallel writes collide. Use **Compose**
  inside the loop and collect the results with a `Select` over the loop afterwards, or keep the loop sequential.
- Filter *before* the loop (OData filter, trigger condition or Filter array), so you only loop over the items that matter.

## Get less data

- Use **Filter rows / Filter query** and **Select columns** on *List rows* or *Get items*, rather than filtering in the flow.
- Use **Top count** when you need only a few records.
- Use **trigger conditions** and Dataverse **filtering columns**, so the flow doesn't run at all for irrelevant changes.

## Prefer Compose over variables

`Compose` is faster, needs no initialize step, and its output appears in run history for each run. Keep
variables for values that really **change** (counters, accumulators in sequential loops).

## Make flows easy to read

- **Rename every action** (`Get_customer`, not `Get_a_row_by_ID_2`) **before** writing expressions that reference
  them. Spaces become underscores in expressions.
- Add **notes** (… → Add a note) to explain anything that isn't obvious.
- Group related steps in **Scopes**. Collapsed scopes read like a table of contents.
- Use one naming convention for flows, for example `[App] - [Trigger] - [Purpose]`.

## Build for ALM

- Create flows **inside solutions**, so they move cleanly between Dev, Test and Prod.
- Use **environment variables** for URLs, email addresses, site names and IDs. Never hard-code them.
- Use **connection references** so each environment binds its own connections.
- Run flows that act on behalf of the business under a **service account**, so they don't break when someone leaves.

## Respect limits

- **Retry policy** handles brief throttling (HTTP 429). Don't add your own retry loops on top.
- Space out bulk work with **Delay**, or batch with `chunk()`, when you call APIs with tight limits.
- Huge arrays (tens of thousands of items) belong in a dataflow, a stored procedure or a batch API, not a loop.
