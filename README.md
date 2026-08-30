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

## Installation

### Claude Code

```bash
/plugin marketplace add Banyango/sengify
/plugin install sengify@sengify-dev
```

Skills are available as `/sengify:ingest`, `/sengify:query`, `/sengify:lint`, `/sengify:feature`, and `/sengify:taskify`.

### Cursor

Add to your `.cursor/plugins.json`:

```json
{
  "plugins": [
    "git+https://github.com/Banyango/sengify.git"
  ]
}
```

### GitHub Codex / Copilot CLI

Add to your Codex plugin configuration:

```json
{
  "plugins": [
    "git+https://github.com/Banyango/sengify.git"
  ]
}
```

### OpenCode

Add to your `opencode.json`:

```json
{
  "plugin": [
    "sengify@git+https://github.com/Banyango/sengify.git"
  ]
}
```

See `.opencode/INSTALL.md` for full OpenCode setup instructions.

### Gemini CLI

Load `GEMINI.md` from this repo into your project root, or import it from your existing `GEMINI.md`.

## License

[MIT](LICENSE).
