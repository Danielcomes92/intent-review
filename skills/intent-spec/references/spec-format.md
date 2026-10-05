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

**S2 · <name>** — decided
Given …
When …
Then …
> Source: "<quoted rule>" (CLAUDE.md:line) or the verified fact (file:line)

**S3 · <name>** — assumed (pending Q1)
Given …
When …
Then …

### Questions
1. **<question>** — blocking · ask <product / data / design / the ticket author>. I'll assume <proposal> unless told otherwise. (→ S3)
2. **<question>** — I'll assume <proposal> unless told otherwise.

### Out of scope
- <what this task does not do> (source)

### Already decided
- <decision that applies> (rule: "<quote>" CLAUDE.md:line · or .intent/decisions.md: <entry> · or ticket comment by <role>)

### Verified
- <fact that settles a would-be question> (evidence: file:line / command / source)

### Done when
- [ ] S1 <name> (test)
- [ ] S2 <name> (test)
- [ ] <migration / translations / docs / flag, if the ticket requires it>
```

Rules:
- `explicit` scenarios quote their source; `decided` ones cite the rule or the verified fact; `assumed` ones point to the question that will confirm them.
- Questions are only for what no rule, decision, comment or verified fact settles. Blocking ones go first and name who answers.
- Observable results only in "Then": what a user, an API caller or the data shows, not how the code does it.
- One behaviour per scenario. Size to the task: 1–3 for a small one, 3–6 for a normal one, more than ~8 means split the ticket.
