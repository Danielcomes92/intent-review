# Intent spec format

Shared by `/intent-spec` (writes it) and `/intent-review` (reads it from the ticket comment titled **Intent spec**). Keep the headings.

```markdown
## Intent spec — <ticket key>

**Intent:** <1–2 sentences: what must be true at the end, for whom.>

### Scenarios
**S1 · <name>** — explicit
Given <state>
When <action>
Then <observable result>
> Source: "<quote>" (<ticket / linked doc>)

**S2 · <name>** — assumed (pending Q1)
Given …
When …
Then …

### Questions
1. **<question>** — I'll assume <proposal> unless told otherwise. (→ S2)

### Out of scope
- <what this task does not do> (source)

### Already decided
- <decision that applies> (from .intent/decisions.md: <entry date / ticket>)

### Done when
- [ ] S1 <name> (test)
- [ ] S2 <name> (test)
- [ ] <migration / translations / docs / flag, if the ticket requires it>
```

Rules:
- `explicit` scenarios quote their source. `assumed` ones point to the question that will confirm them.
- Observable results only in "Then": what a user, an API caller or the data shows, not how the code does it.
- One behaviour per scenario. 3–8 scenarios.
