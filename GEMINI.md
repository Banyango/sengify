## sengify Plugin — Gemini CLI

This project is the sengify skills plugin itself. When working in this repo:

- Skills live in `skills/<name>/SKILL.md`
- Plugin manifests live in `.<provider>-plugin/plugin.json`
- Five skills drive the workflow: `ingest`, `query`, `lint`, `feature`, `taskify` — see `README.md` for how they fit together

When adding a new skill, create `skills/<name>/SKILL.md` with `name` and `description` frontmatter. Update the Commands table in `README.md`.
