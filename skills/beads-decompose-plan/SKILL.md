---
name: beads-decompose-plan
description: Decompose an implementation plan into a graph of beads (work items) — one epic for the plan, sub-epics as strictly sequential execution waves whose beads all run in parallel, each bead labelled with agent fit, effort, and task kind. Use this skill whenever the user wants to turn a plan, spec, RFC, roadmap, or design doc into beads, tickets, or work items, or says things like "break this plan into beads", "decompose this into work items", "create beads from this plan", "split this plan up for sub-agents", or hands over a plan file and asks for it to be sliced into tasks. Always use this skill for plan decomposition — do not write beads ad hoc without it.
---

# Plan to Beads
Turn a whole implementation plan into a graph of beads that separate agents can pick up and execute in parallel.

The thing that makes this hard: each bead is later handed to a **different agent that sees only that bead's text**. It has no access to the plan, no access to this conversation, and no way to ask a follow-up question. A bead that reads well next to the plan and is useless on its own is the default failure mode. Write every bead as if it were the only thing on the worker's screen, because it will be.

## Step 0 — Require the plan file (hard stop)
This skill operates on a plan file. Before doing anything else, check whether the user has given a path to it.

If no path is present in the conversation, **stop immediately**. Do not guess a filename, do not glob the working directory for something plan-shaped, do not offer to decompose from a description in the chat, and do not start drafting beads. Tell the user plainly that a path to the plan file is required and ask them for it. Then end the turn and wait.

If a path is given but does not exist or cannot be read, say so and stop. A wrong file silently decomposed is worse than no decomposition at all.

Only once a readable plan file is in hand, continue.

## Step 1 — Read and map the plan
Read the whole plan before writing anything. Build a picture of:
- The phases the plan defines, in order, and what each delivers. These inform the ordering, but they do **not** map one-to-one onto sub-epics — sub-epics are execution waves, derived in Step 3 from what can actually run at the same time.
- Which pieces of work are genuinely independent, and which touch the same surface or depend on each other's output and therefore cannot run at once.
- Constraints, conventions, non-goals, and decisions already settled in the plan — these have to be carried into individual beads, since the worker cannot see the plan.

If the plan is ambiguous on something that materially changes the decomposition, ask the user before writing beads rather than inventing an answer.

## Step 2 — Choose the execution mode
Default to **local work**, not worktrees. Only use worktrees if the user explicitly asks for them. When defaulting to local, do not mention worktrees in the beads at all.

## Step 3 — Build the structure
- The whole plan becomes **one epic**.
- Each **sub-epic is an execution wave**: a set of beads that all run **simultaneously**, in parallel, with no ordering between them.
- Each sub-epic gets **small coding tasks** underneath.

This structure exists because of how the companion `/beads-implement` skill is used: the user runs one prompt per sub-epic, in numeric order — `/beads-implement <epic>.1`, then `/beads-implement <epic>.2`, and so on — and each run fans **every** bead of that sub-epic out to sub-agents **at once**. There is no "run this bead first, then the rest" inside a sub-epic. Ever.

The hard rule that follows: **a bead must never depend on a sibling in its own sub-epic.** Dependencies only ever point at beads in earlier sub-epics. If two pieces of work inside one of the plan's phases must be sequenced — they share a file, one consumes the other's output — that is not one sub-epic with an internal dependency; it is **two consecutive sub-epics**. Split the plan's phase. The plan's phases are thematic groupings; the sub-epics are execution layers, and when the two disagree, the execution layers win.

Derive the sub-epics like dependency layers: wave 1 is everything startable immediately; wave 2 is everything unblocked once wave 1 is done; and so on. A sub-epic containing a single bead is perfectly fine and common — a lone prerequisite is its own wave, not a straggler to bolt onto its dependents. Do not pad a wave for symmetry, and do not merge waves to reduce their count.

Within a wave, beads must also be **file-disjoint**: agents run concurrently on the same working tree, so two beads that edit the same file cannot share a wave even if neither logically depends on the other — put them in consecutive waves.

Size beads so a single agent can finish one cleanly in one sitting. A bead that quietly contains three unrelated pieces of work is really three beads.

## Step 4 — Label every bead
Every bead gets exactly one label from each of the three sets below. Agent labels (`haiku`, `sonnet`, `opus`) name a model family, never a version, and always resolve to the latest release of that family. Do not put version numbers in labels or bead text.

