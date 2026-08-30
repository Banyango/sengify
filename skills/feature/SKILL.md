---
name: feature
description: Grill the user to define a feature and its use cases through a relentless round-based interview, writing the result to the wiki
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

### 3. Work the Design Tree in Rounds

The **frontier** is every decision whose prerequisites are already settled — the questions you can ask *now* without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body — may include multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body>

➡️ <your recommended answer>
```

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a *later* round, not this one.

Finding *facts* is your job, never the user's. When a frontier question needs a fact from the environment (the codebase, the wiki, existing docs), look it up yourself or dispatch a sub-agent — don't ask the user for anything you could look up. Don't block the round on it: only the questions downstream of a lookup wait; ask the rest of the frontier now. The *decisions* are the user's — put each to them and wait.

Cover at minimum, across however many rounds it takes:

- **Scope**: what's explicitly in scope, what's explicitly out
- **Actors**: who initiates and who benefits
- **Trigger**: what starts this feature's flow
- **Use cases**: the concrete scenarios that must work, stated as `As a <actor>, I want <capability>, so that <benefit>`
- **Constraints**: technical, business, or design constraints that shape it
- **Edge cases**: what happens when things go wrong or inputs are unexpected

### 4. Terminate and Confirm

The session is done when the frontier is empty — every branch visited, nothing silently assumed. Summarize the feature description and full list of use cases, and ask the user to confirm before writing anything.

### 5. Write the Feature Page

On confirmation, create `.sengify/wiki/features/<slug>/feature.md` (slug = kebab-case of the feature name):

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

Update `.sengify/wiki/index.md` to link the new feature page under a `Features` section.

## Rules

- Do not write the feature page until the user has confirmed the summary
- Do not implement anything or touch source code
- One round of questions at a time — never dump the whole tree at once
- Never ask the user for a fact you could find yourself
