# Skills & Commands Reference

## Skills

### Directory Structure

```
skills/
└── my-skill/
    ├── SKILL.md              ← Main skill file (REQUIRED, exact name)
    └── references/            ← Supporting documents (optional)
        ├── detailed-guide.md
        ├── examples.md
        └── templates.md
```

- Each skill lives in its own directory under `skills/`
- The directory name becomes the skill identifier (kebab-case)
- The main file **must** be named `SKILL.md` (all caps, exact)
- Reference files go in a `references/` subdirectory

### SKILL.md Frontmatter

```yaml
---
name: my-skill-name
description: Use when the user wants to [specific trigger scenario]. Also triggers on phrases like "do X", "create Y", or "help with Z".
---
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Skill identifier. kebab-case, matches directory name. |
| `description` | Yes | When to activate this skill. This is critical — Claude uses this to decide when to load the skill. Write it as a CSO-optimized trigger description. |

#### Writing Good Descriptions (CSO Rules)

The `description` field determines when Claude activates your skill. Optimize it:

- **Start with "Use when"** — tells Claude the activation condition
- **Include specific phrases** users might say (e.g., `"build a plugin"`, `"create a plugin"`)
- **Cover synonyms** — include alternative phrasings
- **Be specific** — vague descriptions cause false activations or missed triggers
- **Keep it under 200 characters** — long descriptions reduce matching accuracy

**Good:** `Use when the user wants to create a new plugin, scaffold a project, or mentions "build a plugin" or "make a plugin".`

**Bad:** `Plugin creation skill.`

### SKILL.md Body Structure

A well-structured skill follows progressive disclosure — the main SKILL.md should be 1,500–2,000 words and reference external files for deep details.

```markdown
---
name: my-skill
description: Use when...
---

# Skill Title

Brief overview (2-3 sentences): what this skill does and when to use it.

## HARD-GATE (optional)

Mandatory preconditions before proceeding.

## Checklist

- [ ] Step 1 — description
- [ ] Step 2 — description
...

## Process Flow (optional)

Graphviz dot diagram showing the workflow.

## Core Content

The main instructional content, organized in logical sections.

## Reference Links

For detailed information:
→ @references/detailed-guide.md
→ @references/examples.md

## Red Flags (optional)

| Red Flag | Action |
|----------|--------|
| Scenario | What to do |
```

### Reference Files

Use `@references/filename.md` syntax to link to supporting documents. Claude will load these when the skill directs it to.

- Keep each reference file focused on one topic
- Target 600–1,200 words per reference file
- Use tables and code blocks for scannable content

### Skill Template

```markdown
---
name: example-skill
description: Use when the user wants to do X, create Y, or mentions "keyword A" or "keyword B".
---

# Example Skill

This skill guides Claude through [process]. Use it when [trigger conditions].

## HARD-GATE

Do NOT proceed to implementation until [precondition] is met.

## Checklist

- [ ] Understand requirements
- [ ] Design approach
- [ ] Get user approval
- [ ] Implement
- [ ] Validate

## [Main Content Sections]

...

## Validation

- [ ] Check 1
- [ ] Check 2

## Red Flags

| Red Flag | Action |
|----------|--------|
| Skipping requirements | Go back to step 1 |
```

---

## Commands

### File Structure

Commands are markdown files in the `commands/` directory:

```
commands/
├── build.md                 ← /plugin:build
├── run-tests.md             ← /plugin:run-tests
└── deploy.md                ← /plugin:deploy
```

The filename (without `.md`) becomes the command name, prefixed by the plugin name: `/plugin-name:command-name`.

### Command Frontmatter

```yaml
---
description: What this command does (shown in /help)
allowed-tools: tool1, tool2, tool3
model: sonnet
argument-hint: <required-arg> [optional-arg]
disable-model-invocation: false
---
```

| Field | Required | Description |
|-------|----------|-------------|
| `description` | Yes | Short description shown in help listings and completions. |
| `allowed-tools` | No | Comma-separated list of tools the command can use. Restricts tool access. |
| `model` | No | Override the model for this command (`sonnet`, `haiku`, `opus`). |
| `argument-hint` | No | Hint shown in autocomplete for expected arguments. |
| `disable-model-invocation` | No | If `true`, prevents Claude from invoking this command automatically. User must type it explicitly. |

### Argument Placeholders

Commands receive user input through placeholders:

| Placeholder | Description |
|-------------|-------------|
| `$ARGUMENTS` | The entire argument string after the command name |
| `$1` | First argument (space-separated) |
| `$2` | Second argument |
| `$3` | Third argument |

```markdown
---
description: Create a new component
argument-hint: <component-name> [component-type]
---

Create a new component named "$1" of type "$2".
If no type specified, default to "functional".

Full input: $ARGUMENTS
```

### File References in Commands

Use `@` to reference other files that Claude should read:

```markdown
---
description: Run the code review workflow
---

Follow the code review process defined in @skills/code-review/SKILL.md

Review the following files based on the user's request: $ARGUMENTS
```

### Command Template

```markdown
---
description: Start the guided workflow for [task]
argument-hint: <description of task>
---

Invoke the plugin-name:skill-name skill to guide the user through [task].

User's input: $ARGUMENTS
```

### Commands vs Skills

| Aspect | Skill | Command |
|--------|-------|---------|
| Activation | Automatic (description matching) | Explicit (`/plugin:command`) |
| Purpose | Teach Claude how to do something | Entry point for a specific action |
| Size | 1,500–2,000 words + references | Usually short (10–50 lines) |
| User awareness | User doesn't need to know it exists | User explicitly invokes it |

A common pattern is a **command that invokes a skill**: the command provides the entry point, and the skill provides the detailed workflow.
