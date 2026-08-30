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
