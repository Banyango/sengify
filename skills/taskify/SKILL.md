---
name: taskify
description: Break a feature page into a high-level, dependency-ordered task breakdown for scalpel to plan file-by-file
disable-model-invocation: true
argument-hint: <feature slug, matching .sengify/wiki/features/<slug>/>
---

# Sengify Taskify

Turn a completed feature page into a high-level task breakdown. This is the last sengify step — the output hands off to scalpel (`/scalpel:plan <task description>`), run manually per task afterward.

## Steps

### 1. Locate the Feature

`$ARGUMENTS` is the feature slug. Read `.sengify/wiki/features/<slug>/feature.md`. If it doesn't exist, list the available slugs under `.sengify/wiki/features/` and ask the user to pick one, or suggest running `/sengify:feature` first if none exist.

### 2. Gather Context

Read every entity page linked from the feature page's `Related Entities`. Read the current codebase structure (directories, existing modules) enough to know what already exists versus what's new — this is the one place sengify looks at code.

### 3. Draft the Task Breakdown

Break the feature's use cases into **tasks**: coarse, high-level units of work — not file-level detail (that's scalpel's job) and not tickets with acceptance criteria. Each task should be describable in one or two sentences and cut a complete path through whatever layers it touches.

For each task, determine its **blocked by** list — the other tasks that must land first. A task with no blockers can start immediately.

### 4. Present for Approval

Show the user the proposed breakdown as a numbered list:

```
1. <Task title> — blocked by: none
   <one-line description>
2. <Task title> — blocked by: 1
   <one-line description>
```

Ask whether the granularity and blocking edges look right. Iterate until approved. Do not write the file until approved.

### 5. Write the Task Plan

Write `.sengify/wiki/features/<slug>/tasks.md`:

```markdown
# <Feature Name> — Task Plan

Source feature: [feature.md](./feature.md)

## Tasks

### 1. <Task title>

**Blocked by:** none

<description — the end-to-end behavior this task delivers, from the use cases it covers>

### 2. <Task title>

**Blocked by:** 1

<description>
```

### 6. Report Next Step

Tell the user the task plan is ready and that each task can now be handed to scalpel: `/scalpel:plan <task description>`.

## Rules

- Do not write file paths, code, or acceptance criteria — that's scalpel's job downstream
- Do not write the task plan until the user approves the breakdown
- Do not invoke scalpel yourself — the handoff is manual
- Tasks are high-level and vertical (cut through all layers a use case touches), not layer-by-layer
