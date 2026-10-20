---
name: side-quest
description: Do a single task (a bead or a free-form prompt) in its own throwaway worktree based on main, commit it to a local feature branch, remove the worktree but keep the branch, and hand back the git switch command. Use when the user runs /side-quest <bead-id or prompt>.
argument-hint: <bead-id | prompt>
disable-model-invocation: true
---

# Side quest

Input: the skill argument, which is either a `<bead-id>` or a free-form prompt (e.g. "update documentation to mention new tools").

## Step 0: require an argument

If the argument is empty, STOP and ask for a bead id or a prompt. Do nothing else until you have one.

## Step 1: work out what the argument is

Do not strictly detect bead ids. If the argument is a single token that looks like a bead id, try `bd show <argument>`. If it resolves, treat it as a bead and use its title and description as the task. Otherwise (no match, or it is clearly a sentence), treat the whole argument as the prompt.

## Prompt

Implement `<bead-id or prompt>` in a new worktree based on `main`. When done, commit it to a local branch named from the task, with no push. Remove the worktree directory but keep the branch, and tell the user the `git switch` command.

## Branch name

- Bead: `feature/<bead-id>-<short-title-slug>`
- Prompt: `feature/<short-slug>`, where the slug is a few lowercase hyphenated words summarising the prompt.

## Notes on carrying it out

- Create the worktree from `main` on the new branch (e.g. `git worktree add <path> -b <branch> main`), and do all the work inside it.
- Commit on that branch. Never push.
- When done, remove the worktree directory (`git worktree remove <path>`) but keep the branch.
- Finish by telling the user: `git switch <branch>`.
