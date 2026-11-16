---
name: beads-implement
description: Orchestrate the execution of a sub-epic of beads (work items) by fanning its child beads out to parallel sub-agents, then verifying all the results centrally. Use this skill whenever the user wants to build, run, execute, or implement beads — including phrases like "run phase 1", "implement this sub-epic", "execute these beads", "build out the beads", "kick off the sub-agents for this", or when they hand over a bead ID and ask for the work to be done. Always use this skill for bead execution — do not dispatch sub-agents or start implementing beads without it.
---

# Beads Implement

Orchestrate one sub-epic of a bead graph. Launch its child beads across parallel sub-agents — each seeing only its own bead — then verify every result yourself.

Two things define this role. First, **you write no implementation code.** Your job is dispatch, sequencing, and verification. Second, **all verification is yours alone.** Sub-agents produce artifacts; they do not get to grade their own work. An agent that just wrote a change is the worst possible judge of whether it broke something, and an agent running tests in its own sandbox tells you nothing about the state of the repo after every wave lands together.

Default to local work unless a bead explicitly says otherwise.

## Step 0 — Require the bead ID (hard stop)
This skill operates on a specific bead ID.

If no bead ID is present in the conversation, **stop immediately**. Do not guess, do not list beads and pick a plausible-looking one, do not start from the top of the graph, and do not begin dispatching anything. Tell the user plainly that a bead ID is required and ask for it. Then end the turn and wait.

If the ID is given but does not resolve to a real bead, say so and stop. Executing the wrong branch of the graph is far more expensive to undo than asking.

## Step 1 — Resolve and select
Load the bead and check what kind of node it is:
- **Sub-epic** — run its children. This is the normal case.
- **Epic** — stop. List its sub-epics and ask the user which one to run. An epic is a whole plan; running it wholesale defeats the phase-by-phase model the decomposition was built for.
- **Leaf bead** — run just that one.

## Step 2 — Build the wave
A bead is runnable only when all of its dependencies are `done`. Group the currently runnable beads into a wave.

Then check the wave for collisions: **never put two beads that edit the same files into one wave.** Parallel agents writing the same surface produce work that looks fine individually and is broken together. Where you find a collision, serialise the pair and tell the user which beads you serialised and why, so they can see the shape of what is actually running concurrently.

## Step 3 — Dispatch, one sub-agent per bead
Before dispatching a bead, set its status to `in_progress` (e.g. `bd update <id> --status in_progress`). Do this for every bead in the wave as you pick it up, right before launching its sub-agent — it marks the bead as claimed for the duration of the run, not just planned.

Hand each sub-agent **only its own bead's text**. No plan, no sibling beads, no access to this conversation. The beads were written to stand alone; treat that as a contract and do not paper over a weak bead by smuggling context in alongside it. If a bead genuinely cannot be executed on its own, that is a finding to report, not a gap to fill in silently.

### Dispatch: one `Agent` call per bead
Dispatch each bead with its own `Agent` call, launching the whole wave in a single message so the calls run concurrently. Never use the `Workflow` tool for this skill.

Before dispatching, show the user a table of the wave: bead · model · effort · isolation.

Use `isolation: "worktree"` only when the wave's beads touch disjoint files and the earlier waves are committed. A worktree starts from the last commit, so it cannot see uncommitted work from earlier waves; when that work is uncommitted, dispatch without isolation. Merge each worktree back before verifying.

Pick the model from the bead's `agent:` label — `haiku`, `sonnet`, or `opus` — and the effort from its `effort:` label. Do not substitute either. The labels encode a judgement made with full sight of the plan; overriding them from inside a single bead's context is guesswork.

Pass the label value as the bare alias (`model: "opus"` / `model: "sonnet"` / `model: "haiku"`), not a versioned model ID. The `Agent` tool only accepts aliases, and each alias always resolves to the latest model in its family. Never write a model version number into this skill, a bead, or a dispatch prompt.

