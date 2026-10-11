---
name: feature
description: Grill the user to define a feature and its use cases through a relentless one-question-at-a-time interview, writing the result to the wiki
disable-model-invocation: true
argument-hint: <short description of the feature idea>
---

# Sengify Feature

Interview the user relentlessly about the feature in `$ARGUMENTS` until you reach a shared understanding. Map it as a **design tree**: every decision branches into the decisions that hang off it.

## Steps

### 1. Determine the Starting Point

`$ARGUMENTS` is the initial feature idea. If empty, ask the user to describe it before proceeding.

### 2. Check the Wiki for Context

Read `.sengify/wiki/overview.md` and any `entities/*.md` pages that look relevant to the feature idea — use their vocabulary and don't ask the user to redefine terms the wiki already answers. If `.sengify/wiki/` doesn't exist, proceed without wiki context.

### 3. Work the Design Tree One Question at a Time

The **frontier** is every decision whose prerequisites are already settled — the questions you can ask *now* without guessing at answers you haven't heard yet. Ask exactly one frontier question at a time, using the `AskUserQuestion` tool — not a formatted text block. Give it a clear question, a short header, and 2-4 mutually exclusive options where the choice is discrete; put your recommended option first and mark it "(Recommended)". For open-ended questions (use cases, free-text scope), ask as plain text via `AskUserQuestion`'s free-form option rather than forcing a multiple-choice shape.

After each answer, recompute the frontier — the answer may settle a decision, which pushes the frontier outward and unblocks questions that depended on it — then ask the single next frontier question. Never batch multiple questions into one `AskUserQuestion` call, and never dump a round of questions as text.

Finding *facts* is your job, never the user's. When a frontier question needs a fact from the environment (the codebase, the wiki, existing docs), look it up yourself or dispatch a sub-agent — don't ask the user for anything you could look up. A lookup only delays the question it feeds; ask whatever else is already on the frontier first. The *decisions* are the user's — put each to them, one at a time, and wait.

Cover at minimum, across however many questions it takes:

- **Scope**: what's explicitly in scope, what's explicitly out
- **Actors**: who initiates and who benefits
- **Trigger**: what starts this feature's flow
- **Use cases**: the concrete scenarios that must work, stated as `As a <actor>, I want <capability>, so that <benefit>`
- **Constraints**: technical, business, or design constraints that shape it
- **Edge cases**: what happens when things go wrong or inputs are unexpected

### 4. Terminate and Confirm

The session is done when the frontier is empty — every branch visited, nothing silently assumed. Summarize the feature description and full list of use cases, and ask the user to confirm before writing anything.

### 5. Write the Feature Page

On confirmation, determine the next feature number: list existing directories under `.sengify/wiki/features/`, take the highest `<NNN>-` prefix found (zero-padded, 3 digits), and increment it — start at `001` if none exist. Create `.sengify/wiki/features/<NNN>-<slug>/feature.md` (slug = kebab-case of the feature name), e.g. `.sengify/wiki/features/003-checkout-retry/feature.md`:

```markdown
# <Feature Name>

## Description

<the settled feature description>

## Use Cases

1. As a <actor>, I want <capability>, so that <benefit>.
2. ...

## Constraints

- <constraint>

## Edge Cases

- <edge case> → <expected behavior>

## Related Entities

- [[<entity>]]
```

Update `.sengify/wiki/index.md` to link the new feature page under a `Features` section, listing it as `<NNN> - <Feature Name>` so the numeral stays visible as a stable reference.

## Rules

- Do not write the feature page until the user has confirmed the summary
- Do not implement anything or touch source code
- Ask one question at a time via `AskUserQuestion` — never dump the whole tree, or even a round of it, at once
- Never ask the user for a fact you could find yourself
