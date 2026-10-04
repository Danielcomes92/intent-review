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

## Evaluation

Measured against plain `/code-review high` (same model, same scenarios, same ground truth) on a planted-bug benchmark of React and React Native changes: 24 tickets with acceptance criteria, 67 planted bugs (security, data fetching, hooks, React Native lifecycle, money and time, intent mismatches, architecture), decoys (correct code that looks wrong) and 4 clean changes. Half of the scenarios were used to shape the workflow; the other half was held out and only scored in aggregate.

| | `/code-review` (tuning set) | intent-review (tuning set) | `/code-review` (held out) | intent-review (held out) |
|---|---|---|---|---|
| Bugs found (recall) | 100% | 100% | 94% | 94% |
| Precision of all findings | 31% | **55%** | 25% | **43%** |
| Precision of blocking findings¹ | 60% | **95%** | 48% | **71%** |
| Correct code flagged as a bug (decoys) | 14 | **5** | 20 | **9** |
| Findings matching no known bug | 33 | **8** | 45 | **17** |
| Clean changes blocked | 2 of 2 | 1 of 2 | 2 of 2 | 1 of 2 |
| Cost per change (approx.) | $0.32 | $1.10 | $0.33 | $1.15 |

¹ Counting repeated reports of the same bug as correct.

In short: the same bugs are found, with roughly half the noise and blocking findings that are far more reliable, at about three times the cost of a plain review. Known gaps: one clean change per set still gets a blocking finding, and a few real bugs land in **review** instead of **block**.

Caveats: the benchmark is synthetic (bugs planted by an LLM in two small fake apps), so absolute numbers are optimistic; the comparison between the two tools is the useful part.

## License

MIT
