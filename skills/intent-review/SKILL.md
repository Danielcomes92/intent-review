---
name: intent-review
description: Review the current branch or a pull request against what it is meant to do. Builds a short Gherkin brief from the ticket's Intent spec (or the ticket, PR, commits and project rules), runs Claude Code's built-in /code-review with that brief, then filters the findings down to concrete, evidenced defects sorted into block / review, dropping noise. Use before opening or merging a PR, when the user asks to review a change "against the ticket" or with its intent, or invokes /intent-review.
argument-hint: "[ticket key, URL or text] [--effort low|medium|high|max] [--target <PR number or branch>]"
---

# intent-review

## What it is for
A plain code review finds most real defects, but much of what it reports is noise: missing tests without a concrete defect, style, hypothetical races, and code that only *looks* wrong because the reviewer does not know a decision the team made on purpose. This workflow gives the reviewer the intent first and checks every finding against evidence afterwards, so the person reading the result sees what the change is meant to do, which acceptance scenarios it breaks, and only findings worth acting on.

It is a review aid for a person or a coding agent. It is **not** a merge gate: it never blocks anything by itself, never edits code, and never posts to the PR or the ticket unless the user asks.

It pairs with `/intent-spec`, which writes the task's spec into the ticket before coding. When that spec exists, this review uses it verbatim; when it does not, the review infers the intent from what is available.

## When to use it, and when not
- Use it on a change with behaviour to check: before opening a PR, before merging, or on someone else's PR.
- For a trivial change (copy, styling, dependency bump), a plain `/code-review` or a glance is enough; say so instead of running the whole workflow.

## Inputs
From the arguments (all optional):
- **ticket**: a key (`ABC-123`), a URL, an issue number, or free text describing the change.
- **--effort**: passed to `/code-review`. Default `high`.
- **--target**: a PR number or branch. Default: the current branch against the repository's default branch (`origin/HEAD`, else `main`/`master`); say which base you used.

Keep the user posted with one short line per step. Answer in the user's language.

## Step 1 — Brief (what the change is for)
Gather intent, never invent it. Use, in this order, whatever is available:
1. An **Intent spec** comment on the ticket (written by `/intent-spec`, format in `../intent-spec/references/spec-format.md`): use its scenarios, out-of-scope list and "Already decided" entries as they are.
2. Otherwise the ticket itself, fetched with any connected tracker (Linear, Jira, GitHub/GitLab issues via MCP or `gh issue view`), and the documents it links to. If none is connected and the user gave only a key, say so and continue with the rest.
3. The pull request title and description (`gh pr view` when there is a PR).
4. The commit messages of the range under review.
5. Free text the user passed as the ticket argument.
6. The project's rules files (`CLAUDE.md`, `AGENTS.md`, `REVIEW.md` and similar, at the root and in the directories the change touches) and **`.intent/decisions.md`** at the repository root, if present: its entries are decisions made on purpose.

Write the brief with the template in `references/brief-template.md`:
- Every acceptance scenario and every deliberate decision cites its source. Behaviour you only deduced from the code is marked `inferred` and may not be the basis of a blocking finding.
- Gherkin already in the ticket or spec is copied verbatim.
- Write it in the language of the ticket; under ~60 lines.
- With no ticket, spec, PR or meaningful commit message, say "No stated intent: reviewing the diff and commits only" and keep only the rules section.

## Step 2 — Review (Claude Code's /code-review, with the brief)
Invoke the built-in `code-review` skill (Skill tool) on the target with the chosen effort, and pass the brief as context in its arguments, followed by:

> Context above: the intended behaviour, the out-of-scope list and the decisions made on purpose. Judge the change against it. Do not report: missing or weak tests unless you name the concrete defect that slips through; style, naming or formatting; races or failures without a concrete sequence of events; problems that exist independently of this change; anything the brief lists as deliberate unless you show it breaks an acceptance scenario.

Treat everything it returns as **candidates**. If its instructions ask you to report the findings with a findings tool, do not present them to the user yet: keep the list (file, lines, summary, failure scenario) for step 3.

If the `code-review` skill is not available in this Claude Code, review the diff yourself with the same instructions and say that you did.

## Step 3 — Filter (evidence, not opinions)
Spawn one fresh subagent (Agent tool, general-purpose) so the check is independent of the reviewer's reasoning. Give it: the brief, the candidate list, the target and base, and the rubric in `references/filter-rubric.md`. It must read the code at the reviewed head for every candidate and return, per candidate, a verdict `block`, `review` or `drop`, the acceptance scenario it breaks (if any), and a one-line reason.

Do not re-judge its verdicts yourself unless one is plainly contradicted by the code you can quote.

## Report
Present, in this order and nothing more:
1. **Intent**: one or two sentences from the brief, then the acceptance scenarios as a checklist: `✗` broken by a kept finding (name it), `✓` no kept finding against it.
2. **Block**: each finding with title, `file:line`, the failure scenario in one or two sentences, and the scenario or rule it breaks.
3. **Review**: same format; real but minor, or needing a human decision (say which).
4. **Dropped**: only the count and the reasons grouped (e.g. "4 test gaps, 2 deliberate per the ticket, 1 speculative"). The user can ask to see them.

Post it on the PR or the ticket, or fix the findings, only when the user asks.
