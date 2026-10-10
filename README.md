<p align="center">
  <img src="assets/banner.svg" alt="Power Automate: Expressions, Tips & Best Practices" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Power%20Automate-0066FF?style=for-the-badge&logo=powerautomate&logoColor=white" alt="Power Automate">
  <img src="https://img.shields.io/badge/Dataverse-038387?style=for-the-badge&logo=microsoft&logoColor=white" alt="Dataverse">
  <img src="https://img.shields.io/badge/Guides-6-8661C5?style=for-the-badge" alt="6 guides">
  <img src="https://img.shields.io/badge/Copy--paste-ready-107C10?style=for-the-badge" alt="Copy-paste ready">
</p>

<p align="center">
  A practical field guide to <b>Microsoft Power Automate</b> cloud flows: the expressions I use every day,
  Dataverse shortcuts, error handling, parent/child flows, and tricks that save actions and API calls.
</p>

> [!TIP]
> Every snippet is copy-paste ready. Paste it into the **Expression** tab of the dynamic content panel,
> or wrap it in `@{ }` inside any text field.

---

## 📚 Guides

<table>
  <tr>
    <td width="33%" valign="top">
      <h3>🧮 <a href="docs/01-expressions-cheatsheet.md">Expressions</a></h3>
      Text, dates and time zones, numbers, arrays, null handling, conversions and run info.
    </td>
    <td width="33%" valign="top">
      <h3>🗃️ <a href="docs/02-dataverse.md">Dataverse</a></h3>
      Lookup text without <i>Get a row</i>, OData filters, setting lookups, FetchXML, trigger loops.
    </td>
    <td width="33%" valign="top">
      <h3>🛡️ <a href="docs/03-error-handling.md">Error handling</a></h3>
      Try / Catch / Finally scopes, <code>result()</code>, run links, Terminate, retry policies.
    </td>
  </tr>
  <tr>
    <td width="33%" valign="top">
      <h3>🔗 <a href="docs/04-child-flows.md">Child flows</a></h3>
      Reusable parent/child flows: inputs and outputs, fire-and-forget, returning errors.
    </td>
    <td width="33%" valign="top">
      <h3>⚡ <a href="docs/05-performance-best-practices.md">Performance</a></h3>
      Select and Filter instead of loops, concurrency, naming, solutions and ALM.
    </td>
    <td width="33%" valign="top">
      <h3>🎩 <a href="docs/06-tricks.md">Tricks</a></h3>
      Sum without a loop, de-duplicate, HTML to PDF, email attachments, trigger conditions.
    </td>
  </tr>
</table>

---

## ⭐ The 15 expressions I use most

