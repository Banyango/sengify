---
name: taskify
description: Break a feature page into a high-level task breakdown with dependencies within and across features, for scalpel to plan file-by-file
argument-hint: <feature slug or numeral, matching .sengify/wiki/features/<NNN>-<slug>/>
---

# Sengify Taskify

Turn a completed feature page into a high-level task breakdown. This is the last sengify step — the output hands off to scalpel (`/scalpel:plan <task description>`), run manually per task afterward.

## Steps

### 1. Locate the Feature

`$ARGUMENTS` identifies the feature — either its numeral (`003`), its slug (`checkout-retry`), or the full directory name (`003-checkout-retry`). Match it against the directories under `.sengify/wiki/features/<NNN>-<slug>/` and read that feature's `feature.md`. If nothing matches, list the available `<NNN>-<slug>` directories and ask the user to pick one, or suggest running `/sengify:feature` first if none exist. Keep the resolved `<NNN>` — every task in this plan inherits it.

### 2. Gather Context

Read every entity page linked from the feature page's `Related Entities`. Read the current codebase structure (directories, existing modules) enough to know what already exists versus what's new — this and `feature`'s fact-finding are the only points where sengify reads code.

Scan the other `features/<NNN>-<slug>/feature.md` pages for capabilities this feature needs, including features it does not explicitly link. Read relevant features' `tasks.md` files when present to identify the specific tasks that deliver those prerequisites. Shared entities or similar names alone do not establish a dependency; identify the behavior, data, or interface the current feature requires.

### 3. Draft the Task Breakdown

Break the feature's use cases into **tasks**: coarse, high-level units of work — not file-level detail (that's scalpel's job) and not tickets with acceptance criteria. Each task should be describable in one or two sentences and cut a complete path through whatever layers it touches.

Each task's identifier is `<NNN>-<task#>`, where `<NNN>` is the feature's numeral (resolved in step 1) and `<task#>` is the task's position within this plan (1, 2, 3, ...) — e.g. the second task of feature `003` is `003-2`.

For each task, determine its **blocked by** list — the other tasks that must land first, within this feature or in another feature. Reference existing tasks by their full `<NNN>-<task#>` id; link external tasks to their heading in the other feature's `tasks.md` and explain what prerequisite they provide. Use the specific prerequisite tasks rather than blocking on an entire feature when only part of it is required.

If a required capability belongs to a feature without a task plan, link its `feature.md`, label the blocker `unplanned feature <NNN>`, and describe the capability needed. Do not invent task ids or treat a missing task plan as no dependency. If implementation status is unclear, flag it for confirmation during approval; an existing plan does not prove its tasks have landed. Capabilities confirmed as already implemented are context, not outstanding blockers.

Order the local tasks by their dependencies and check for cycles, following relevant external blocker chains in existing plans. Surface cycles or unresolved prerequisites for the user to resolve before approval. A task can start immediately only when it has no outstanding blockers, including external or unplanned prerequisites.

### 4. Present for Approval

Show the user the proposed breakdown as a numbered list, using each task's full id:

```
003-1. <Task title> — blocked by: none
   <one-line description>
003-2. <Task title> — blocked by: 003-1
   <one-line description>
003-3. <Task title> — blocked by: [002-2](../002-accounts/tasks.md#002-2-create-accounts)
   <one-line description; explain the capability supplied by 002-2>
003-4. <Task title> — blocked by: [unplanned feature 004](../004-notifications/feature.md)
   <one-line description; identify the required notification capability>
```

Include the reasons and source links for cross-feature dependencies, and call out unplanned prerequisites, uncertain implementation status, or cycles. Ask whether the granularity and blocking edges look right. Iterate until approved. Do not write the file until approved.

### 5. Write the Task Plan

Write `.sengify/wiki/features/<NNN>-<slug>/tasks.md`:

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

### 003-3. <Task title>

**Blocked by:** [002-2](../002-accounts/tasks.md#002-2-create-accounts)

**Dependency reason:** <the capability 002-2 must deliver before this task can land>

<description>

### 003-4. <Task title>

**Blocked by:** [unplanned feature 004](../004-notifications/feature.md)

**Dependency reason:** <required capability; feature 004 needs a task plan before this blocker can be narrowed to task ids>

<description>
```

(substitute the actual feature numerals, directories, task ids, and heading anchors throughout; include external and unplanned blockers only when found)

### 6. Report Next Step

Tell the user the task plan is ready and that each task can be handed to scalpel by its full id: `/scalpel:plan <NNN>-<task#>: <task description>`. Highlight outstanding cross-feature blockers and any prerequisite features that need taskifying before dependent work can proceed.

## Rules

- Do not write implementation file paths, code, or acceptance criteria — that's scalpel's job downstream; wiki source and dependency links belong in the plan
- Do not create or modify other features' task plans as part of this breakdown
- Do not write the task plan until the user approves the breakdown
- Do not invoke scalpel yourself — the handoff is manual
- Tasks are high-level and vertical (cut through all layers a use case touches), not layer-by-layer
