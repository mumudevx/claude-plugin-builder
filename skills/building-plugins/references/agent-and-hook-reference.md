# Agents & Hooks Reference

## Agents

### File Structure

Agents are markdown files in the `agents/` directory:

```
agents/
├── code-reviewer.md         ← Custom code review agent
├── security-analyst.md      ← Security-focused agent
└── test-writer.md           ← Test generation agent
```

### Agent Frontmatter

```yaml
---
name: code-reviewer
description: |
  Use this agent to review code for quality, correctness, and best practices.
  <example>
  Context: User has completed implementing a feature.
  user: "Review the authentication module I just built"
  assistant: "I'll use the code-reviewer agent to analyze your authentication implementation"
  </example>
  <example>
  Context: PR review is needed.
  user: "Check this PR for issues"
  assistant: "Let me use the code-reviewer agent to review the PR changes"
  </example>
model: sonnet
color: blue
tools:
  - Read
  - Glob
  - Grep
  - Bash
---
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Agent identifier (kebab-case). |
| `description` | Yes | When to use this agent. Include `<example>` blocks showing user/assistant interactions. |
| `model` | No | Model to use: `opus`, `sonnet`, `haiku`. Defaults to conversation model. |
| `color` | No | Display color for agent output. |
| `tools` | No | List of tools the agent can access. Restricts the agent's capabilities. |

### Example Blocks in Description

The `<example>` blocks in the description help Claude understand when to dispatch to this agent:

```yaml
description: |
  Use this agent for [purpose].
  <example>
  Context: [situation where this agent is useful]
  user: "[what the user might say]"
  assistant: "[how Claude should respond, mentioning the agent]"
  <commentary>[why this agent is the right choice]</commentary>
  </example>
```

Include 2–3 examples covering different trigger scenarios.

### Agent Body (System Prompt)

The markdown body after the frontmatter becomes the agent's system prompt:

```markdown
---
name: security-analyst
description: ...
tools:
  - Read
  - Grep
  - Glob
---

You are a security analyst specializing in web application security.

## Your Role
- Review code for OWASP Top 10 vulnerabilities
- Identify insecure patterns and suggest fixes
- Check for hardcoded secrets, SQL injection, XSS, and CSRF

## Process
1. Scan the codebase for security-sensitive files
2. Analyze each file for vulnerabilities
3. Rate severity (Critical, High, Medium, Low)
4. Provide specific fix recommendations

## Output Format
For each finding:
- **File**: path/to/file.ext:line
- **Severity**: Critical/High/Medium/Low
- **Issue**: Description
- **Fix**: Recommended change
```

### System Prompt Best Practices

- **Define the persona** clearly in the first sentence
- **List specific responsibilities** — what the agent should and shouldn't do
- **Define a process** — step-by-step workflow the agent follows
- **Specify output format** — how results should be presented
- **Set boundaries** — what's out of scope

### Agent Template

```markdown
---
name: agent-name
description: |
  Use this agent to [purpose].
  <example>
  Context: [when to use]
  user: "[trigger phrase]"
  assistant: "[response mentioning agent]"
  </example>
model: sonnet
tools:
  - Read
  - Glob
  - Grep
---

You are a [role] specializing in [domain].

## Your Role
- [Responsibility 1]
- [Responsibility 2]

## Process
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Output Format
[How to present results]
```

---

## Hooks

### hooks.json Structure

The `hooks.json` file defines event handlers at the plugin root:

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "pattern",
        "type": "command",
        "command": "script or command to run"
      }
    ]
  }
}
```

### Hook Events

| Event | When It Fires | Matcher Target |
|-------|--------------|----------------|
| `PreToolUse` | Before a tool is executed | Tool name (e.g., `"Bash"`, `"Write"`) |
| `PostToolUse` | After a tool executes successfully | Tool name |
| `PostToolUseFailure` | After a tool execution fails | Tool name |
| `PermissionRequest` | When Claude requests permission for a tool | Tool name |
| `UserPromptSubmit` | When the user submits a prompt | — |
| `Notification` | When Claude sends a notification | — |
| `Stop` | When Claude stops generating | Stop reason |
| `SubagentStart` | When a subagent is launched | Agent name |
| `SubagentStop` | When a subagent finishes | Agent name |
| `SessionStart` | When a session begins | — |
| `SessionEnd` | When a session ends | — |
| `TeammateIdle` | When a teammate becomes idle | — |
| `TaskCompleted` | When a task is completed | — |
| `PreCompact` | Before context compaction | — |

### Hook Types

| Type | Description | Example |
|------|-------------|---------|
| `command` | Execute a shell command | `"command": "${CLAUDE_PLUGIN_ROOT}/scripts/check.sh"` |
| `prompt` | Inject a prompt into the conversation | `"prompt": "Remember to follow our coding standards."` |
| `agent` | Launch an agent | `"agent": "code-reviewer"` |

### Matcher Patterns

The `matcher` field filters which events trigger the hook:

| Pattern Type | Example | Matches |
|-------------|---------|---------|
| Exact string | `"Bash"` | Only the `Bash` tool |
| Regex | `"/^(Bash\|Write)$/"` | `Bash` or `Write` tools |
| Wildcard | `"*"` | All events of that type |
| Omitted | (no matcher) | All events of that type |

### Hook Input & Output

**Input:** Hooks receive a JSON object on stdin with event-specific data:

```json
{
  "session_id": "abc123",
  "event": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": { "command": "rm -rf /" }
}
```

**Output (command hooks):** The hook's stdout is parsed as JSON:

```json
{
  "decision": "block",
  "reason": "Dangerous command detected"
}
```

### Exit Codes

| Exit Code | Meaning |
|-----------|---------|
| 0 | Success — continue normally |
| 2 | Block the action (for `PreToolUse` and `PermissionRequest`) |
| Non-zero (not 2) | Error — logged but doesn't block |

### Using `${CLAUDE_PLUGIN_ROOT}`

Always use this variable for script paths so the plugin works regardless of installation location:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate-command.sh"
      }
    ],
    "SessionStart": [
      {
        "type": "prompt",
        "prompt": "Welcome! This project uses strict coding standards."
      }
    ]
  }
}
```

### hooks.json Template

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
    "UserPromptSubmit": [
      {
        "type": "prompt",
        "prompt": "Remember: follow the project's coding conventions."
      }
    ],
    "SessionStart": [
      {
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/setup-session.sh"
      }
    ]
  }
}
```

### Hook Best Practices

- **Keep hooks fast** — they run synchronously and block the workflow
- **Use `${CLAUDE_PLUGIN_ROOT}`** for all paths — never hardcode
- **Handle errors gracefully** — a crashing hook disrupts the user's flow
- **Be specific with matchers** — avoid `"*"` unless truly needed
- **Log, don't block** — prefer warnings over hard blocks unless safety-critical
- **Test hooks in isolation** — run scripts manually before wiring them up
