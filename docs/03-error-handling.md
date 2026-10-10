# Error Handling

By default, a flow that fails just stops and turns red in run history. Production flows should **catch** the
failure, **report** it with useful detail, and **end with the correct status**.

---

## The Try / Catch / Finally pattern

Group actions into three **Scope** actions and wire them with **Configure run after**:

```
┌─ Try ──────────────┐
│  your main actions │
└────────────────────┘
        │ run after: has failed, has timed out
┌─ Catch ────────────┐
│  log / notify      │
│  Terminate: Failed │
└────────────────────┘
        │ run after: is successful, has failed, is skipped, has timed out
┌─ Finally ──────────┐
│  clean-up          │
└────────────────────┘
```

| Scope | Configure run after (on the previous scope) |
|---|---|
| **Try** | normal |
| **Catch** | Try **has failed** + **has timed out** |
| **Finally** | Catch **is successful**, **has failed**, **is skipped**, **has timed out** |

In code view, the Catch scope looks like this:

```json
"Catch": {
  "type": "Scope",
  "runAfter": { "Try": [ "Failed", "TimedOut" ] },
  "actions": { }
}
```

## Find out what failed

`result('Try')` returns the status, inputs, outputs and error of every action inside the scope. Use it in a
**Filter array** to find the failed actions:

- **From:** `result('Try')`
- **Condition (advanced mode):** `@equals(item()?['status'], 'Failed')`

Then build a readable message. Connector errors usually sit in `outputs.body.error`; built-in action errors in `error`:

```text
coalesce(
  first(body('Filter_failed'))?['outputs']?['body']?['error']?['message'],
  first(body('Filter_failed'))?['error']?['message'],
  'Unknown error'
)
```

Failed action name: `first(body('Filter_failed'))?['name']`

> `result()` only lists actions **directly** inside the scope. Wrap loops in their own scope if you need the failure detail from inside them.

## Notify with a link to the failed run

Add this to the Catch email or Teams message so whoever picks it up can open the exact run:

```text
concat('https://make.powerautomate.com/environments/', workflow()?['tags']?['environmentName'],
       '/flows/', workflow()?['name'], '/runs/', workflow()?['run']?['name'])
```

A good alert includes: the **flow name** (`workflow()?['tags']?['flowDisplayName']`), the **failed action**, the
**error message**, the **record or item ID** being processed, and the **run link**.

## End with the right status: Terminate

After you catch an error, the run still shows **Succeeded** unless you say otherwise. End the Catch scope with
**Terminate → Status: Failed**, plus a code and message. Monitoring, run history and parent flows then all see
the real outcome.

## Retry policies

Every connector action has **Settings → Retry policy**:

| Policy | Use it for |
|---|---|
| **Default** (exponential, 4 retries) | Most actions: covers throttling (429) and brief outages |
| **Exponential** (custom count/interval) | Busy APIs where you want more patient retries |
| **Fixed interval** | APIs with a known recovery time |
| **None** | Non-idempotent calls (for example "create invoice"), where a retry could create duplicates |

## Other good habits

- **Validate early:** check required inputs at the top and **Terminate** with a clear message if they're missing.
- **Secure inputs/outputs** (action **Settings**): hides passwords, tokens and personal data from run history.
- **Timeouts:** set **Settings → Timeout** (ISO 8601, for example `PT10M`) on actions that can hang, such as
  approvals or HTTP calls, and handle the *timed out* branch.
- **Do until** loops need a sensible **Count** and **Timeout** limit, so a bad condition can't run for days.
- **Log to a table** (Dataverse or SharePoint) from the Catch scope, so you can see error trends over time.
