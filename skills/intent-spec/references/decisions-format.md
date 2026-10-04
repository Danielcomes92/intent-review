# `.intent/decisions.md` format

One file per project, at the repository root, committed like any other file. It holds the answers that will matter again, so they are not asked twice and `/intent-review` does not flag them as bugs. It grows one approved entry at a time; nothing is added without the user's OK.

```markdown
# Project decisions

<!-- Read by /intent-spec and /intent-review. Newest first. Each entry: date, decision, source. -->

## Decisions

- **2026-10-03 · Inactive members are not charged by the monthly run.** They keep their history and can be reactivated. Source: ABC-482 (answer by the product owner).
- **2026-09-20 · Prices are stored in integer cents; VAT is rounded half up per line.** Source: ABC-311. Supersedes the 2026-08 entry on totals.

## Checklist

<!-- Project-specific questions to always ask, on top of the generic gap checklist. -->
- Does it affect electronic invoicing?
- Does it change what the mobile app caches offline?

## Sources

<!-- Where things live, so the skills look in the right place. -->
- Product specs: <tracker / workspace>
- Pricing rules: <document>
```

Rules:
- One line per decision, with date and source (ticket, doc or person's role). No secrets, no personal data.
- A new decision that changes an old one says "Supersedes <date>"; delete the old entry when it no longer applies.
- Keep it short. If an entry is really a code rule, move it to `CLAUDE.md`.