<table>
  <tr><th width="68%" align="left">Expression</th><th align="left">What you get</th></tr>
  <tr><td colspan="2"><img src="https://img.shields.io/badge/Dates-0F6CBD?style=flat-square" alt="Dates">&nbsp; <b>🗓️ Dates &amp; time</b></td></tr>
  <tr><td><code>formatDateTime(utcNow(), 'yyyy-MM-dd')</code></td><td>Today: <code>2026-10-10</code></td></tr>
  <tr><td><code>convertTimeZone(utcNow(), 'UTC', 'Pakistan Standard Time', 'dd/MM/yyyy HH:mm')</code></td><td>Local time: <code>10/10/2026 18:30</code></td></tr>
  <tr><td><code>addDays(utcNow(), -7)</code></td><td>The date a week ago</td></tr>
  <tr><td colspan="2"><img src="https://img.shields.io/badge/Text-8661C5?style=flat-square" alt="Text">&nbsp; <b>🔤 Text</b></td></tr>
  <tr><td><code>last(split(item()?['Email'], '@'))</code></td><td>Email domain: <code>contoso.com</code></td></tr>
  <tr><td><code>decodeUriComponent('%0A')</code></td><td>A new line in text</td></tr>
  <tr><td><code>formatNumber(1234.5, 'C2', 'en-US')</code></td><td>Currency: <code>$1,234.50</code></td></tr>
  <tr><td colspan="2"><img src="https://img.shields.io/badge/Logic-CA5010?style=flat-square" alt="Logic">&nbsp; <b>🧠 Logic &amp; nulls</b></td></tr>
  <tr><td><code>coalesce(item()?['Nickname'], item()?['FirstName'], 'there')</code></td><td>The first value that isn't empty</td></tr>
  <tr><td><code>if(empty(x), 'N/A', x)</code></td><td>A default for blanks</td></tr>
  <tr><td colspan="2"><img src="https://img.shields.io/badge/Arrays-107C10?style=flat-square" alt="Arrays">&nbsp; <b>📦 Arrays</b></td></tr>
  <tr><td><code>length(body('Filter_array'))</code></td><td>Number of matching items</td></tr>
  <tr><td><code>join(body('Select_emails'), ';')</code></td><td>Array → <code>a@x.com;b@y.com</code></td></tr>
  <tr><td><code>split('a,b,c', ',')</code></td><td>Text → <code>["a","b","c"]</code></td></tr>
  <tr><td><code>union(variables('Arr'), variables('Arr'))</code></td><td>Duplicates removed</td></tr>
  <tr><td colspan="2"><img src="https://img.shields.io/badge/Dataverse-038387?style=flat-square" alt="Dataverse">&nbsp; <b>🗃️ Dataverse</b></td></tr>
  <tr><td><code>first(outputs('List_rows')?['body/value'])?['name']</code></td><td>First row, no <i>Apply to each</i></td></tr>
  <tr><td><code>item()?['_ownerid_value@OData.Community.Display.V1.FormattedValue']</code></td><td>Lookup <b>display name</b>, not the GUID</td></tr>
  <tr><td colspan="2"><img src="https://img.shields.io/badge/Flow-D13438?style=flat-square" alt="Flow">&nbsp; <b>🔗 Flow &amp; run info</b></td></tr>
  <tr><td><code>concat('https://make.powerautomate.com/environments/', workflow()?['tags']?['environmentName'], '/flows/', workflow()?['name'], '/runs/', workflow()?['run']?['name'])</code></td><td>A clickable <b>link to this run</b>, for error alerts</td></tr>
</table>

<p align="right"><a href="docs/01-expressions-cheatsheet.md">See the full cheat sheet →</a></p>

---

## 💡 Featured tips

> [!TIP]
> **Get a lookup's text value in *List rows*, without *Get a row***
>
> *List rows* returns a lookup as a GUID. Add `@OData.Community.Display.V1.FormattedValue` to the end of the column name to get its display text:
>
> | Column | Returns |
> |---|---|
> | `_ms_teacher_value` | Teacher's **GUID** |
> | `_ms_teacher_value@OData.Community.Display.V1.FormattedValue` | Teacher's **name** |

> [!NOTE]
> **Get the first row without an *Apply to each***
> ```text
> first(outputs('List_Academic_Session')?['body/value'])?['mb_sessionname']
> ```
> Set **Row count = 1** on *List rows*, so Dataverse returns a single row.

> [!IMPORTANT]
> **Wrap every production flow in Try / Catch.** Put the main actions in a **Try** scope and add a **Catch**
> scope that runs when Try fails. Send an alert with the error message and a run link, then **Terminate as Failed**.
> [Full pattern →](docs/03-error-handling.md)

> [!WARNING]
> **Never set variables inside a parallel *Apply to each*.** Variables are shared across iterations, so parallel
> writes overwrite each other. Use **Compose** or **Select** instead. [Why →](docs/05-performance-best-practices.md)

---

## 🏆 Golden rules

| | Rule | Why |
|:-:|---|---|
| 1️⃣ | **Rename actions first** | Expressions reference action names; renaming later breaks them |
| 2️⃣ | **Filter at the source** | OData filters and trigger conditions beat filtering inside the flow |
| 3️⃣ | **Select / Filter array over loops** | One action instead of hundreds of iterations |
| 4️⃣ | **Solutions + environment variables** | Clean moves from Dev to Test to Prod, with nothing hard-coded |
| 5️⃣ | **Try / Catch + Terminate** | Failures are reported and correctly marked as failed |

---

<p align="center">
  <sub>Maintained by <b>Muhammad Bilal</b>, Power Platform developer (Power Apps · Dataverse · Power Automate).<br>
  Found a mistake or have a trick to add? Open an issue or a pull request. ⭐ Star the repo if it helped you!</sub>
</p>
