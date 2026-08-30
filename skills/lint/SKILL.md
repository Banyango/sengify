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
