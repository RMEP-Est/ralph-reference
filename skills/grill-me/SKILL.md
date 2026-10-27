---
name: grill-me
description: Run a relentless interview, asked in small batches through Claude's interactive question box, that stress-tests a plan, design, decision, memo, or deal thesis until every open branch is resolved and the user and Claude share the same understanding. Trigger whenever the user says "grill me", "/grill-me", "grill this", "interview me", "stress-test this plan", "poke holes in my thinking", "pressure-test this", "challenge me on this", "was übersehe ich", "löcher in meinen plan", or otherwise asks to be interrogated about a half-formed plan before committing to it. Also trigger when the user presents a rough plan and asks to have its assumptions surfaced rather than asking for a written deliverable. Do NOT use when the user wants a finished artifact (memo, spec, PRD, decision paper) — that is a different skill.
---

# Grill Me

A stateless interview that hardens a plan before it is executed. It produces no
document — the only output is a sharpened, shared understanding in the
conversation itself.

Adapted from Matt Pocock's `grill-me` / `grilling` skills
(github.com/mattpocock/skills).

## The mental model

Every plan is a decision tree. Decisions depend on other decisions. The job is
to walk down that tree in waves: each wave asks every question that does not
depend on an unanswered one, then the answers reshape which questions come
next. Batching independent questions saves round-trips; batching dependent ones
destroys the ordering. Only the first is allowed.

## How questions are asked

Use the **`AskUserQuestion` tool** — Claude's interactive question box — for
every batch. Do not type the questions into the chat as text when the tool is
available.

- **Up to 4 questions per batch** (the tool's limit). Fewer is fine; never pad a
  batch to fill it.
- **Every question in a batch must be independent.** If Q2's wording or options
  would change depending on the answer to Q1, Q2 waits for the next batch.
- **2–4 concrete options per question.** Put the recommended option first and
  end its label with "(Recommended)". The option `description` carries the
  one-line why and the trade-off. The tool adds "Other" automatically, so the
  user can always override in free text — do not add an "Other" option yourself.
- **`header`** is a short chip label (max 12 characters) naming the branch, e.g.
  "Scope", "Exit route", "Pricing", "Timing".
- **`multiSelect: true`** only when the choices genuinely stack (e.g. "Which of
  these risks are real?"). Default to single-select.
- **`preview`** is optional: use it when options are easier to compare side by
  side (two term-sheet structures, two org charts, two code shapes).
- **Fallback.** If the tool is not available in this surface, ask the same batch
  as a numbered text list (Q1–Q4), each with its options and the recommended one
  marked, and tell the user they can answer in shorthand ("Q1: a, Q2: agree").

## Rules of the interview

1. **Small batches, in dependency order.** One `AskUserQuestion` call per round,
   at most 4 independent questions, then stop and wait for the answers.
2. **Always attach a recommended answer.** Every question has a recommended
   option first, with its reasoning in the description. The user should be
   reacting to a proposal, not staring at a blank page.
3. **Look it up before you ask it.** If a fact is discoverable from the
   environment — files, the repo, tools, connected systems, earlier messages —
   go and find it. Only genuine decisions get put to the user.
4. **The decisions belong to the user.** Never resolve a judgement call on their
   behalf, and never quietly assume one to keep moving.
5. **Be relentless, not agreeable.** A vague, hand-wavy, or internally
   inconsistent answer (including a thin "Other" reply) gets pushed on again in
   the next batch rather than accepted. Keep going until the branch is closed.
6. **Depth before breadth.** Keep each batch on one or two branches of the tree.
   Do not mix unrelated topics just to fill four slots.
7. **Do not start building.** No implementation, no drafting, no deliverable
   until the user confirms that shared understanding has been reached.

## Procedure

1. **Read the plan.** Take whatever the user brought — a paragraph, a doc, a
   deck, a repo, a half-thought — and map it into a decision tree. Do not show
   the tree; use it to sequence the interview.
2. **Open with the root batch.** The decisions that constrain the most
   downstream choices and do not depend on each other. Ask them in one
   `AskUserQuestion` call.
3. **Descend in waves.** After each batch, re-evaluate the tree. Collect every
   branch the answers opened, drop the ones that became irrelevant, and ask the
   next independent set. Before each batch, add one short line in the chat
   saying roughly how many open decisions remain.
4. **Probe the soft spots.** Priority targets:
   - Unstated assumptions presented as facts
   - Undefined terms that different people would read differently
   - Reversible vs. one-way-door decisions being treated the same
   - Success criteria that cannot be measured
   - Dependencies on people, systems, or approvals not yet secured
   - The failure mode the user is quietly hoping does not happen
5. **Close.** When no open branches remain, summarise the settled understanding
   in a short bulleted list and ask the user to confirm it. If they confirm,
   stop there and ask what they want to do with it.

## Boundaries

- **No artifacts.** This skill writes nothing to disk and creates no files. If
  the user wants the outcome written up afterwards, hand off — do not
  re-interview them.
- **Skip it for trivial or already fully-specified tasks.** An interview there
  is pure friction; say so and get on with the work.
- **Non-technical plans are in scope.** Investment theses, org changes,
  negotiation strategies, and personal decisions grill the same way as
  architectures.
