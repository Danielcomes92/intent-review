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

## Step 1 — Gather (read, don't invent)
Use whatever the user has connected, in this order:
1. The ticket: description, acceptance criteria, comments, attachments, parent / epic, linked and duplicate tickets.
2. Documents the ticket links to (Notion, Google Docs, Confluence, design files' descriptions) and, if the project has one, the documentation section on this feature.
3. The project's rules files (`CLAUDE.md`, `AGENTS.md`, `REVIEW.md`) and **`.intent/decisions.md`** at the repository root (decisions already taken; see `references/decisions-format.md`).
4. The code the task will touch: enough to know the current behaviour, the data involved and who calls it. Do not design the implementation here.

If a tracker is not connected, say which and work with what the user pasted.

## Step 2 — Write the spec
Use `references/spec-format.md`. In short:
- **Intent** in one or two sentences.
- **Scenarios**, Given/When/Then, each tagged `explicit` (quotes the ticket or a linked doc), `decided` (settled by a project rule or a verified fact, cited) or `assumed` (your proposal, to be confirmed). Copy Gherkin already present in the ticket verbatim.
- **Size the spec to the task.** Small task (a few lines, one screen): 1–3 scenarios. Normal task: 3–6. More than ~8 means the ticket is too big: say so and propose how to split it. Add a regression scenario only when the change touches that behaviour's code; do not list untouched behaviour "just in case". If the spec ends up heavier than the task, say so in one line.
- Write in the language of the ticket.

## Step 3 — Find the gaps, ask only what matters
Walk `references/gap-checklist.md` against the ticket and the code. Sort every gap into exactly one bucket, in this order:
1. **Decided**: a project rule (`CLAUDE.md` and similar), `.intent/decisions.md`, or the ticket's own comments already settle it. Apply it, cite it, and put it under "Already decided". Do not ask. (Example: a "no dead code" rule settles whether an entry point left without callers is deleted.)
2. **Verified**: a fact you checked settles it (the translation already exists, the only caller is X, the flag is already on). Put it under "Verified" with the evidence (file:line, command or source). Do not ask.
3. **Doesn't matter**: the answer would not change the implementation. Drop it.
4. **Question**: only what is left. These are about intent, product or data owned by someone else. Write each with your proposed answer ("I'll assume X unless you say otherwise"), name who should answer it when it is not the ticket's author (product, data, design, another team), and add the matching `assumed` scenario.

Rank the questions by what a wrong guess would cost: anything irreversible or seen outside the code (analytics events and dashboards, data deletion or migration, public APIs, billing, other teams' consumers) goes first and is marked **blocking**. Never park a blocking question as "I'll mention it in the PR". Zero questions is a valid result.

**Be consistent.** When the spec removes something, it removes everything that only existed for it (routes, types, assets, translation keys, analytics events, flags), or it says explicitly why one stays and asks if that is a decision for someone else.

## Step 4 — Definition of done
A checklist: one line per scenario (each becomes a test named after it), plus anything else the ticket requires (migration, translation, documentation, flag).

## Output
Show the spec in the chat (Intent, Scenarios, Questions, Already decided, Verified, Out of scope, Done when). Then:
- Offer to post it on the ticket (or do it if `--post` was given and the user approved): one comment titled **Intent spec**, so the team and `/intent-review` find it.
- The user keeps working on the `explicit` scenarios while the questions are pending.

## --answers: close the loop
When answers arrive:
1. Update the spec: answered `assumed` scenarios become `explicit` (quote the answer), or change as answered; update the posted comment if there is one.
2. Propose the new entries for `.intent/decisions.md` (format in `references/decisions-format.md`): only answers that will matter again for other tasks, not one-off details. Write them only after the user approves; create the file and the `.intent/` folder if they do not exist.
3. If an answer settles a recurring question, propose adding it to the project's checklist section of the same file.

Never write to the ticket or to the repository without the user's approval.
