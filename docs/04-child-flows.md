# Parent & Child Flows

A **child flow** is a reusable flow that other flows call with **Run a Child Flow**: the Power Automate
equivalent of a function. Use one to share logic (for example "build the approval email" or "create a
customer record") instead of copying the same 15 actions into every flow.

---

## Requirements

- Both flows must be in a **solution**. *Run a Child Flow* only lists solution-aware flows.
- The child uses the **Manually trigger a flow** trigger, which also defines its inputs (text, number, yes/no, file, email...).
- The child must end with **Respond to a PowerApp or flow** (or *Response*). That action returns the outputs and tells the parent it can continue.
- Under **Run only users**, set each of the child's connections to **use the connection reference**, not
  "Provided by run-only user". Otherwise the call fails when another flow invokes it.

## Building it

**Child flow: "Get customer tier"**

1. Trigger: *Manually trigger a flow*, with inputs `CustomerId` (text) and `Amount` (number).
2. Do the work: look up the customer, calculate the tier.
3. *Respond to a PowerApp or flow*, with outputs `Tier` (text) and `Discount` (number).

**Parent flow**

1. *Run a Child Flow* → pick "Get customer tier" → fill `CustomerId` and `Amount`.
2. Read the outputs:

```text
body('Run_a_Child_Flow')?['tier']
body('Run_a_Child_Flow')?['discount']
```

> [!NOTE]
> Output names are lower-cased in the response body. If an output isn't found, check the casing in run history.

## Patterns

| Pattern | How |
|---|---|
| **Reusable logic** | One child for a calculation or record creation that several flows need |
| **Fire and forget** | Put *Respond* **first** in the child, then do the long work after it. The parent continues immediately |
| **Return a status** | Return `Status` and `Message` outputs. The parent checks `equals(body('Run_a_Child_Flow')?['status'], 'OK')` |
| **Surface errors** | In the child's Catch scope, *Respond* with an error status, then Terminate as Failed |
| **Process items in parallel** | Call the child inside an *Apply to each* with concurrency switched on. Each item runs in its own child run |

## Things to watch

- The parent **waits for the child's response**, so return it as soon as the outputs are ready.
- A child flow can only be called from the **same environment**.
- Deploy the **child before the parent** when you move solutions between environments. The parent stores a reference to the child.
- Avoid deep nesting (child → child → child). Debugging crosses several run histories.
- Child runs appear in the **child's own run history**, not inside the parent run.
