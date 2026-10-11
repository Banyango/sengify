---
name: taskify
description: Break a feature page into a high-level, dependency-ordered task breakdown for scalpel to plan file-by-file
disable-model-invocation: true
argument-hint: <feature slug or numeral, matching .sengify/wiki/features/<NNN>-<slug>/>
---

# Sengify Taskify

Turn a completed feature page into a high-level task breakdown. This is the last sengify step — the output hands off to scalpel (`/scalpel:plan <task description>`), run manually per task afterward.

## Steps

### 1. Locate the Feature

`$ARGUMENTS` identifies the feature — either its numeral (`003`), its slug (`checkout-retry`), or the full directory name (`003-checkout-retry`). Match it against the directories under `.sengify/wiki/features/<NNN>-<slug>/` and read that feature's `feature.md`. If nothing matches, list the available `<NNN>-<slug>` directories and ask the user to pick one, or suggest running `/sengify:feature` first if none exist. Keep the resolved `<NNN>` — every task in this plan inherits it.

### 2. Gather Context

Read every entity page linked from the feature page's `Related Entities`. Read the current codebase structure (directories, existing modules) enough to know what already exists versus what's new — this and `feature`'s fact-finding are the only points where sengify reads code.

### 3. Draft the Task Breakdown

Break the feature's use cases into **tasks**: coarse, high-level units of work — not file-level detail (that's scalpel's job) and not tickets with acceptance criteria. Each task should be describable in one or two sentences and cut a complete path through whatever layers it touches.

Each task's identifier is `<NNN>-<task#>`, where `<NNN>` is the feature's numeral (resolved in step 1) and `<task#>` is the task's position within this plan (1, 2, 3, ...) — e.g. the second task of feature `003` is `003-2`.

For each task, determine its **blocked by** list — the other tasks that must land first, referenced by their full `<NNN>-<task#>` id. A task with no blockers can start immediately.

### 4. Present for Approval

Show the user the proposed breakdown as a numbered list, using each task's full id:

```
003-1. <Task title> — blocked by: none
   <one-line description>
003-2. <Task title> — blocked by: 003-1
   <one-line description>
```

Ask whether the granularity and blocking edges look right. Iterate until approved. Do not write the file until approved.

### 5. Write the Task Plan

Write `.sengify/wiki/features/<slug>/tasks.md`:

```markdown
# <Feature Name> — Task Plan

Source feature: [feature.md](./feature.md)

## Tasks

### 003-1. <Task title>

**Blocked by:** none

<description — the end-to-end behavior this task delivers, from the use cases it covers>

### 003-2. <Task title>

**Blocked by:** 003-1

<description>
```

(substitute the feature's actual numeral for `003` throughout)

### 6. Report Next Step

Tell the user the task plan is ready and that each task can now be handed to scalpel by its full id: `/scalpel:plan <NNN>-<task#>: <task description>`.

## Rules

- Do not write file paths, code, or acceptance criteria — that's scalpel's job downstream
- Do not write the task plan until the user approves the breakdown
- Do not invoke scalpel yourself — the handoff is manual
- Tasks are high-level and vertical (cut through all layers a use case touches), not layer-by-layer
