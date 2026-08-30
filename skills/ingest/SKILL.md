---
name: ingest
description: Read an external source document and update the project wiki with project description, entities, and cross-references
disable-model-invocation: true
argument-hint: <path or URL to the source document>
---

# Sengify Ingest

Read a single external source and fold what it teaches into the wiki. The source is read-only — never modify it.

## Steps

### 1. Locate the Wiki

Check whether `.sengify/wiki/` exists. If not, create it:

- `.sengify/wiki/index.md` from `skills/ingest/templates/index.template.md`
- `.sengify/wiki/overview.md` from `skills/ingest/templates/overview.template.md`
- `.sengify/wiki/log.md` from `skills/ingest/templates/log.template.md`
- `.sengify/wiki/entities/` (empty directory, populated in step 4)
- `.sengify/wiki/features/` (empty directory, populated by the `feature` skill)

### 2. Read the Source

`$ARGUMENTS` is the path or URL to ingest. If empty, ask the user for one before proceeding. Read it in full.

### 3. Extract Project Description

Identify statements about what the project is, who it's for, and what problem it solves. Merge these into `.sengify/wiki/overview.md`:

- If `overview.md` still has only the template placeholder, replace it with the description directly.
- If it already has real content, integrate new information without duplicating existing statements. Flag (don't silently drop) any contradiction between the new source and existing overview text — surface it to the user instead of guessing which is right.

Cite the source at the bottom of any paragraph drawn from it: `_(source: <path-or-url>)_`.

### 4. Extract Entities

Identify domain entities (nouns with attributes/behavior/relationships — e.g. "User", "Order", "Subscription"). For each:

- If `.sengify/wiki/entities/<entity>.md` does not exist, create it using `skills/ingest/templates/entity.template.md` as the starting shape, filled in.
- If it exists, add new attributes/relationships/citations; flag contradictions with existing content the same way as step 3.

Entity filenames are lowercase, hyphenated (`payment-method.md`, not `PaymentMethod.md`).

### 5. Cross-Reference

Any entity mentioned inside `overview.md` or another entity page gets wrapped as `[[entity-name]]` (matching the entity's filename without `.md`). Do this for both new and pre-existing mentions you touch in this run.

### 6. Update the Index

Update `.sengify/wiki/index.md` — a flat list of links to every page under `wiki/`, grouped by `Overview`, `Entities`, `Features`.

### 7. Log the Operation

Append one line to `.sengify/wiki/log.md`:

```markdown
- YYYY-MM-DD HH:MM — ingest — source: <path-or-url> — pages touched: overview.md, entities/user.md, ...
```

## Rules

- Never modify the source document
- Never delete existing wiki content — only add, merge, or flag contradictions
- Always cite sources on new claims
- Do not invent entities or facts not present in the source
