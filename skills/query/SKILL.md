---
name: query
description: Ask a question against the project wiki and get a synthesized, cited answer
disable-model-invocation: true
argument-hint: <question>
---

# Sengify Query

Answer a question using only the wiki — no live codebase exploration, no external search. Never edit the wiki except via the explicit save-as-page step.

## Steps

### 1. Determine the Question

`$ARGUMENTS` is the question. If empty, ask the user for one.

### 2. Load the Wiki

Read `.sengify/wiki/index.md` to see what pages exist. If `.sengify/wiki/` doesn't exist yet, tell the user there's no wiki to query and stop — suggest running `/sengify:ingest` first.

### 3. Find Relevant Pages

Read every page that looks relevant to the question (overview, entity pages, feature pages). Follow `[[cross-references]]` one level deep if a linked page looks relevant too.

### 4. Synthesize an Answer

Write a direct answer to the question, then list the pages it draws from:

```markdown
**Answer:** <answer>

**Sources:** [[entity-a]], [[entity-b]], features/checkout/feature.md
```

If the wiki doesn't contain enough information to answer, say so plainly — do not guess or fill gaps with assumptions.

### 5. Offer to Save

If the answer is substantial enough to be reused later (not a one-line fact lookup), ask the user: "Save this as a wiki page?" If yes, ask where it belongs (a new page under `entities/`, a new `concepts/<slug>.md`, or appended to an existing page) and write it there, citing this query as the source. If no, or if the answer was trivial, don't write anything.

## Rules

- Read-only against the wiki unless the user explicitly approves saving a new page
- Never search the codebase or the web — answer strictly from wiki content
- Always cite which pages the answer came from
- Say "I don't know" rather than guessing when the wiki lacks the answer
