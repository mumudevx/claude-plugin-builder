# Marketplace & Distribution

## Distribution Options

| Method | Scope | Persistence | Best For |
|--------|-------|-------------|----------|
| `claude --plugin-dir ./path` | Single session | No | Development & testing |
| `claude plugin add ./path` | Local install | Yes | Personal use |
| `claude plugin add <git-url>` | From repo | Yes | Sharing with others |
| Marketplace | Public discovery | Yes | Wide distribution |

## What Is a Marketplace?

A marketplace is a **Git repository** containing a `marketplace.json` file that lists available plugins. Users add a marketplace to discover and install plugins from it.

Marketplaces are decentralized — anyone can create one. The Anthropic official marketplace is just one of many.

## marketplace.json Schema

```json
{
  "name": "My Plugin Marketplace",
  "description": "A curated collection of Claude Code plugins.",
  "plugins": [
    {
      "name": "my-plugin",
      "description": "What the plugin does.",
      "version": "1.0.0",
      "repository": "https://github.com/author/my-plugin",
      "author": "Author Name",
      "keywords": ["keyword1", "keyword2"],
      "license": "MIT"
    },
    {
      "name": "another-plugin",
      "description": "Another plugin description.",
      "version": "2.1.0",
      "repository": "https://github.com/author/another-plugin",
      "author": "Author Name",
      "keywords": ["keyword3"],
      "license": "MIT"
    }
  ]
}
```

### Plugin Entry Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Plugin name (must match plugin's `plugin.json` name) |
| `description` | Yes | Short description for marketplace listing |
| `version` | Yes | Current version (semver) |
| `repository` | Yes | Git URL where the plugin is hosted |
| `author` | No | Plugin author name |
| `keywords` | No | Discovery keywords |
| `license` | No | SPDX license identifier |

## Creating a Marketplace on GitHub

1. **Create a new GitHub repository** (e.g., `my-marketplace`)

2. **Add `marketplace.json`** at the repository root with your plugin listings

3. **Push to GitHub**:
   ```bash
   git init
   git add marketplace.json
   git commit -m "Initial marketplace"
   git remote add origin https://github.com/you/my-marketplace.git
   git push -u origin main
   ```

4. **Users register your marketplace**:
   ```bash
   claude marketplace add https://github.com/you/my-marketplace
   ```

5. **Users browse and install**:
   ```bash
   claude marketplace search <keyword>
   claude plugin add <plugin-name>
   ```

## Version Management

Follow [Semantic Versioning](https://semver.org/):

- **MAJOR** (1.0.0 → 2.0.0): Breaking changes — removed components, renamed commands, changed behavior
- **MINOR** (1.0.0 → 1.1.0): New features — added skills, commands, or components
- **PATCH** (1.0.0 → 1.0.1): Bug fixes — corrected typos, fixed scripts, updated docs

### Updating a Plugin

1. Update the version in `plugin.json`
2. Update `marketplace.json` in your marketplace repo
3. Commit and push both repositories
4. Users update with: `claude plugin update <plugin-name>`

## User Installation Flow

### From Git URL (Direct)
```bash
claude plugin add https://github.com/author/my-plugin
```

### From Marketplace
```bash
# Add a marketplace (once)
claude marketplace add https://github.com/author/my-marketplace

# Search for plugins
claude marketplace search "code review"

# Install a plugin
claude plugin add plugin-name

# Update a plugin
claude plugin update plugin-name

# Remove a plugin
claude plugin remove plugin-name
```

## Anthropic Official Marketplace

To submit your plugin to the official Anthropic marketplace:

1. Ensure your plugin meets quality guidelines:
   - Valid `plugin.json` with all required fields
   - Clear README with installation and usage instructions
   - No security vulnerabilities in hook scripts
   - Tested with current Claude Code version

2. Host your plugin in a public GitHub repository

3. Submit via the official process (check Anthropic documentation for current submission steps)

## Best Practices

- **Write a clear README** — installation, usage, configuration, and examples
- **Include a LICENSE** — MIT recommended for maximum compatibility
- **Tag releases** — use Git tags matching your semver versions
- **Keep dependencies minimal** — fewer dependencies = fewer install issues
- **Test before publishing** — always verify with `claude --plugin-dir` before sharing
- **Document environment variables** — if your plugin needs API keys or config, document setup clearly
- **Provide uninstall instructions** — make it easy for users to remove if needed
