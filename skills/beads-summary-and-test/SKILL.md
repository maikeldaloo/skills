---
name: beads-summary-and-test
description: Summarize the work done on a Beads epic, verify it against the actual codebase (committed or not), then walk the user through manual testing one step at a time. Use when the user runs /beads-summary-and-test, or asks to "summarize and test this epic", "walk me through testing this epic", or hands over an epic ID after implementation and wants a brief recap plus guided manual verification. Do not use for running/implementing beads — that is beads-implement.
---

# Beads Summary and Test
Recap what an epic actually did, confirm it against the real codebase, then hand the user a guided, one-step-at-a-time manual test script with concrete URLs. Brevity is the point — this is a wrap-up tool, not a report.

## Step 0 — Require the epic ID (hard stop)
This skill operates on a specific Beads epic ID.

If no epic ID is present in the conversation or command args, **stop immediately**. Do not guess, do not list epics and pick one, do not infer it from recent git history. Ask the user for the epic ID plainly, then end the turn and wait.

If the ID doesn't resolve to a real bead, say so and stop.

## Step 1 — Gather the epic's actual scope
Load the epic and walk its children (recursively through sub-epics) via `bd show`/`bd dep list` or equivalent. For each closed/done child, read its description, acceptance criteria, and any close notes. Ignore anything still open or blocked — this skill summarizes and tests what's *done*, and should flag incomplete children rather than testing them.

## Step 2 — Verify against the codebase, not against the beads' claims
Beads describe intent; the working tree is ground truth. For every child bead's claimed changes:

- Confirm the files it claims to touch actually exist and contain the described change (`git diff`, `git log`, `Read` the current file — check committed AND uncommitted state, since work may not be committed yet).
- Do not re-run full test suites here unless something looks contradictory — if a prior orchestration pass (e.g. `beads-implement`) already verified this epic's work, trust that unless the working tree has changed since. Spend effort on catching drift between what a bead *says* happened and what is *actually in the repo* — stale claims, half-applied diffs, files that were reverted after being marked done.
- Note anything that doesn't match: claimed but missing, described differently than implemented, or silently reverted.

Keep this step internal — its output feeds the summary and the test plan, but don't dump raw diffs or file listings on the user.

## Step 3 — Give a brief summary
Output format, kept short:
- One line per completed child bead: what changed, in plain terms (not the bead's title verbatim).
- One line flagging any verification mismatches found in Step 2, if any. If none, say so in a half-line, don't skip it silently.
- Nothing else. No architecture explanations, no restating acceptance criteria, no "here's what we'll do next" preamble beyond a one-line transition into testing.

If the epic has open/blocked children, name them in one line so the user knows the summary is partial.

## Step 4 — Build the test plan, but reveal it one step at a time
Before showing anything, silently plan the full sequence of manual test steps covering every completed child's user-observable behavior. Each step must be self-contained and include:

- **A concrete URL** to open — resolve the actual dev server port from `CLAUDE_LOCAL.md`/`CLAUDE.md`/project config, never guess. If a bead touches multiple apps, use each app's own port.
- **What to do** — the exact action (click X, fill Y, navigate to Z), not a restatement of the acceptance criteria.
- **What to expect** — the concrete, observable outcome, specific enough that a mismatch is unambiguous.

Split every step into lettered checks, one action and one outcome each, so the user can test one thing at a time and report failures by id. Never group several actions under one expectation. Format each check exactly like this (number = step number, letter = check identifier, uppercase):

```
**Check 3A**
- **DO:** instructions
- **EXPECT:** expected results
```

The step's heading ("Step 3 of 39 — title") and its URL come first, then Check 3A, 3B, and so on. The user replies with the failed ids (e.g. "3C, 3F") or "all pass". Count a step as passed only when all its checks pass.

Do NOT print the whole plan up front. Send only step 1. Wait for the user's reply (pass, fail, or feedback) before sending step 2, and so on, in the same message-per-step rhythm all the way through. This is a hard requirement, not a default — even if you're confident the user wants the full list, hold it back.

Since the full plan is built silently before the first step is sent, its total length is already known — label every step with its position and the total, plus a short title, e.g. "Step 1 of 6 — Confirm a cart id appears in local storage". Keep the total fixed once testing starts; if a mid-plan finding (e.g. a bug the user reports) leads you to add a step, say so explicitly ("adding a step — now 7 total") rather than silently changing the count.

When the user reports a failure or unexpected result on a step, do not immediately jump to fixing code. Acknowledge it, ask only what's needed to understand the mismatch, and let the user decide whether to pause testing to fix it now or keep going and fix afterward. Resume the step sequence from the same point once they're ready.

After the last step, give a one-line close: how many steps passed/failed, nothing more unless the user asks for detail.

## Non-goals
- Don't run or implement beads — that's `beads-implement`.
- Don't write or edit code as part of this skill, even to fix something the user finds while testing, unless the user explicitly asks you to switch into fixing it.
- Don't dump the full test plan at once — one step, wait, next step.
- Don't pad the summary. If a one-liner covers a bead, use one line.
