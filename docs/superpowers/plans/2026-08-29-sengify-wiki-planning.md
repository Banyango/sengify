# sengify LLM-Wiki Feature Planning Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build sengify, a scalpel-structured, multi-provider agent skills plugin implementing an LLM-wiki (ingest/query/lint) workflow plus feature-grilling and taskify skills that hand off to scalpel.

**Architecture:** A pure markdown+JSON skills plugin — no runtime code. Five `skills/<name>/SKILL.md` files drive the workflow; provider plugin manifests (`.claude-plugin/`, `.codex-plugin/`, `.cursor-plugin/`, `.kimi-plugin/`, `.opencode/`) register them; root `CLAUDE.md`/`AGENTS.md`/`GEMINI.md` orient agents working in this repo; `README.md` documents install and workflow.

**Tech Stack:** None (markdown + JSON only, mirroring scalpel's own structure — its `tsconfig.json`/build script are vestigial and unused, so this plan does not add real TypeScript either).

**Spec:** `docs/superpowers/specs/2026-08-29-sengify-wiki-planning-design.md`

## Global Constraints

- Mirror scalpel's plugin structure and provider manifests, renamed to `sengify` (spec: "Plugin Structure")
- Author: Kyle Reczek, `kyle@banyango.com`, license MIT (spec: "Plugin Structure")
- No entry-point skill (`using-sengify` is deliberately omitted) — each `SKILL.md` is self-contained via its own frontmatter (spec: "Plugin Structure")
- `ingest` only reads external documents, never the codebase; `taskify` is the only skill that reads the codebase (spec: "Out of Scope")
- `lint` and `query` never auto-fix or silently edit — report/prompt only (spec: "query", "lint")
- `taskify` writes a task-plan doc; it never invokes scalpel itself — handoff is manual (spec: "taskify", "Relationship to scalpel")
- All five skills use `disable-model-invocation: true` (slash-command only), matching scalpel's convention

Since this is a markdown/JSON content plugin with no test framework, "testable deliverable" per task means: JSON validates (`jq .`), required frontmatter fields are present, and the file's content is reviewed against the spec section it implements. There is no `pytest`/`jest` equivalent — do not invent one.

---

### Task 1: Project Scaffold

**Files:**
- Modify: `package.json` (replace placeholder scaffold)
- Create: `tsconfig.json`
- Create: `.gitignore`
- Delete: `index.js` (unrelated `npm init` stub, not part of the plugin)

**Interfaces:** None — no other task depends on this task's content beyond the repo existing.

- [ ] **Step 1: Replace `package.json`**

```json
{
  "name": "sengify",
  "version": "0.1.0",
  "description": "LLM-wiki feature planning skills: ingest docs into a persistent wiki, grill features into use cases, and taskify them into a plan scalpel can implement",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc"
  },
  "skills": "./skills/",
  "keywords": [
    "skills",
    "agent",
    "claude",
    "codex",
    "cursor",
    "gemini",
    "wiki",
    "planning"
  ],
  "author": {
    "name": "Kyle Reczek",
    "email": "kyle@banyango.com"
  },
  "license": "MIT",
  "homepage": "https://github.com/Banyango/sengify",
  "repository": "https://github.com/Banyango/sengify",
  "dependencies": {},
  "devDependencies": {
    "typescript": "^5.5.3"
  },
  "private": false
}
```

- [ ] **Step 2: Create `tsconfig.json`**

```json
{
  "compilerOptions": {
    "target": "es2016",
    "module": "commonjs",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "outDir": "dist"
  },
  "include": ["src"]
}
```

- [ ] **Step 3: Create `.gitignore`**

```
/tmp
/out-tsc
/dist

/node_modules
npm-debug.log*
yarn-debug.log*
yarn-error.log*
/.pnp
.pnp.js

.vscode/*
.DS_Store

*.local.md

.idea/
```

- [ ] **Step 4: Delete `index.js`**

```bash
rm index.js
```

- [ ] **Step 5: Verify JSON validity**

Run: `jq . package.json && jq . tsconfig.json`
Expected: both print the parsed JSON back with no error.

- [ ] **Step 6: Commit**

```bash
git add package.json tsconfig.json .gitignore
git rm index.js
git commit -m "Scaffold sengify plugin project files"
```

---

### Task 2: Claude Code Plugin Manifests

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`

**Interfaces:** None.

- [ ] **Step 1: Create `.claude-plugin/plugin.json`**

```json
{
  "name": "sengify",
  "description": "LLM-wiki feature planning: ingest docs into a persistent project wiki, grill features into use cases, and taskify them into a plan ready for scalpel",
  "version": "0.1.0",
  "author": {
    "name": "Kyle Reczek",
    "email": "kyle@banyango.com"
  },
  "homepage": "https://github.com/Banyango/sengify",
  "repository": "https://github.com/Banyango/sengify",
  "license": "MIT",
  "keywords": [
    "skills",
    "workflows",
    "wiki",
    "planning"
  ],
  "skills": "./skills/"
}
```

- [ ] **Step 2: Create `.claude-plugin/marketplace.json`**

```json
{
  "name": "sengify-dev",
  "description": "Development marketplace for the sengify skills plugin",
  "owner": {
    "name": "Kyle Reczek",
    "email": "kyle@banyango.com"
  },
  "plugins": [
    {
      "name": "sengify",
      "description": "LLM-wiki feature planning: ingest docs into a persistent project wiki, grill features into use cases, and taskify them into a plan ready for scalpel",
      "version": "0.1.0",
      "source": "./",
      "author": {
        "name": "Kyle Reczek",
        "email": "kyle@banyango.com"
      }
    }
  ]
}
```

- [ ] **Step 3: Verify JSON validity**

Run: `jq . .claude-plugin/plugin.json && jq . .claude-plugin/marketplace.json`
Expected: both print the parsed JSON back with no error.

- [ ] **Step 4: Commit**

```bash
git add .claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "Add Claude Code plugin manifests"
```

---

### Task 3: Other Provider Plugin Manifests

**Files:**
- Create: `.codex-plugin/plugin.json`
- Create: `.cursor-plugin/plugin.json`
- Create: `.kimi-plugin/plugin.json`
- Create: `.opencode/INSTALL.md`

**Interfaces:** None.

Note: scalpel's `.codex-plugin/plugin.json` and `.cursor-plugin/plugin.json` reference `"hooks": "./hooks/hooks-codex.json"` / `"./hooks/hooks-cursor.json"`, but no `hooks/` directory exists anywhere in the scalpel repo — those are dangling references. Do not replicate them.

- [ ] **Step 1: Create `.codex-plugin/plugin.json`**

```json
{
  "name": "sengify",
  "version": "0.1.0",
  "description": "LLM-wiki feature planning: ingest docs into a persistent project wiki, grill features into use cases, and taskify them into a plan ready for scalpel",
  "author": {
    "name": "Kyle Reczek",
    "email": "kyle@banyango.com",
    "url": "https://github.com/Banyango"
  },
  "homepage": "https://github.com/Banyango/sengify",
  "repository": "https://github.com/Banyango/sengify",
  "license": "MIT",
  "keywords": [
    "skills",
    "workflows",
    "wiki",
    "planning"
  ],
  "skills": "./skills/",
  "interface": {
    "displayName": "sengify",
    "shortDescription": "LLM-wiki feature planning for AI-assisted development",
    "longDescription": "sengify builds a persistent, queryable project wiki from your product docs, then grills feature ideas into concrete use cases and a high-level task plan — ready to hand to scalpel for file-level implementation planning.",
    "developerName": "Kyle Reczek",
    "category": "Coding",
    "capabilities": [
      "Interactive",
      "Read",
      "Write"
    ],
    "defaultPrompt": [
      "I want to ingest a design doc into the project wiki.",
      "Help me grill this feature idea into use cases."
    ],
    "websiteURL": "https://github.com/Banyango/sengify",
    "brandColor": "#3B82F6"
  }
}
```

- [ ] **Step 2: Create `.cursor-plugin/plugin.json`**

```json
{
  "name": "sengify",
  "displayName": "sengify",
  "description": "LLM-wiki feature planning: ingest docs into a persistent project wiki, grill features into use cases, and taskify them into a plan ready for scalpel",
  "version": "0.1.0",
  "author": {
    "name": "Kyle Reczek",
    "email": "kyle@banyango.com"
  },
  "homepage": "https://github.com/Banyango/sengify",
  "repository": "https://github.com/Banyango/sengify",
  "license": "MIT",
  "keywords": [
    "skills",
    "workflows",
    "wiki",
    "planning"
  ],
  "skills": "./skills/"
}
```

- [ ] **Step 3: Create `.kimi-plugin/plugin.json`**

```json
{
  "name": "sengify",
  "version": "0.1.0",
  "description": "LLM-wiki feature planning: ingest docs into a persistent project wiki, grill features into use cases, and taskify them into a plan ready for scalpel",
  "author": {
    "name": "Kyle Reczek",
    "email": "kyle@banyango.com"
  },
  "homepage": "https://github.com/Banyango/sengify",
  "license": "MIT",
  "keywords": [
    "skills",
    "workflows",
    "wiki",
    "planning"
  ],
  "skills": "./skills/",
  "skillInstructions": "Kimi Code tool mapping for sengify skills:\n\n- When a sengify skill says to ask the user, call Kimi Code's `AskUserQuestion` tool with 1 question and 2-4 concrete options. Put the recommended option first with `(Recommended)` suffix.\n- When a skill refers to `TodoWrite` or task tracking, use Kimi Code's `TodoList` tool.\n- When a skill says to dispatch a sub-agent, use Kimi Code's `Agent` tool with `subagent_type: \"explore\"` for read-only lookups.\n- When a skill refers to the `Skill` tool, use Kimi Code's native `Skill` tool.\n- Use Kimi Code's `Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, `FetchURL`, and `WebSearch` tools by their actual exposed names.",
  "interface": {
    "displayName": "sengify",
    "shortDescription": "LLM-wiki feature planning for AI-assisted development",
    "developerName": "Kyle Reczek",
    "capabilities": [
      "Interactive",
      "Read",
      "Write"
    ],
    "websiteURL": "https://github.com/Banyango/sengify"
  }
}
```

- [ ] **Step 4: Create `.opencode/INSTALL.md`**

```markdown
# Installing sengify for OpenCode

## Prerequisites

- [OpenCode.ai](https://opencode.ai) installed

## Installation

Add sengify to the `plugin` array in your `opencode.json` (global or project-level):

​```json
{
  "plugin": ["sengify@git+https://github.com/Banyango/sengify.git"]
}
​```

Restart OpenCode. The plugin installs through OpenCode's plugin manager and registers all skills.

Verify by asking: "Tell me about your sengify skills"

## Usage

Use OpenCode's native `skill` tool:

​```
use skill tool to list skills
use skill tool to load ingest
​```

## Tool Mapping

Skills speak in actions ("create a todo", "dispatch a subagent", "read a file"). On OpenCode these resolve to:

- "Create a todo" / "mark complete in todo list" → `todowrite`
- Subagent dispatch → `task` tool with appropriate `subagent_type`
- "Invoke a skill" → OpenCode's native `skill` tool
- "Read a file" → `read`
- "Create/edit/delete a file" → `apply_patch`
- "Run a shell command" → `bash`
- "Search file contents" → `grep`
- "Find files by name" → `glob`
- "Fetch a URL" → `webfetch`

## Pinning a Specific Version

​```json
{
  "plugin": ["sengify@git+https://github.com/Banyango/sengify.git#v0.1.0"]
}
​```

## Getting Help

Report issues: https://github.com/Banyango/sengify/issues
```

- [ ] **Step 5: Verify JSON validity**

Run: `jq . .codex-plugin/plugin.json && jq . .cursor-plugin/plugin.json && jq . .kimi-plugin/plugin.json`
Expected: all three print the parsed JSON back with no error.

- [ ] **Step 6: Commit**

```bash
git add .codex-plugin/plugin.json .cursor-plugin/plugin.json .kimi-plugin/plugin.json .opencode/INSTALL.md
git commit -m "Add Codex, Cursor, Kimi, and OpenCode plugin manifests"
```

---

### Task 4: Root Agent Docs (CLAUDE.md / AGENTS.md / GEMINI.md)

**Files:**
- Create: `CLAUDE.md`
- Create: `AGENTS.md`
- Create: `GEMINI.md`

**Interfaces:** None. These reference the five skills by name (`ingest`, `query`, `lint`, `feature`, `taskify`) — no code interface.

Unlike scalpel's current `CLAUDE.md`/`AGENTS.md`/`GEMINI.md` (which still `@`-import a `skills/using-scalpel/` path that no longer exists in that repo — a leftover bug from when that skill was removed), sengify never had an entry skill, so these files carry their own short body directly, with no dangling `@`-import.

- [ ] **Step 1: Create `CLAUDE.md`**

```markdown
## sengify Plugin — Claude Code

This project is the sengify skills plugin itself. When working in this repo:

- Skills live in `skills/<name>/SKILL.md`
- Plugin manifests live in `.<provider>-plugin/plugin.json`
- Five skills drive the workflow: `ingest`, `query`, `lint`, `feature`, `taskify` — see `README.md` for how they fit together

When adding a new skill, create `skills/<name>/SKILL.md` with `name` and `description` frontmatter. Update the Commands table in `README.md`.
```

- [ ] **Step 2: Create `AGENTS.md`**

```markdown
## sengify Plugin — Codex / Copilot CLI

This project is the sengify skills plugin itself. When working in this repo:

- Skills live in `skills/<name>/SKILL.md`
- Plugin manifests live in `.<provider>-plugin/plugin.json`
- Five skills drive the workflow: `ingest`, `query`, `lint`, `feature`, `taskify` — see `README.md` for how they fit together

When adding a new skill, create `skills/<name>/SKILL.md` with `name` and `description` frontmatter. Update the Commands table in `README.md`.
```

- [ ] **Step 3: Create `GEMINI.md`**

```markdown
## sengify Plugin — Gemini CLI

This project is the sengify skills plugin itself. When working in this repo:

- Skills live in `skills/<name>/SKILL.md`
- Plugin manifests live in `.<provider>-plugin/plugin.json`
- Five skills drive the workflow: `ingest`, `query`, `lint`, `feature`, `taskify` — see `README.md` for how they fit together

When adding a new skill, create `skills/<name>/SKILL.md` with `name` and `description` frontmatter. Update the Commands table in `README.md`.
```

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md AGENTS.md GEMINI.md
git commit -m "Add root agent orientation docs"
```

---

### Task 5: `ingest` Skill

**Files:**
- Create: `skills/ingest/SKILL.md`
- Create: `skills/ingest/templates/index.template.md`
- Create: `skills/ingest/templates/overview.template.md`
- Create: `skills/ingest/templates/log.template.md`
- Create: `skills/ingest/templates/entity.template.md`

**Interfaces:**
- Produces: `.sengify/wiki/index.md`, `.sengify/wiki/overview.md`, `.sengify/wiki/entities/<entity>.md`, `.sengify/wiki/log.md` — the wiki structure every later skill (`query`, `lint`, `feature`, `taskify`) reads from.

- [ ] **Step 1: Create `skills/ingest/templates/index.template.md`**

```markdown
# Wiki Index

## Overview

- [Overview](overview.md)

## Entities

## Features
```

- [ ] **Step 2: Create `skills/ingest/templates/overview.template.md`**

```markdown
# Overview

_No content yet — run `/sengify:ingest <source>` to populate this page._
```

- [ ] **Step 3: Create `skills/ingest/templates/log.template.md`**

```markdown
# Operation Log
```

- [ ] **Step 4: Create `skills/ingest/templates/entity.template.md`**

```markdown
# <Entity>

## Definition

<one or two sentences>

## Attributes

- <attribute>: <description>

## Relationships

- [[<other-entity>]]: <how they relate>

## Sources

- <path-or-url>
```

- [ ] **Step 5: Create `skills/ingest/SKILL.md`**

```markdown
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

​```markdown
- YYYY-MM-DD HH:MM — ingest — source: <path-or-url> — pages touched: overview.md, entities/user.md, ...
​```

## Rules

- Never modify the source document
- Never delete existing wiki content — only add, merge, or flag contradictions
- Always cite sources on new claims
- Do not invent entities or facts not present in the source
```

- [ ] **Step 6: Verify frontmatter and templates**

Run: `grep -c '^name: ingest$' skills/ingest/SKILL.md && grep -c '^disable-model-invocation: true$' skills/ingest/SKILL.md && ls skills/ingest/templates`
Expected: both `grep` counts print `1`; `ls` lists all four `.template.md` files.

- [ ] **Step 7: Commit**

```bash
git add skills/ingest
git commit -m "Add sengify ingest skill"
```

---

### Task 6: `query` Skill

**Files:**
- Create: `skills/query/SKILL.md`

**Interfaces:**
- Consumes: `.sengify/wiki/index.md` and any page under `.sengify/wiki/` (produced by Task 5's `ingest` skill).
- Produces: no required output — only writes a new page on explicit user approval.

- [ ] **Step 1: Create `skills/query/SKILL.md`**

```markdown
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

​```markdown
**Answer:** <answer>

**Sources:** [[entity-a]], [[entity-b]], features/checkout/feature.md
​```

If the wiki doesn't contain enough information to answer, say so plainly — do not guess or fill gaps with assumptions.

### 5. Offer to Save

If the answer is substantial enough to be reused later (not a one-line fact lookup), ask the user: "Save this as a wiki page?" If yes, ask where it belongs (a new page under `entities/`, a new `concepts/<slug>.md`, or appended to an existing page) and write it there, citing this query as the source. If no, or if the answer was trivial, don't write anything.

## Rules

- Read-only against the wiki unless the user explicitly approves saving a new page
- Never search the codebase or the web — answer strictly from wiki content
- Always cite which pages the answer came from
- Say "I don't know" rather than guessing when the wiki lacks the answer
```

- [ ] **Step 2: Verify frontmatter**

Run: `grep -c '^name: query$' skills/query/SKILL.md && grep -c '^disable-model-invocation: true$' skills/query/SKILL.md`
Expected: both print `1`.

- [ ] **Step 3: Commit**

```bash
git add skills/query
git commit -m "Add sengify query skill"
```

---

### Task 7: `lint` Skill

**Files:**
- Create: `skills/lint/SKILL.md`

**Interfaces:**
- Consumes: every page under `.sengify/wiki/` and `.sengify/wiki/log.md` (produced by Task 5's `ingest` skill).
- Produces: a report only — no files.

- [ ] **Step 1: Create `skills/lint/SKILL.md`**

```markdown
---
name: lint
description: Audit the project wiki for contradictions, stale claims, orphaned pages, and broken cross-references — report only, no fixes
disable-model-invocation: true
---

# Sengify Lint

Audit the wiki. Report findings; never fix them automatically.

## Steps

### 1. Load the Wiki

Read every page under `.sengify/wiki/`. If the directory doesn't exist, tell the user and stop.

### 2. Check for Contradictions

Compare claims across pages (especially entity pages that reference each other, and `overview.md` vs entity pages). Flag any pair of statements that can't both be true.

### 3. Check for Stale Claims

For each page with a `Sources` citation, check `log.md` for a later ingest that touched the same page or a page it references. Flag claims whose cited source predates a later, contradicting ingest.

### 4. Check for Orphaned Pages

A page is orphaned if no other wiki page links to it via `[[...]]` and it isn't linked from `index.md`. List every orphan.

### 5. Check for Broken Cross-References

For every `[[reference]]` found in any page, confirm a matching file exists (`entities/<reference>.md`, `features/<reference>/feature.md`, etc). List every reference with no matching file.

### 6. Report

Present findings grouped by category (Contradictions / Stale Claims / Orphaned Pages / Broken References). One line per finding, naming the file(s) and a short reason. If a category has no findings, say so explicitly rather than omitting it.

## Rules

- Report only — never edit any wiki page
- Do not delete orphaned pages or fix broken references yourself
- Be specific: name exact file paths in every finding
```

- [ ] **Step 2: Verify frontmatter**

Run: `grep -c '^name: lint$' skills/lint/SKILL.md && grep -c '^disable-model-invocation: true$' skills/lint/SKILL.md`
Expected: both print `1`.

- [ ] **Step 3: Commit**

```bash
git add skills/lint
git commit -m "Add sengify lint skill"
```

---

### Task 8: `feature` Skill

**Files:**
- Create: `skills/feature/SKILL.md`

**Interfaces:**
- Consumes: `.sengify/wiki/overview.md` and `.sengify/wiki/entities/*.md` (produced by Task 5's `ingest` skill) as optional context.
- Produces: `.sengify/wiki/features/<slug>/feature.md` and an updated `.sengify/wiki/index.md` `Features` section — consumed by Task 9's `taskify` skill.

- [ ] **Step 1: Create `skills/feature/SKILL.md`**

```markdown
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

​```
❓ **Q1** - **<question title>**: <question body — may include multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body>

➡️ <your recommended answer>
​```

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

​```markdown
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
​```

Update `.sengify/wiki/index.md` to link the new feature page under a `Features` section.

## Rules

- Do not write the feature page until the user has confirmed the summary
- Do not implement anything or touch source code
- One round of questions at a time — never dump the whole tree at once
- Never ask the user for a fact you could find yourself
```

- [ ] **Step 2: Verify frontmatter**

Run: `grep -c '^name: feature$' skills/feature/SKILL.md && grep -c '^disable-model-invocation: true$' skills/feature/SKILL.md`
Expected: both print `1`.

- [ ] **Step 3: Commit**

```bash
git add skills/feature
git commit -m "Add sengify feature skill"
```

---

### Task 9: `taskify` Skill

**Files:**
- Create: `skills/taskify/SKILL.md`

**Interfaces:**
- Consumes: `.sengify/wiki/features/<slug>/feature.md` and its `Related Entities` (produced by Task 8's `feature` skill).
- Produces: `.sengify/wiki/features/<slug>/tasks.md` — the final sengify artifact, handed to scalpel manually.

- [ ] **Step 1: Create `skills/taskify/SKILL.md`**

```markdown
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

​```
1. <Task title> — blocked by: none
   <one-line description>
2. <Task title> — blocked by: 1
   <one-line description>
​```

Ask whether the granularity and blocking edges look right. Iterate until approved. Do not write the file until approved.

### 5. Write the Task Plan

Write `.sengify/wiki/features/<slug>/tasks.md`:

​```markdown
# <Feature Name> — Task Plan

Source feature: [feature.md](./feature.md)

## Tasks

### 1. <Task title>

**Blocked by:** none

<description — the end-to-end behavior this task delivers, from the use cases it covers>

### 2. <Task title>

**Blocked by:** 1

<description>
​```

### 6. Report Next Step

Tell the user the task plan is ready and that each task can now be handed to scalpel: `/scalpel:plan <task description>`.

## Rules

- Do not write file paths, code, or acceptance criteria — that's scalpel's job downstream
- Do not write the task plan until the user approves the breakdown
- Do not invoke scalpel yourself — the handoff is manual
- Tasks are high-level and vertical (cut through all layers a use case touches), not layer-by-layer
```

- [ ] **Step 2: Verify frontmatter**

Run: `grep -c '^name: taskify$' skills/taskify/SKILL.md && grep -c '^disable-model-invocation: true$' skills/taskify/SKILL.md`
Expected: both print `1`.

- [ ] **Step 3: Commit**

```bash
git add skills/taskify
git commit -m "Add sengify taskify skill"
```

---

### Task 10: README and End-to-End Review

**Files:**
- Create: `README.md`

**Interfaces:** None — documentation only, references all five skills by name.

- [ ] **Step 1: Create `README.md`**

```markdown
<h1 align="center">sengify</h1>

<p align="center">
  <em>Understanding your own feature is not optional.</em>
</p>

> Plan features the wiki remembers.

sengify builds a persistent, queryable project wiki from your product docs (PRDs, design notes, specs), then grills feature ideas into concrete use cases and a high-level task plan — ready to hand to [scalpel](https://github.com/Banyango/scalpel) for file-level implementation planning.

## How It Works

1. Drop external docs anywhere and run `/sengify:ingest <path>` to fold them into a persistent project wiki (`.sengify/wiki/`).
2. Ask the wiki questions with `/sengify:query <question>` any time you need a cited answer instead of re-reading source docs.
3. Run `/sengify:lint` periodically to catch contradictions, stale claims, and orphaned pages before they mislead you.
4. Run `/sengify:feature <idea>` to get grilled — one round of questions at a time — until the feature and its use cases are pinned down and written to the wiki.
5. Run `/sengify:taskify <slug>` to turn that feature page into a high-level, dependency-ordered task plan.
6. Hand each task to scalpel: `/scalpel:plan <task description>` for file-level `.plan` files, then `/scalpel:implement`.

### Benefits

1. The wiki compounds — each ingested doc makes the next `query`, `feature`, or `taskify` run smarter, instead of re-reading raw sources every time.
2. Feature scope gets pinned down by a real interview (one round at a time, with recommended answers) instead of a single freeform prompt.
3. `lint` catches contradictions and stale claims before they quietly poison a feature plan.
4. The handoff to scalpel is a plain document — no lock-in, no required integration.

### Where this approach works best

1. You're accumulating product/design docs over time and want them to compound into shared understanding instead of being re-read from scratch each time.
2. You want feature scope interrogated before any code gets planned.
3. You already use (or plan to use) scalpel for the file-level planning step.

### Where this doesn't work

1. You don't have external docs to ingest — there's nothing for the wiki to compound.
2. You want fully autonomous feature generation with no interview step.
3. You need the wiki to reflect the live codebase automatically — sengify's wiki is sourced from docs you feed it, not from scanning code (except `taskify`'s one-time read for context).

## Commands

Five slash commands drive the workflow:

| Command             | What it does                                                        |
|----------------------|-----------------------------------------------------------------------|
| `/sengify:ingest`   | Read an external doc, update the wiki (overview, entities, index, log) |
| `/sengify:query`    | Ask a question against the wiki, get a cited answer                  |
| `/sengify:lint`     | Audit the wiki for problems — report only, no fixes                  |
| `/sengify:feature`  | Grill a feature idea into use cases, write a feature page             |
| `/sengify:taskify`  | Turn a feature page into a high-level, dependency-ordered task plan   |

## Wiki Structure

​```
.sengify/
└── wiki/
    ├── index.md                    # navigation across all pages
    ├── overview.md                 # project description (from ingest)
    ├── entities/
    │   └── <entity>.md             # one page per domain entity
    ├── features/
    │   └── <slug>/
    │       ├── feature.md          # description + use cases (from feature)
    │       └── tasks.md            # high-level task breakdown (from taskify)
    └── log.md                      # chronological ingest/query/lint operation log
​```

## Installation

### Claude Code

​```bash
/plugin marketplace add Banyango/sengify
/plugin install sengify@sengify-dev
​```

Skills are available as `/sengify:ingest`, `/sengify:query`, `/sengify:lint`, `/sengify:feature`, and `/sengify:taskify`.

### Cursor

Add to your `.cursor/plugins.json`:

​```json
{
  "plugins": [
    "git+https://github.com/Banyango/sengify.git"
  ]
}
​```

### GitHub Codex / Copilot CLI

Add to your Codex plugin configuration:

​```json
{
  "plugins": [
    "git+https://github.com/Banyango/sengify.git"
  ]
}
​```

### OpenCode

Add to your `opencode.json`:

​```json
{
  "plugin": [
    "sengify@git+https://github.com/Banyango/sengify.git"
  ]
}
​```

See `.opencode/INSTALL.md` for full OpenCode setup instructions.

### Gemini CLI

Load `GEMINI.md` from this repo into your project root, or import it from your existing `GEMINI.md`.

## License

[MIT](LICENSE).
```

- [ ] **Step 2: End-to-end structural review**

Run:
```bash
find . -name "SKILL.md" -not -path "./node_modules/*" | sort
jq . .claude-plugin/plugin.json .claude-plugin/marketplace.json .codex-plugin/plugin.json .cursor-plugin/plugin.json .kimi-plugin/plugin.json > /dev/null && echo "all manifests valid JSON"
grep -L 'disable-model-invocation: true' skills/*/SKILL.md
```
Expected: the `find` lists exactly `skills/feature/SKILL.md`, `skills/ingest/SKILL.md`, `skills/lint/SKILL.md`, `skills/query/SKILL.md`, `skills/taskify/SKILL.md`; the `jq` line prints `all manifests valid JSON`; the `grep -L` prints nothing (every `SKILL.md` has the line).

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "Add sengify README"
```

## Manual QA (not automatable — no test framework exists for markdown skill content)

After all tasks land, before considering this done:

1. Skim every `SKILL.md` against the design spec section it implements (`docs/superpowers/specs/2026-08-29-sengify-wiki-planning-design.md`) and confirm no step was dropped.
2. If this plugin gets installed into a real Claude Code session later, do a real smoke test: `/sengify:ingest` a short sample doc, confirm `.sengify/wiki/` is created correctly, then `/sengify:query` it back.
