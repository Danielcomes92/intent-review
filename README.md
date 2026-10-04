# intent-review

Intent-aware code review for [Claude Code](https://claude.com/claude-code), built on top of its own `/code-review`.

`/code-review` is broad and cheap: it finds most real defects in one pass. Its weakness is noise — missing-test remarks, style, hypothetical races, and code that only looks wrong because the reviewer does not know a decision made on purpose. `intent-review` wraps it in two steps:

1. **Brief.** Before reviewing, it reads the ticket, the pull request, the commits and the project's rules files (`CLAUDE.md`, `AGENTS.md`, `REVIEW.md`) and writes a short brief: the intended behaviour as Gherkin scenarios (each one quoting its source), what is out of scope, and the decisions made on purpose.
2. **Review.** It runs the built-in `/code-review` with that brief as context.
3. **Filter.** A fresh, independent agent checks every finding against the code and keeps only concrete, evidenced defects caused by the change, sorted into **block** and **review**. Everything else is dropped and counted.

The result opens with what the change is meant to do, shows which acceptance scenarios are broken, and lists only findings worth acting on.

## Install

In Claude Code:

```
/plugin marketplace add Danielcomes92/intent-review
/plugin install intent-review@intent-review
```

Or load it from a local clone for one session:

```sh
claude --plugin-dir /path/to/intent-review
```

## Use

```
/intent-review                          # current branch vs the default branch
/intent-review ABC-123                  # with a ticket (fetched through any connected tracker)
/intent-review "Customers can pay with saved cards; a declined card shows an error"
/intent-review ABC-123 --target 482     # a pull request
/intent-review --effort max
```

The ticket can come from any tracker you have connected to Claude Code (Jira, Linear, GitHub or GitLab issues). Without one, it uses the PR description and commit messages; with nothing at all, it says so and reviews the diff against your rules files.

**Tip:** write acceptance criteria as Gherkin in the ticket. They are copied verbatim into the brief, so the review judges exactly what you meant.

## How it decides

- A finding is kept only with a concrete failure scenario, quoted evidence, and a cause in this change.
- **block**: breaks an acceptance scenario or a project rule, or is a security, data, money/time or core-flow defect.
- **review**: real but minor, or needs a human decision.
- Scenarios the brief only *inferred* from the code never produce a blocking finding.
- The full rubric is in [`skills/intent-review/references/filter-rubric.md`](skills/intent-review/references/filter-rubric.md).

## Cost

Roughly one `/code-review` plus two short agent runs (brief and filter) per review.

## Status

Early. An evaluation against a planted-bug benchmark (recall, precision and false blocks compared with plain `/code-review`) is in progress; results will be published here.

## License

MIT
