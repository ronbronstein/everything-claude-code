# 4. MCP Servers: Extending Capabilities

[← Back to Index](./README.md) | [Previous: Hooks](./03-hooks.md) | [Next: Agents & Skills →](./05-agents-skills.md)

---

## Overview

Model Context Protocol (MCP) allows Claude Code to connect to external tools and data sources, extending its capabilities beyond built-in tools.

---

## Configuration Methods

### Method 1: CLI (Recommended)

```bash
claude mcp add github \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=your_token \
  -- npx @modelcontextprotocol/server-github
```

### Method 2: JSON Configuration

In `~/.claude.json` or project `.mcp.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      },
      "description": "GitHub operations"
    }
  }
}
```

**Note:** Environment variable expansion with `${VAR}` syntax is supported.

---

## Configuration Structure

**Verified:** Official format from documentation.

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@scope/server-package"],
      "env": {
        "API_KEY": "${ENV_VAR}"
      },
      "type": "stdio",
      "description": "What this server does"
    }
  }
}
```

### Transport Types

| Type | Description |
|------|-------------|
| `stdio` | Standard input/output (default) |
| `http` | HTTP transport |
| `sse` | Server-Sent Events |

### HTTP Example

```json
{
  "mcpServers": {
    "vercel": {
      "type": "http",
      "url": "https://mcp.vercel.com",
      "description": "Vercel deployments"
    }
  }
}
```

---

## Context Budget Warning

> **Community Observation:** The following is based on community findings, not official documentation.

MCP tools consume your context window:

- **Community reports:** 8-30% overhead just from tool registration
- **Rule of thumb:** Keep under 10 MCPs enabled per project, under 80 tools total
- Use `/context` command to monitor MCP overhead

**Disable unused MCPs per project:**

```json
{
  "disabledMcpServers": ["cloudflare-docs", "railway", "supabase"]
}
```

---

## Example Servers

> **Note:** Anthropic does not officially recommend specific third-party servers. These are examples shown in documentation.

### Official Examples

| Server | Package | Purpose |
|--------|---------|---------|
| GitHub | `@modelcontextprotocol/server-github` | PRs, issues, repos |
| Filesystem | `@modelcontextprotocol/server-filesystem` | Extended file operations |
| Puppeteer | `@modelcontextprotocol/server-puppeteer` | Browser automation |
| Memory | `@modelcontextprotocol/server-memory` | Persistent memory |

### Community Popular

| Server | Package | Purpose |
|--------|---------|---------|
| Supabase | `@supabase/mcp-server-supabase` | Database operations |
| Context7 | `@context7/mcp-server` | Live documentation |
| Sequential Thinking | `@modelcontextprotocol/server-sequential-thinking` | Chain-of-thought |

---

## Full Configuration Example

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      },
      "description": "GitHub operations - PRs, issues, repos"
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"],
      "description": "Persistent memory across sessions"
    },
    "vercel": {
      "type": "http",
      "url": "https://mcp.vercel.com",
      "description": "Vercel deployments"
    },
    "supabase": {
      "command": "npx",
      "args": ["-y", "@supabase/mcp-server-supabase@latest", "--project-ref=${SUPABASE_REF}"],
      "description": "Supabase database operations"
    }
  }
}
```

---

## Managing MCP Servers

### CLI Commands

```bash
# List registered servers
claude mcp list

# Add a server
claude mcp add <name> -- <command>

# Remove a server
claude mcp remove <name>

# Check status within session
/mcp
```

### Scope Options

```bash
# Local only (default) - just for you in this project
claude mcp add github --scope local -- npx @modelcontextprotocol/server-github

# Project scope - shared with team via .mcp.json
claude mcp add github --scope project -- npx @modelcontextprotocol/server-github
```

---

## Troubleshooting

### Debug Mode

```bash
claude --mcp-debug
```

### Common Issues

| Issue | Solution |
|-------|----------|
| Server not starting | Check `npx` is available, verify package name |
| Permission denied | Verify environment variables are set |
| Too slow | Reduce number of enabled MCPs |
| Context running out | Use `disabledMcpServers` to disable unused |

### Windows Considerations

On native Windows (not WSL), use `cmd`:

```json
{
  "mcpServers": {
    "example": {
      "command": "cmd",
      "args": ["/c", "npx", "@modelcontextprotocol/server-example"]
    }
  }
}
```

---

## Security Considerations

> **Important:** From official documentation:
> "Anthropic does not audit third-party MCP servers."

1. **Only use servers from trusted providers**
2. **Review server source code** when possible
3. **Configure MCP permissions separately** from main permissions
4. **Monitor context usage** - some servers are chatty
5. **Keep tokens/secrets secure** - use environment variables, not hardcoded

---

## Deprecated Configuration

**`.claude.json` is deprecated** (v2.0.8+) - use `settings.json` hierarchy instead.

---

## Verification Status

| Claim | Status | Source |
|-------|--------|--------|
| Configuration structure | ✅ Verified | Official docs |
| Transport types | ✅ Verified | Official docs |
| CLI commands | ✅ Verified | Official docs |
| Context budget claims | ⚠️ Community | Not in official docs |
| Specific server recommendations | ⚠️ Community | Anthropic doesn't recommend |

---

[← Back to Index](./README.md) | [Previous: Hooks](./03-hooks.md) | [Next: Agents & Skills →](./05-agents-skills.md)