### Agent fit
- `agent:haiku` — simple, low-reasoning tasks with a clear, narrow scope: updating docs, renaming, config tweaks, small mechanical edits, copy changes. If the worker has to make a design decision or untangle unfamiliar code, it is not Haiku work.
- `agent:sonnet` — small to medium tasks; the work a junior to mid-level developer would handle.
- `agent:opus` — work that needs a senior developer.

When torn between `agent:haiku` and `agent:sonnet`, pick `agent:sonnet`. A Haiku bead that comes back wrong costs a re-dispatch. Haiku beads should carry `effort:low` or `effort:medium`.

### Effort
`effort:low`, `effort:medium`, `effort:high`, or `effort:xhigh`, based on what it takes to complete the task successfully *and* cleanly — not just to get something working.

### Task kind
Exactly one per bead:
- `code-change` — creates or modifies files (features, fixes, refactors, config edits)
- `review-only` — reads code, produces findings
- `research` — investigation or spike; output is notes
- `docs` — documentation writing
- `test-only` — adds or fixes tests only

When torn between `code-change` and anything else, apply the tiebreaker: if the bead requires writing files, it is `code-change`.

### A note on `bd create --parent`
`bd create` **inherits labels from the parent issue by default** when `--parent` is set — it merges the parent's labels with whatever `--labels` you pass, rather than replacing them. Creating a child under an epic that already carries e.g. `agent:opus`/`effort:xhigh` while also passing `--labels agent:haiku,effort:low` produces a bead with **both** values on **both** label sets — a conflicting bead that `/beads-implement` must stop and flag rather than guess through. Always pass `--no-inherit-labels` alongside `--labels` when creating a bead under a parent, so the child's three labels are exactly the ones stated, nothing inherited. This applies whether beads are created via a batch/graph file or one at a time ad hoc (e.g. adding a single follow-up bead mid-session) — the flag is easy to forget precisely in the ad-hoc case, since there is no decomposition checklist to catch it.

## Step 5 — Write the beads

### Shape
A bead opens with **the problem**, then says **what to build**, then pins down **defaults and non-goals**.

A bead that opens with a stack of "must" clauses is a spec fragment, not a task. The worker can satisfy every clause in it and still build the wrong thing, because nobody told them what it is *for*. Lead with the situation and the goal; the constraints land afterwards, once the worker knows what they are constraining.

### Self-contained
The description carries the goal, the why, the constraints, and whatever background from the plan and the discussion the worker needs. Never write "as discussed", "per the plan", "see the epic", or anything else that points at context the worker cannot see. If a decision was made upstream and the worker needs it, restate it in full inside the bead.

### WHAT and WHY, never HOW
No code blocks. No dictated file names, function signatures, or directory layouts. The worker decides the implementation — that is the entire reason for handing over a problem instead of a transcription. Describe the outcome and the boundaries; leave the route to the worker.

Where the plan has genuinely already settled an implementation detail (a library choice, a data contract, a naming convention that must match something else), state it as a constraint in prose, not as a snippet to copy.

### Titles
Short, action-oriented, specific. A reader scanning a list of titles should be able to tell the beads apart.

### Acceptance criteria
State how to tell the bead is done in observable terms — what a human or a test can actually check. "The code is correct" is not acceptance criteria. "Submitting the form with an expired token returns a 401 and the user is redirected to sign-in" is.

### Hard requirements and exceptions
State them explicitly, in prose. Do not illustrate them with examples — the worker does not need examples, and examples tend to get implemented literally.

### Manual testing instructions
Every bead includes manual testing instructions to be displayed to the user when the work is completed. Write them as steps the user performs themselves — where to go, what to do, what they should see. Assume the user is not the person who wrote the code.

## Step 6 — Check the beads before handing them over
Before presenting the decomposition, reread each bead cold, as if you had never seen the plan:
- Could you start work on this with only this text? If not, what is missing?
- Does it open with a problem, or with a wall of requirements?
- Has any HOW leaked in — a filename, a snippet, a signature?
- Are the acceptance criteria observable?
- Does it have exactly one task-kind label, one agent label, and one effort label?
- If it is `agent:haiku`: is the task really mechanical, with no judgement calls and no cross-file reasoning? If not, bump it to `agent:sonnet`.
- Are there manual testing instructions?
- Does it depend on a sibling in its own sub-epic? If so the structure is wrong — split the wave, do not keep the dependency.
- Could it really run at the same instant as every sibling in its sub-epic — no shared files, no consumed outputs?

Then summarise for the user: the epic, the sub-epics in order, and the beads under each with their labels — making clear that each sub-epic is one `/beads-implement` prompt whose beads all fan out at once.