This is a hard constraint on the tool call itself, not a suggestion to weigh:
- **Model — always.** Explicitly set the `model` parameter to the bead's `agent:` label on every single dispatch. Never omit it to let the tool auto-route or inherit a default, and never swap one model for another (e.g. `opus` for `sonnet`, or `sonnet` for `haiku`) because the task looks simpler or costlier than the label implies.
- **Effort — always.** Explicitly set the `effort` parameter to the bead's `effort:` label on every single dispatch. Never omit it: an omitted effort silently falls back to the per-model default in the user's settings, not the label. Never round an `effort:xhigh` bead down to `high` because a model doesn't nominally support it.

If a bead is missing either label, that is a defect in the bead — stop and flag it to the user rather than guessing a model or effort to fill the gap.

The same applies in the other direction: if a bead carries **more than one** value from the same label set (e.g. both `agent:opus` and `agent:haiku`, or both `effort:low` and `effort:xhigh`), that is a conflicting bead, not a bead you can resolve by picking the "obviously right" one. This commonly happens when a bead was created with `bd create --parent <epic>` without `--no-inherit-labels` — the parent epic's own labels silently merge with whatever `--labels` was passed. Stop and ask the user which value to use, exactly as you would for a missing label; do not guess, and do not default to the parent epic's label on the theory that it's more authoritative.

Honour the task-kind label:
- `code-change` — writes files
- `review-only` — produces findings, changes nothing
- `research` — produces notes
- `docs` — produces documentation
- `test-only` — touches tests only

### What to tell every sub-agent
- Meet the acceptance criteria by producing the required artifacts — code, docs, notes, review findings, or tests.
- **Do not run any tests, linters, type checks, e2e suites, builds, or other verification commands.**
- **Do not perform validation beyond making sure your own changes are internally consistent** — obvious syntax errors you can see at a glance, nothing more.
- Report in observable terms: what you did, which files you created, modified, or deleted, and any manual testing instructions you propose for the orchestrator to run.
- Leave all verification to the orchestrator.

## Step 4 — Run the wave and verify centrally
Launch the whole wave concurrently. Wait for all of it to finish before judging any of it — a bead that looks green in isolation can still be broken by what its sibling landed.

Then, for each completed bead:
1. Collect its reported changes and its proposed manual testing instructions.
2. **Run all verification yourself**: tests, e2e, lint, type checks, build — whatever the repo actually has.
3. Mark the bead `done` only when both hold: every acceptance criterion is met in observable terms, *and* every verification command you ran passes. If you accept a failure, document explicitly what failed and why you accepted it — never mark something done past a silent failure.

On verification failure, keep the bead open. Record what failed — the commands, the output, the files involved — and either re-dispatch with that context attached or flag it for a human if it looks like the bead itself is wrong.

The retry-loop rule: re-dispatch a failed bead at most once on your own. A second re-dispatch of the same bead or fix needs the user's confirmation first; say what failed both times.

If an `agent:haiku` bead fails verification, flag it to the user rather than re-dispatching on the same model. The label may have been too optimistic. Do not silently upgrade it to `sonnet`. Ask first (see the retry-loop rule).

## Step 5 — Next wave, then report
Recompute which beads are now runnable and run the next wave. Continue until every child is `done` or blocked.

Don't ask the user to run manual tests per wave. Append the proposed manual checks to the parent epic's notes (`bd update <epic> --append-notes`; never `--notes`, which replaces them) for `/beads-summary-and-test`. Only ask now if a later wave depends on the result. Collect any verification failures you accept in one end-of-epic bead rather than fixing them mid-wave.

Then report:
- What completed.
- What is blocked, and why.
- The full set of verification and manual testing steps you ran across the whole sub-epic, with their results.

Finish with one standalone Conventional Commit subject line for the completed work, for example `feat(<feature>): <description>`. Include no additional explanation with that line.

## Non-goals
- Don't start beads outside this sub-epic.
- Don't edit a bead's description to make it pass. If a bead is wrong, say so.
- Don't collapse multiple beads into a single sub-agent, even when they look adjacent.
- Don't delegate test execution, linting, type checking, e2e, or builds to sub-agents under any framing. Verification is yours.
