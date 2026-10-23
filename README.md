# md (Maikel's skills)

A Claude Code plugin bundling my personal skills for planning, bead-driven implementation, and staying focused.

## Install

```
/plugin marketplace add maikeldaloo/skills
/plugin install md@md
```

Skills are namespaced by the plugin name, so they are invoked as `/md:<skill>` (e.g. `/md:grill-me`).

### Updates

The plugin deliberately has **no `version`** in `plugin.json`, so Claude Code uses the git commit SHA as the version: every commit pushed to this repo is a new version.

- Pushing is not enough on its own. Claude Code only checks the marketplace on startup **if auto-update is enabled** for it. Third-party marketplaces have auto-update **off** by default; turn it on via `/plugin` → Marketplaces → `md` → Enable auto-update.
- Or update manually any time: `/plugin marketplace update md`, then restart / `/reload-plugins`.
- If you ever add a `version` field to `plugin.json`, updates only arrive when you bump it.

## Repo layout

```
.claude-plugin/plugin.json        plugin manifest
.claude-plugin/marketplace.json   makes this repo installable as a marketplace
skills/<name>/SKILL.md            one folder per skill
```

To add a skill: create `skills/<name>/SKILL.md` with `name` and `description` frontmatter, commit, push.

## Beads prerequisite

Four skills (`beads-decompose-plan`, `beads-implement`, `beads-summary-and-test`, and optionally `side-quest`) work with [beads](https://github.com/steveyegge/beads), the `bd` CLI issue tracker. **These skills do not install or initialise beads.** They assume `bd` is installed and the repo has already been set up (`bd init`). If it is not, the skills will simply fail to find beads, so set it up first.

## Skills

| Skill | Needs beads | Auto-triggers |
|---|---|---|
| `grill-me` | no | yes |
| `plan-digest` | no | yes |
| `beads-decompose-plan` | yes | yes |
| `beads-implement` | yes | yes |
| `beads-summary-and-test` | yes | yes |
| `side-quest` | optional | no (manual only) |

### grill-me
Interviews you one question at a time about a plan or design until you reach shared understanding. Shows progress ("Question 3 of ~10"), recommends an answer for each, and hands back the decisions made plus questions for other people.
```
/md:grill-me  I want to move our checkout to a server-rendered flow
```

### plan-digest
Compresses a plan into a under-a-minute digest: outcome, blast radius, locked-in decisions, risks, verification, go/no-go. Usually run right after `grill-me`.
```
/md:plan-digest the plan we just built
```

### beads-decompose-plan
Turns a plan file into a bead graph: one epic, sequential sub-epics (execution waves), and parallel beads inside each wave, labelled with agent, effort and task kind. Requires a path to the plan file.
```
/md:beads-decompose-plan docs/checkout-plan.md
```

### beads-implement
Runs one sub-epic: fans its beads out to parallel sub-agents (one per bead, model chosen from the bead's label), then verifies results itself. Requires a bead ID; run sub-epics in order.
```
/md:beads-implement proj-12.1
/md:beads-implement proj-12.2 - use a workflow
```

### beads-summary-and-test
After implementation, summarises what an epic did, checks it against the actual codebase, then walks you through manual testing one step (with lettered checks) at a time. Requires an epic ID.
```
/md:beads-summary-and-test proj-12
```

### side-quest
Does one task in a throwaway worktree off `main`, commits to a local `feature/...` branch (never pushes), removes the worktree, and gives you the `git switch` command. Argument is a bead ID or a free-form prompt.
```
/md:side-quest proj-45
/md:side-quest update the docs to mention the new tools
```

## Typical workflow

```
grill-me → plan-digest → (save plan) → beads-decompose-plan
        → beads-implement <epic>.1 → <epic>.2 → ... → beads-summary-and-test <epic>
```
