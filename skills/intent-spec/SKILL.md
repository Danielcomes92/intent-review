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
- **Scenarios**, 3–8, Given/When/Then, each tagged `explicit` (quotes the ticket or a linked doc) or `assumed` (your proposal, to be confirmed). Copy Gherkin already present in the ticket verbatim.
- More than ~8 scenarios means the ticket is too big: say so and propose how to split it.
- Write in the language of the ticket.

## Step 3 — Find the gaps, ask only what matters
Walk `references/gap-checklist.md` against the ticket and the code. For each gap:
- If `.intent/decisions.md` or the rules files already answer it, apply that answer, cite it, and do not ask.
- If the answer would not change the implementation, drop it.
- Otherwise write a question with your proposed answer ("I'll assume X unless you say otherwise") and add the matching `assumed` scenario.
Aim for few, sharp questions. Zero is a valid result.

## Step 4 — Definition of done
A checklist: one line per scenario (each becomes a test named after it), plus anything else the ticket requires (migration, translation, documentation, flag).

## Output
Show the spec in the chat (Intent, Scenarios, Questions, Done). Then:
- Offer to post it on the ticket (or do it if `--post` was given and the user approved): one comment titled **Intent spec**, so the team and `/intent-review` find it.
- The user keeps working on the `explicit` scenarios while the questions are pending.

## --answers: close the loop
When answers arrive:
1. Update the spec: answered `assumed` scenarios become `explicit` (quote the answer), or change as answered; update the posted comment if there is one.
2. Propose the new entries for `.intent/decisions.md` (format in `references/decisions-format.md`): only answers that will matter again for other tasks, not one-off details. Write them only after the user approves; create the file and the `.intent/` folder if they do not exist.
3. If an answer settles a recurring question, propose adding it to the project's checklist section of the same file.

Never write to the ticket or to the repository without the user's approval.
