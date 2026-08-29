# sengify: LLM-Wiki Feature Planning — Design

## Summary

sengify is a cross-provider agent skills plugin (structured like [scalpel](https://github.com/Banyango/scalpel)) that implements Karpathy's LLM-wiki pattern to turn external product/design documents into a persistent, queryable project wiki, and turns that wiki into feature plans a developer can hand to scalpel for file-level implementation planning.

Reference for the underlying pattern: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

## Relationship to scalpel

Standalone plugin, not a dependency of scalpel. sengify's `taskify` skill produces a high-level task-plan document; the user manually runs scalpel's `/scalpel:plan <task>` against each task afterward. sengify does not invoke scalpel and does not require it to be installed, but is designed to hand off cleanly to it.

## Skills

Five skills, each `skills/<name>/SKILL.md`, `disable-model-invocation: true` (slash-command only, matching scalpel's convention):

### `ingest <path-or-url>`

Reads an external source document (a PRD, design doc, meeting notes — arbitrary path or URL, no fixed sources folder). Never modifies the source.

- Extracts project-description material and merges it into `.sengify/wiki/overview.md`
- Extracts domain entities and creates/updates `.sengify/wiki/entities/<entity>.md` (definition, attributes, relationships, citation back to the source)
- Maintains `[[entity]]`-style cross-references between pages
- Appends an entry to `.sengify/wiki/log.md`: timestamp, source, pages touched

### `query <question>`

Pure Q&A against the wiki. Searches relevant pages, synthesizes a cited answer (names which pages it drew from). Never edits the wiki on its own initiative. If the answer looks substantial/reusable, asks the user whether to save it as a new page; only writes on explicit yes.

### `lint`

Audits the wiki. Report-only — never auto-fixes, matching scalpel's "surface, don't resolve silently" convention. Checks:

- Contradictions between pages
- Claims that look stale relative to their cited source
- Orphaned pages (no inbound/outbound cross-references)
- Broken `[[links]]`

### `feature <description>`

Grill-me style interrogation, following [mattpocock/skills `grilling`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grilling/SKILL.md) format exactly:

- Maps the feature as a design tree; works it in rounds
- Each round asks every question on the current frontier (decisions whose prerequisites are settled), numbered, each with a recommended answer
- Facts (filesystem/codebase lookups) are the agent's job — dispatch a sub-agent, don't ask the user for anything look-up-able
- Decisions are the user's — wait for their answers before the next round
- Terminates when the frontier is empty and the user confirms shared understanding

Output: `.sengify/wiki/features/<slug>/feature.md` — description plus a numbered list of use cases (`As a <actor>, I want <capability>, so that <benefit>`), referencing relevant `entities/*.md`.

### `taskify`

Reads `features/<slug>/feature.md` (and relevant entity pages for vocabulary). Breaks the feature into a dependency-ordered, high-level task list — vertical slices with blocking edges declared, in the spirit of mattpocock's `to-tickets` but coarser: task-level, no acceptance criteria or file paths (that detail is scalpel's job downstream). Presents the breakdown to the user for approval before writing.

Output: `.sengify/wiki/features/<slug>/tasks.md`.

**Handoff**: sengify stops here. The user runs `/scalpel:plan <task description>` per task manually to get scalpel's file-level `.plan` files.

## Wiki Structure

```
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
```

## Plugin Structure

Mirrors scalpel exactly, renamed:

- `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`
- `.codex-plugin/plugin.json`
- `.cursor-plugin/plugin.json`
- `.kimi-plugin/plugin.json` (includes `skillInstructions` tool-mapping block, matching scalpel's Kimi manifest)
- `.opencode/INSTALL.md`
- `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` — short, describing where skills and manifests live (no entry-point skill — sengify deliberately has no `using-sengify` skill; scalpel found its equivalent annoying and dropped it, and each SKILL.md is self-contained via its own frontmatter)
- `README.md` — documents the `ingest → feature → taskify → scalpel` workflow and per-provider install instructions
- `package.json` — same shape as scalpel's, `name: "sengify"`

Author: Kyle Reczek (kyle@banyango.com), license MIT, same keyword/interface conventions as scalpel's provider manifests.

## Out of Scope

- sengify does not scan or read the codebase during `ingest` — only external documents. (`feature`'s sub-agent fact-finding and `taskify`'s codebase read are the only points where source code is consulted.)
- No auto-fix in `lint` or `query` — report/prompt only.
- No direct integration/API call into scalpel — the task-plan document is the interface; handoff is manual.
