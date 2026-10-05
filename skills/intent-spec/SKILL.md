---
name: intent-spec
description: Turn a task's ticket into an agreed spec before writing code. Reads the ticket and everything linked to it through the connected tools (Linear, Jira, GitHub/GitLab issues, Notion, docs), writes the intended behaviour as Gherkin scenarios, finds the cases the ticket does not cover and turns them into questions with a proposed answer, and lists when the task is done. Posts the spec to the ticket on request and records answered questions in the project's .intent/decisions.md. Use when the user starts a task or ticket, asks to "spec", "understand" or "plan" a ticket, asks what is missing from a ticket, or invokes /intent-spec.
argument-hint: "<ticket key, URL or text> [--post] [--answers]"
---

# intent-spec

Make the task unambiguous before coding: what must be true at the end, which cases the ticket leaves open, and the questions to ask now instead of after the code is written. The spec is the contract that implementation, tests and `/intent-review` all use.

## Inputs
- **ticket** (required): a key (`ABC-123`), a URL, or free text.
- **--post**: after the user approves, post the spec as a comment on the ticket.
- **--answers**: the questions were answered (in the ticket or by the user): update the spec and the project's decisions.

## Step 0 — Is a spec worth it?
Skip the spec, and say so in one line, when the task is trivial: a copy or style change, a dependency bump, a one-line bug with an obvious fix, or a pure refactor with no behaviour change (then the "spec" is "existing tests stay green"). For a bug, the spec is one scenario: the reproduction, as Given/When/Then.

## Step 1 — Investigate (mandatory, before writing anything)
Read and verify first; every later step depends on it. Never skip a part because a rule "probably" answers it: the rules decide nothing until you have read them and checked the facts they apply to.

1. **The ticket**, through whatever the user has connected: description, acceptance criteria, **all comments**, attachments, parent / epic, linked and duplicate tickets. Documents it links to (Notion, Google Docs, Confluence, design files' descriptions) and the project's documentation on this feature, if any. If a tracker is not connected, say which and work with what the user pasted.
2. **The rules files**: read `CLAUDE.md`, `AGENTS.md`, `REVIEW.md` and similar at the repository root and in the directories the task touches, and **`.intent/decisions.md`** (format in `references/decisions-format.md`). Keep the list of files you read; the output names them.
3. **The code the task touches**: current behaviour, the data involved, every caller. Every factual claim the spec makes about the code ("the only entry point", "nothing else uses it", "the copy is already live") is checked and cited: `file:line`, or the search you ran and what it returned. A claim you did not verify is written as a question or as an assumption, never as a fact ("if nothing else uses it" is not allowed).
4. **Side effects, mandatory when the task removes, renames or replaces something**: search for and list everything tied to it: analytics events and properties, translation keys, feature flags and remote config, deep links and routes, types, assets, background jobs, and anything else left orphaned. Each item ends up in the spec as removed, kept with a reason, or a question.

Do not design the implementation here.

## Step 2 — Write the spec
Use `references/spec-format.md`. In short:
- **Intent** in one or two sentences.
- **Scenarios**, Given/When/Then, each tagged `explicit` (quotes the ticket or a linked doc), `decided` (settled by a rule you quoted or a fact you verified, cited) or `assumed` (your proposal, to be confirmed). Copy Gherkin already present in the ticket verbatim.
- **Size the spec to the task.** Small task (a few lines, one screen): 1–3 scenarios. Normal task: 3–6. More than ~8 means the ticket is too big: say so and propose how to split it. Add a regression scenario only when the change touches that behaviour's code; do not list untouched behaviour "just in case". If the spec ends up heavier than the task, say so in one line.
- Write in the language of the ticket.

## Step 3 — Find the gaps, ask only what matters
Walk `references/gap-checklist.md` against the ticket and what step 1 found. Sort every gap into exactly one bucket, **in this order**:

1. **Owned outside the code → always a question.** Anything product, analytics/data, copy, design, legal or another team owns is never decided silently, even if a rule or the code seems to settle it: losing or changing an analytics event, keeping or deleting translation keys or copy, a visible behaviour change the ticket does not state, data deletion, anything a dashboard, report or other team consumes. Write it as a question with your proposed answer and who should answer it.
2. **Doesn't matter**: the answer would not change the implementation. Drop it.
3. **Already answered**, applied last and only with evidence from step 1:
   - a project rule you read, quoted with its location (example: a "no dead code" rule settles that a component left without callers is deleted, once you verified it has no callers);
   - an entry in `.intent/decisions.md` or a ticket comment, quoted;
   - a fact you verified, cited with `file:line` or the search.
   It goes under **Already decided** (rules, decisions, comments) or **Verified** (facts), never as a question. Without a quote or a citation it is not answered: it is a question.
4. **Question**: everything left. Each with your proposed answer ("I'll assume X unless told otherwise"), who answers it when not the ticket's author, and the matching `assumed` scenario.

Rank the questions by what a wrong guess would cost: irreversible or externally visible ones (analytics and dashboards, data deletion or migration, public APIs, billing, other teams' consumers) go first and are marked **blocking**. Never park a blocking question as "I'll mention it in the PR". Zero questions is a valid result only when step 1 found nothing owned outside the code.

**Be consistent.** When the spec removes something, every item from step 1.4 is removed, kept with a reason, or asked. Nothing is left orphaned silently and nothing is deferred to "a separate cleanup" without asking.

## Step 4 — Definition of done
A checklist: one line per scenario (each becomes a test named after it), plus anything else the ticket requires (migration, translation, documentation, flag).

## Output
Show the spec in the chat with every section of `references/spec-format.md`: Intent, Scenarios, Questions, Already decided, Verified, Out of scope, Done when. **Already decided is always present**: when nothing applies, write "None" and list the rules files you read. Then:
- Offer to post it on the ticket (or do it if `--post` was given and the user approved): one comment titled **Intent spec**, so the team and `/intent-review` find it.
- The user keeps working on the `explicit` scenarios while the questions are pending.

## --answers: close the loop
When answers arrive:
1. Update the spec: answered `assumed` scenarios become `explicit` (quote the answer), or change as answered; update the posted comment if there is one.
2. Propose the new entries for `.intent/decisions.md` (format in `references/decisions-format.md`): only answers that will matter again for other tasks, not one-off details. Write them only after the user approves; create the file and the `.intent/` folder if they do not exist.
3. If an answer settles a recurring question, propose adding it to the project's checklist section of the same file.

Never write to the ticket or to the repository without the user's approval.
