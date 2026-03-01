# Plugin Patterns

Five common plugin archetypes with complete templates. Choose the pattern that best matches your plugin's purpose.

---

## 1. Knowledge Plugin

**Components:** Skills only
**Purpose:** Teach Claude domain expertise, processes, or guidelines
**Examples:** frontend-design, security-guidance, coding-standards

### Structure

```
knowledge-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── domain-expertise/
│       ├── SKILL.md
│       └── references/
│           ├── core-principles.md
│           ├── patterns-catalog.md
│           └── examples.md
├── README.md
└── LICENSE
```

### plugin.json

```json
{
  "name": "knowledge-plugin",
  "version": "1.0.0",
  "description": "Teaches Claude [domain] expertise and best practices."
}
```

### When to Choose

- You want Claude to follow specific guidelines or methodologies
- The plugin provides reference material, not actions
- No user-triggered commands or automations needed

---

## 2. Workflow Plugin

**Components:** Skills + Commands
**Purpose:** Guided multi-step processes with explicit entry points
**Examples:** superpowers, code-review-workflow, release-manager

### Structure

```
workflow-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── main-workflow/
│       ├── SKILL.md
│       └── references/
│           ├── step-details.md
│           └── templates.md
├── commands/
│   ├── start.md
│   ├── review.md
│   └── finish.md
├── README.md
└── LICENSE
```

### plugin.json

```json
{
  "name": "workflow-plugin",
  "version": "1.0.0",
  "description": "Guided workflow for [process] with step-by-step commands."
}
```

### Pattern Notes

- Commands serve as entry points into the skill's workflow
- Each command typically invokes or references the skill
- Skills provide the detailed process; commands provide quick access
- Example command body: `Invoke the workflow-plugin:main-workflow skill. User input: $ARGUMENTS`

---

## 3. Integration Plugin

**Components:** MCP Server + Skills (optional)
**Purpose:** Connect Claude to external services, APIs, or data sources
**Examples:** context7, database-connector, slack-integration

### Structure

```
integration-plugin/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json
├── mcp-server/
│   ├── index.js              ← MCP server implementation
│   ├── package.json
│   └── node_modules/
├── skills/
│   └── usage-guide/          ← Optional: teach Claude how to use the integration
│       └── SKILL.md
├── commands/
│   └── connect.md            ← Optional: explicit connection command
├── README.md
└── LICENSE
```

### plugin.json

```json
{
  "name": "integration-plugin",
  "version": "1.0.0",
  "description": "Connects Claude to [service] for [capability]."
}
```

### .mcp.json

```json
{
  "mcpServers": {
    "service-name": {
      "type": "stdio",
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-server/index.js"],
      "env": {
        "API_KEY": "${SERVICE_API_KEY}"
      }
    }
  }
}
```

### Pattern Notes

- The MCP server provides tools and resources to Claude
- An optional skill teaches Claude when and how to use the integration
- Environment variables let users configure API keys without modifying plugin files
- Consider adding a setup command to help users configure the integration

---

## 4. Automation Plugin

**Components:** Hooks + Commands
**Purpose:** Event-driven automation and guardrails
**Examples:** hookify, code-quality-guard, auto-format, commit-checker

### Structure

```
automation-plugin/
├── .claude-plugin/
│   └── plugin.json
├── hooks.json
├── scripts/
│   ├── pre-bash-check.sh
│   ├── post-write-lint.sh
│   └── session-setup.sh
├── commands/
│   ├── enable.md              ← Toggle automation on
│   └── configure.md           ← Adjust settings
├── README.md
└── LICENSE
```

### plugin.json

```json
{
  "name": "automation-plugin",
  "version": "1.0.0",
  "description": "Automatically [action] when [event] occurs."
}
```

### hooks.json

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/pre-bash-check.sh"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/post-write-lint.sh"
      }
    ],
    "SessionStart": [
      {
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/session-setup.sh"
      }
    ]
  }
}
```

### Pattern Notes

- Hook scripts should be fast (they block the workflow)
- Always use `${CLAUDE_PLUGIN_ROOT}` for script paths
- Make scripts executable (`chmod +x`)
- Consider providing commands to toggle or configure the automation
- Test scripts independently before wiring into hooks

---

## 5. Full-Featured Plugin

**Components:** All types (Skills + Commands + Agents + Hooks + MCP/LSP)
**Purpose:** Comprehensive development platform or complex toolchain
**Examples:** superpowers, plugin-dev, full-stack-toolkit

### Structure

```
full-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── primary-skill/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── ...
│   └── secondary-skill/
│       └── SKILL.md
├── commands/
│   ├── init.md
│   ├── build.md
│   └── deploy.md
├── agents/
│   ├── reviewer.md
│   └── debugger.md
├── hooks.json
├── .mcp.json
├── scripts/
│   └── ...
├── mcp-server/
│   └── ...
├── settings.json
├── README.md
└── LICENSE
```

### Pattern Notes

- Start with the minimum viable set of components and add more as needed
- Each component should have a clear, distinct responsibility
- Skills provide the knowledge; commands provide entry points; agents handle specialized tasks; hooks automate repetitive checks
- Document the interplay between components in your README
- Full-featured plugins have higher maintenance burden — only go this route if genuinely needed

---

## Choosing a Pattern

```
What's the primary purpose?

Teaching Claude knowledge/processes?
  └─ Knowledge Plugin

Multi-step guided workflow?
  └─ Workflow Plugin

Connecting to external services?
  └─ Integration Plugin

Automating reactions to events?
  └─ Automation Plugin

All of the above / complex platform?
  └─ Full-Featured Plugin
```

Start simple. You can always add components later as needs evolve. A knowledge plugin can grow into a workflow plugin by adding commands. A workflow plugin can grow into a full-featured plugin by adding hooks and agents.
