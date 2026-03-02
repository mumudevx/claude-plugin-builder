# Marketplace & Distribution

## Distribution Options

| Method | Scope | Persistence | Best For |
|--------|-------|-------------|----------|
| `claude --plugin-dir ./path` | Single session | No | Development & testing |
| `/plugin install ./path` | Local install | Yes | Personal use |
| `/plugin install <git-url>` | From repo | Yes | Sharing with others |
| Marketplace | Public discovery | Yes | Wide distribution |

## What Is a Marketplace?

A marketplace is a **Git repository** containing a `.claude-plugin/marketplace.json` file that lists available plugins. Users add a marketplace to discover and install plugins from it.

Marketplaces are decentralized — anyone can create one. The Anthropic official marketplace is just one of many.

## marketplace.json Schema

```json
{
  "name": "author-name",
  "owner": {
    "name": "Author Display Name"
  },
  "metadata": {
    "description": "Plugins by Author Display Name (author-name)",
    "homepage": "https://github.com/author-name/my-plugin"
  },
  "plugins": [
    {
      "name": "my-plugin",
      "version": "1.0.0",
      "source": "./plugin",
      "description": "What the plugin does."
    },
    {
      "name": "another-plugin",
      "version": "2.1.0",
      "source": "./plugin",
      "description": "Another plugin description."
    }
  ]
}
```

### Top-Level Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Marketplace identifier (kebab-case). Users see this when installing: `/plugin install plugin-name@marketplace-name` |
| `owner.name` | Yes | Display name of the marketplace owner |
| `metadata.description` | No | Short description of the marketplace |
| `metadata.homepage` | No | Homepage URL for the marketplace |

### Plugin Entry Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Plugin identifier (kebab-case). Users see this when installing: `/plugin install plugin-name@marketplace-name` |
| `source` | Yes | Where to fetch the plugin. Can be a relative path (`"./plugins/my-plugin"`), a GitHub object (`{"source": "github", "repo": "owner/repo"}`), a git URL object, or an npm object |
| `description` | No | Short description for marketplace listing |
| `version` | No | Current version (semver). If also set in `plugin.json`, `plugin.json` takes priority |

## Creating a Marketplace on GitHub

1. **Create a new GitHub repository** (e.g., `my-marketplace`)

2. **Add `.claude-plugin/marketplace.json`** with your plugin listings

3. **Push to GitHub**:
   ```bash
   git init
   git add .claude-plugin/marketplace.json
   git commit -m "Initial marketplace"
   git remote add origin https://github.com/you/my-marketplace.git
   git push -u origin main
   ```

4. **Users register your marketplace**:
   ```
   /plugin marketplace add you/my-marketplace
   ```

5. **Users install**:
   ```
   /plugin install <plugin-name>@<marketplace-name>
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
4. Users update with: `claude plugin update <plugin-name>` or `/plugin update <plugin-name>`

## User Installation Flow

### From Marketplace
```
# Add a marketplace (once)
/plugin marketplace add author/my-marketplace

# Install a plugin
/plugin install plugin-name@marketplace-name
```

### CLI Commands

All plugin management commands are also available as CLI commands:

```bash
# Install a plugin
claude plugin install plugin-name@marketplace-name

# Update a plugin
claude plugin update plugin-name

# Disable a plugin (without uninstalling)
claude plugin disable plugin-name

# Enable a disabled plugin
claude plugin enable plugin-name

# Uninstall a plugin
claude plugin uninstall plugin-name
```

All CLI commands support a `--scope` flag (`user`, `project`, `local`) to control where the plugin is installed.

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
