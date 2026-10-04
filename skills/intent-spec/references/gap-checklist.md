# Gap checklist

Walk it against the ticket and the code the task touches. A gap becomes a question only when its answer would change the implementation and nothing (ticket, linked docs, rules files, `.intent/decisions.md`) already answers it. Add the project's own recurring questions from the "Checklist" section of `.intent/decisions.md`.

**Who**
- Other roles: who else can see or do this, and who must not?
- Other tenants / accounts / organizations: is the data scoped?
- Signed out, expired session, missing permission.

**Data**
- Empty, missing or null values; zero; negative.
- Duplicates; the same action twice (double click, retry, two tabs).
- Large volumes: pagination, limits, performance.
- Existing data: does old data need migrating, or behave differently?

**Failure**
- The network or a dependency fails midway: what does the user see, what is saved?
- Partial success: some items succeed, others fail.
- Cancel, undo, go back.

**Time**
- Day boundaries, time zones, daylight saving changes.
- Past and future dates; expiry; things that change while the screen is open.

**Money and quantities**
- Rounding, currency, taxes; integers vs decimals.
- What is charged, refunded or counted, and when.

**Side effects**
- Notifications, emails, webhooks, analytics, audit logs.
- Other flows that use the same data or code.
- Reports and exports that will show the new data.

**Surfaces**
- Web vs mobile; small screens; offline mode.
- Languages and formats; accessibility (labels, focus, contrast).
