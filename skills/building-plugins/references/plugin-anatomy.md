# Plugin Anatomy

## Directory Structure

A Claude Code plugin is a directory with a `.claude-plugin/plugin.json` file at its root. All other components are placed at the plugin root level — **never inside** `.claude-plugin/`.

```
my-plugin/                          ← Plugin root
├── .claude-plugin/
│   └── plugin.json                 ← REQUIRED: plugin identity & metadata
├── skills/
│   ├── skill-one/
│   │   ├── SKILL.md                ← Skill definition (exact name required)
│   │   └── references/             ← Supporting docs for the skill
│   │       ├── guide.md
│   │       └── examples.md
│   └── skill-two/
│       └── SKILL.md
├── commands/
│   ├── do-something.md             ← Slash command: /plugin:do-something
│   └── another-command.md
├── agents/
│   ├── specialist.md               ← Custom agent definition
│   └── reviewer.md
├── hooks.json                      ← Hook event handlers
├── .mcp.json                       ← MCP server configurations
├── .lsp.json                       ← LSP server configurations
├── settings.json                   ← Plugin settings (or inline in plugin.json)
├── scripts/                        ← Hook scripts, utilities
│   ├── pre-commit-check.sh
│   └── format-output.py
├── README.md
└── LICENSE
```

## plugin.json Schema

The `plugin.json` file is the only required file. It lives at `.claude-plugin/plugin.json`.

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "A concise description of what this plugin does.",
  "author": {
    "name": "Author Name",
    "email": "author@example.com",
    "url": "https://github.com/author"
  },
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "repository": {
    "type": "git",
    "url": "https://github.com/author/my-plugin"
  },
  "homepage": "https://github.com/author/my-plugin#readme",
  "bugs": {
    "url": "https://github.com/author/my-plugin/issues"
  },
  "engines": {
    "claude-code": ">=1.0.33"
  },
  "settings": {
    "agent": {
      "customKey": "value"
    }
  }
}
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Plugin identifier. Must be kebab-case, unique. Used in command prefixes (`/name:command`). |
| `version` | string | Semantic version (e.g., `"1.0.0"`). Follow semver conventions. |
| `description` | string | One-sentence description of the plugin. Shown in plugin listings. |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `author` | object | Author info: `name` (required), `email`, `url` |
| `license` | string | SPDX license identifier (e.g., `"MIT"`, `"Apache-2.0"`) |
| `keywords` | string[] | Search/discovery keywords for marketplace |
| `repository` | object | Source repository: `type` and `url` |
| `homepage` | string | Plugin homepage URL |
| `bugs` | object | Bug tracker: `url` and optional `email` |
| `engines` | object | Compatibility: `claude-code` version range |
| `settings` | object | Default plugin settings (currently only `agent` key supported) |

## `${CLAUDE_PLUGIN_ROOT}` Environment Variable

When Claude executes hook scripts or MCP/LSP servers from a plugin, the `${CLAUDE_PLUGIN_ROOT}` environment variable is set to the absolute path of the plugin's root directory. **Always use this variable** instead of hardcoded paths.

```json
{
  "hooks": {
    "PreToolUse": [{
      "type": "command",
      "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
    }]
  }
}
```

This ensures the plugin works regardless of where it's installed on the user's machine.

## Component Auto-Discovery

Claude Code automatically discovers plugin components based on directory conventions:

| Component | Discovery Path | File Pattern |
|-----------|---------------|-------------|
| Skills | `skills/*/SKILL.md` | Must be named exactly `SKILL.md` |
| Commands | `commands/*.md` | Any `.md` file in `commands/` |
| Agents | `agents/*.md` | Any `.md` file in `agents/` |
| Hooks | `hooks.json` | Exactly `hooks.json` at plugin root |
| MCP | `.mcp.json` | Exactly `.mcp.json` at plugin root |
| LSP | `.lsp.json` | Exactly `.lsp.json` at plugin root |
| Settings | `settings.json` | Or inline in `plugin.json` under `settings` |

## Naming Conventions

- **Plugin name**: kebab-case (e.g., `my-awesome-plugin`)
- **Skill directories**: kebab-case (e.g., `skills/code-review/`)
- **Command files**: kebab-case (e.g., `commands/run-tests.md`)
- **Agent files**: kebab-case (e.g., `agents/security-reviewer.md`)
- **Script files**: kebab-case with appropriate extension

## Common Mistakes

| Mistake | Why It Fails | Fix |
|---------|-------------|-----|
| Placing skills/commands inside `.claude-plugin/` | Auto-discovery looks at plugin root | Move to root-level `skills/`, `commands/`, etc. |
| Naming skill file `skill.md` or `Skill.md` | Must be exactly `SKILL.md` | Rename to `SKILL.md` (all caps) |
| Using spaces or camelCase in file names | Convention violation, potential issues | Use kebab-case everywhere |
| Hardcoding paths in hooks or MCP configs | Breaks on other machines | Use `${CLAUDE_PLUGIN_ROOT}` |
| Missing `version` in plugin.json | Required field | Add semver version string |
| Placing `hooks.json` inside `.claude-plugin/` | Won't be discovered | Move to plugin root |
