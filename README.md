# Plugin Builder

A Claude Code plugin that guides you through creating new Claude Code plugins from scratch. It provides a structured workflow: brainstorm requirements, design architecture, scaffold the project, implement components, and distribute.

## Installation

### Temporary (single session)
```bash
claude --plugin-dir /path/to/plugin-builder
```

### Permanent
```bash
claude plugin install plugin-builder@<marketplace-name>
```

## Usage

### Via Command
```
/plugin-builder:build-plugin <your plugin idea>
```

### Via Natural Language
Just describe what you want:
- "I want to create a plugin that enforces coding standards"
- "Build a plugin for automated code review"
- "Help me make a plugin that integrates with Jira"

The skill activates automatically when you mention creating, building, or making a plugin.

## What It Does

1. **Brainstorm** — Clarifies what your plugin should do and who it's for
2. **Design** — Determines which components (skills, commands, agents, hooks, MCP, LSP) are needed
3. **Plan** — Creates a detailed implementation plan for your approval
4. **Scaffold** — Sets up the directory structure and plugin.json
5. **Implement** — Writes each component following best practices
6. **Validate** — Verifies the plugin structure and tests it
7. **Distribute** — (Optional) Helps publish to a marketplace

## Requirements

- Claude Code 1.0.33 or later

## Optional Dependencies

- **[superpowers](https://github.com/obra/superpowers)** plugin — If installed, provides enhanced brainstorming and workflow orchestration. The plugin works without it, using its own built-in workflow.

## Plugin Components Reference

This plugin includes comprehensive reference documentation for all Claude Code plugin components:

- **Plugin anatomy** — Directory structure, plugin.json schema, naming conventions
- **Skills & Commands** — SKILL.md format, command frontmatter, argument placeholders
- **Agents & Hooks** — Agent system prompts, hook events, matcher patterns
- **MCP & LSP** — Server configurations, transport types, settings
- **Plugin patterns** — 5 common archetypes with full templates
- **Marketplace** — Distribution, versioning, marketplace creation

## License

MIT
