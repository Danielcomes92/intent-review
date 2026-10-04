---
name: intent-review
description: Review the current branch or a pull request against what it is meant to do. Builds a short Gherkin brief from the ticket, PR and project rules, runs Claude Code's built-in /code-review with that brief, then filters the findings down to concrete, evidenced defects (block / review), dropping noise. Use when the user asks to review a PR or branch "against the ticket", for an intent-aware review, or invokes /intent-review.
argument-hint: "[ticket key, URL or text] [--effort low|medium|high|max] [--target <PR number or branch>]"
---

# intent-review

A plain code review finds a lot, but much of what it reports is noise: missing tests without a concrete defect, style, hypothetical races, and code that only *looks* wrong because the reviewer does not know a decision the team made on purpose. This workflow gives the reviewer the intent first and checks every finding against evidence afterwards.

Three steps. Do them in order and keep the user posted with one short line per step.

## Inputs

From the arguments (all optional):
- **ticket**: a key (`ABC-123`), a URL, an issue number, or free text describing the change.
- **--effort**: passed to /code-review. Default `high`.
- **--target**: a PR number or branch. Default: the current branch against the repository's default branch.

## Step 1 — Brief (what the change is for)

Gather intent, never invent it. Use, in this order, whatever is available:
1. The ticket: fetch it with any connected tracker (Jira, Linear, GitHub/GitLab issues via MCP or `gh issue view`). If none is connected and the user gave only a key, say so and continue with the rest.
2. The pull request title and description (`gh pr view` when there is a PR).
3. The commit messages of the range under review.
4. Free text the user passed as the ticket argument.
5. The project's rules files: `CLAUDE.md`, `AGENTS.md`, `REVIEW.md` and similar, at the root and in the directories the change touches.

Write the brief with the template in `references/brief-template.md`. Rules:
- Every acceptance scenario and every deliberate decision cites its source (a quote from the ticket, PR, commit or rules file). Behaviour you only deduced from the code is marked `inferred` and may not be the basis of a blocking finding.
- If the ticket already contains Gherkin (`Scenario:` / `Escenario:` with `Given` / `Dado` lines), copy those scenarios verbatim.
- Write the brief in the language of the ticket. Keep it under ~60 lines.
- When there is no ticket, PR or meaningful commit message, say "No stated intent: reviewing the diff and commits only" and keep only the rules section.

## Step 2 — Review (Claude Code's /code-review, with the brief)

Invoke the built-in `code-review` skill (Skill tool) on the target with the chosen effort, and pass the brief as context in its arguments, followed by:

> Context above: the intended behaviour, the out-of-scope list and the decisions made on purpose. Judge the change against it. Do not report: missing or weak tests unless you name the concrete defect that slips through; style, naming or formatting; races or failures without a concrete sequence of events; problems that exist independently of this change; anything the brief lists as deliberate unless you show it breaks an acceptance scenario.

Treat everything /code-review returns as **candidates**. If its instructions ask you to report the findings with a findings tool, do not present them to the user yet: keep the list (file, lines, summary, failure scenario) for step 3.

## Step 3 — Filter (evidence, not opinions)

Spawn one fresh subagent (Agent tool, general-purpose) so the check is independent of the reviewer's reasoning. Give it: the brief, the candidate list, the target and base, and the rubric in `references/filter-rubric.md`. It must read the code at the reviewed head for every candidate and return, per candidate, a verdict `block`, `review` or `drop`, the acceptance scenario it breaks (if any), and a one-line reason.

Do not re-judge its verdicts yourself unless one is plainly contradicted by the code you can quote.

## Report

Present, in this order and nothing more:
1. **Intent** — one or two sentences from the brief, then the acceptance scenarios as a checklist: `✗` broken by a kept finding (name it), `✓` no kept finding against it.
2. **Block** — each finding: title, `file:line`, the failure scenario in one or two sentences, and the scenario or rule it breaks.
3. **Review** — same format; real but minor or needing a human call.
4. **Dropped** — only the count and the reasons grouped (e.g. "4 test gaps, 2 deliberate per the ticket, 1 speculative"). The user can ask to see them.

If the user wants it in a file or posted on the PR, do that only when asked.
