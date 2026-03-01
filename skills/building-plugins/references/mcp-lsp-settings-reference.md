# MCP, LSP & Settings Reference

## MCP (Model Context Protocol)

### .mcp.json File Format

The `.mcp.json` file configures MCP servers at the plugin root:

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio",
      "command": "command-to-run",
      "args": ["arg1", "arg2"],
      "env": {
        "API_KEY": "${ENV_VAR_NAME}"
      }
    }
  }
}
```

### Server Types

#### stdio (most common)

Communicates via stdin/stdout. Best for local tools and scripts.

```json
{
  "mcpServers": {
    "my-tools": {
      "type": "stdio",
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-server/index.js"],
      "env": {
        "NODE_ENV": "production"
      }
    }
  }
}
```

#### SSE (Server-Sent Events)

Connects to an HTTP endpoint using SSE transport.

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "sse",
      "url": "https://api.example.com/mcp/sse"
    }
  }
}
```

#### HTTP (Streamable HTTP)

Connects to an HTTP endpoint with request/response pattern.

```json
{
  "mcpServers": {
    "http-api": {
      "type": "http",
      "url": "https://api.example.com/mcp"
    }
  }
}
```

#### WebSocket

Connects via WebSocket for full-duplex communication.

```json
{
  "mcpServers": {
    "ws-server": {
      "type": "websocket",
      "url": "wss://api.example.com/mcp/ws"
    }
  }
}
```

### Environment Variable Expansion

Environment variables in `.mcp.json` are expanded at runtime:

- `${ENV_VAR}` — expands to the value of the environment variable
- `${CLAUDE_PLUGIN_ROOT}` — expands to the plugin's root directory path

```json
{
  "mcpServers": {
    "database": {
      "type": "stdio",
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servers/db-server.js"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}",
        "DB_PASSWORD": "${DB_PASSWORD}"
      }
    }
  }
}
```

### .mcp.json Template

```json
{
  "mcpServers": {
    "my-server": {
      "type": "stdio",
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp-server/index.js"],
      "env": {}
    }
  }
}
```

---

## LSP (Language Server Protocol)

### .lsp.json File Format

The `.lsp.json` file configures language servers at the plugin root:

```json
{
  "lspServers": {
    "server-name": {
      "command": "language-server-binary",
      "args": ["--stdio"],
      "extensionToLanguage": {
        ".ext": "language-id"
      }
    }
  }
}
```

### Fields

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `command` | Yes | string | Command to start the language server |
| `extensionToLanguage` | Yes | object | Maps file extensions to language IDs |
| `args` | No | string[] | Arguments passed to the command |
| `transport` | No | string | Transport protocol (default: `"stdio"`) |
| `env` | No | object | Environment variables for the server |
| `initializationOptions` | No | object | LSP initialization options |
| `settings` | No | object | LSP workspace settings |
| `rootUri` | No | string | Workspace root URI override |
| `workspaceFolders` | No | array | Additional workspace folders |

### .lsp.json Template

```json
{
  "lspServers": {
    "my-language": {
      "command": "${CLAUDE_PLUGIN_ROOT}/bin/my-language-server",
      "args": ["--stdio"],
      "extensionToLanguage": {
        ".mylang": "my-language",
        ".ml": "my-language"
      },
      "initializationOptions": {
        "diagnostics": true,
        "formatting": true
      },
      "settings": {
        "my-language": {
          "lint": { "enabled": true }
        }
      }
    }
  }
}
```

### Example: TypeScript LSP

```json
{
  "lspServers": {
    "typescript": {
      "command": "typescript-language-server",
      "args": ["--stdio"],
      "extensionToLanguage": {
        ".ts": "typescript",
        ".tsx": "typescriptreact",
        ".js": "javascript",
        ".jsx": "javascriptreact"
      }
    }
  }
}
```

---

## Settings

### settings.json Format

Plugin settings are defined in a `settings.json` file at the plugin root or inline in `plugin.json` under the `settings` key.

```json
{
  "agent": {
    "key": "value"
  }
}
```

Currently, only the `agent` key is supported in settings.

### Inline vs Separate File

**Inline (in plugin.json):**
```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "...",
  "settings": {
    "agent": {
      "customSetting": "value"
    }
  }
}
```

**Separate file (settings.json):**
```json
{
  "agent": {
    "customSetting": "value"
  }
}
```

Use a separate `settings.json` when settings are complex or frequently changed. Use inline when settings are simple and tightly coupled to the plugin identity.

### Settings Best Practices

- Keep settings minimal — only expose what users need to configure
- Document all settings in your README.md
- Provide sensible defaults
- Use the `agent` key namespace to avoid conflicts
