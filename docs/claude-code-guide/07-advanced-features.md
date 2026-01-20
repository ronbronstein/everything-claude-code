# 7. Advanced Features

[← Back to Index](./README.md) | [Previous: Workflow Patterns](./06-workflow-patterns.md) | [Next: Security →](./08-security.md)

---

## Overview

Claude Code includes several advanced features for safety, extensibility, and enhanced reasoning. Many of these were introduced in late 2024-early 2025.

---

## Checkpoint System & /rewind

**Version:** 2.0+

Claude Code automatically saves code state before each change, allowing rollback.

### How It Works

- Checkpoints saved automatically before modifications
- Restore via `/rewind` command or double-tap `Esc`
- Choose to restore: code only, conversation only, or both

### Using /rewind

```
/rewind           # Interactive checkpoint selection
/rewind 3         # Rewind 3 checkpoints back
```

### Restore Options

| Option | Effect |
|--------|--------|
| Code only | Restores files, keeps conversation |
| Conversation only | Resets conversation, keeps files |
| Both | Full restore to checkpoint state |

### Best Practices

1. **Commit before risky operations** - Checkpoints + git = maximum safety
2. **Use at logical boundaries** - Before major refactoring
3. **Combine with `/clear`** - Fresh start with restored code

---

## Plugin System

**Version:** October 2025+

Full plugin marketplace for extending Claude Code.

### Plugin Commands

```bash
/plugin install <name>      # Install from marketplace
/plugin enable <name>       # Enable a plugin
/plugin disable <name>      # Disable a plugin
/plugin marketplace         # Browse available plugins
/plugin list                # List installed plugins
```

### Repository Configuration

```json
{
  "extraKnownMarketplaces": [
    "https://custom-marketplace.example.com"
  ]
}
```

### Plugin Security

- Plugins execute with Claude Code permissions
- Review plugin source before installing
- Use official marketplace when possible

---

## Sandbox Mode

**Version:** Late 2025+

Isolated execution environment for BashTool on Linux/Mac.

### Configuration

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true
  }
}
```

### What Sandbox Provides

| Feature | Effect |
|---------|--------|
| Filesystem isolation | Limited to project directory |
| Network restrictions | Configurable network access |
| Process isolation | Contained process execution |

### When to Use

- **Enable:** Untrusted codebases, experimental work
- **Disable:** When full system access required (deployment, etc.)

---

## Thinking Mode Controls

**Version:** Various

Control Claude's reasoning depth.

### Activation

| Method | Effect |
|--------|--------|
| `Tab` key | Toggle thinking mode (sticky) |
| `"think"` in prompt | Standard thinking |
| `"think hard"` | Extended thinking |
| `"think harder"` | More extended |
| `"ultrathink"` | Maximum reasoning depth |

### Model Defaults

- **Opus 4.5:** Thinking enabled by default
- **Sonnet 4.5:** Thinking available on demand
- **Haiku 4.5:** Limited thinking capability

### Best Use Cases

| Thinking Level | Use For |
|----------------|---------|
| Standard | Simple tasks, routine changes |
| `think` | Multi-file changes |
| `think hard` | Complex architecture decisions |
| `ultrathink` | Critical design, debugging hard issues |

---

## Model Switching

### During Prompt

`Alt+P` (Option+P on Mac) switches model mid-prompt.

### Via Command

```
/model haiku    # Switch to Haiku
/model sonnet   # Switch to Sonnet
/model opus     # Switch to Opus
```

### Model Selection Guide

| Model | Best For | Cost |
|-------|----------|------|
| Haiku 4.5 | Quick tasks, lightweight agents, pair programming | 3x cheaper |
| Sonnet 4.5 | Main development, complex coding | Standard |
| Opus 4.5 | Architectural decisions, deep reasoning | Premium |

---

## Background Tasks

**Feature:** Auto-backgrounding of long-running commands

### How It Works

- Commands exceeding timeout auto-background
- Configurable via `BASH_DEFAULT_TIMEOUT_MS`
- Monitor with `/tasks` command

### Configuration

```bash
export BASH_DEFAULT_TIMEOUT_MS=120000  # 2 minutes
```

### Monitoring

```
/tasks          # List background tasks
/tasks kill 1   # Kill task by ID
```

---

## Language Configuration

Claude can respond in your preferred language.

### Setting Language

```
"Please respond in Japanese from now on."
```

Or in CLAUDE.md:

```markdown
## Language
Respond in Spanish for all interactions.
```

---

## /teleport Command

**Version:** 2.1.0+

Transfer sessions to claude.ai/code web interface.

```
/teleport
```

This generates a link to continue the conversation in browser.

---

## Stats and Monitoring

### /stats Command

```
/stats          # View usage statistics
```

Shows:
- Token usage
- API calls
- Session duration
- Cost estimate

### /context Command

```
/context        # View context window usage
```

Shows:
- Current context size
- Remaining capacity
- MCP tool overhead

### /doctor Command

```
/doctor         # Run diagnostics
```

Checks:
- Configuration validity
- MCP server status
- Permission issues
- Hook configuration

---

## Vim Mode

```
/vim            # Enable vim keybindings
```

For users who prefer vim-style editing in the Claude Code interface.

---

## Session Persistence

### SessionStart Hook

Automatically run setup when session starts:

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "#!/bin/bash\necho 'Session started' >&2"
      }]
    }]
  }
}
```

### SessionEnd Hook

Cleanup when session terminates:

```json
{
  "hooks": {
    "SessionEnd": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "#!/bin/bash\necho 'Session ended' >&2"
      }]
    }]
  }
}
```

---

## Feature Summary Table

| Feature | Version | Purpose |
|---------|---------|---------|
| Checkpoints/Rewind | 2.0 | Code state rollback |
| Plugin System | Oct 2025 | Extensibility |
| Sandbox Mode | Late 2025 | Isolated execution |
| Thinking Controls | Various | Reasoning depth |
| Model Switching | Various | Optimize cost/capability |
| Background Tasks | Various | Long-running commands |
| /teleport | 2.1.0 | Web transfer |

---

[← Back to Index](./README.md) | [Previous: Workflow Patterns](./06-workflow-patterns.md) | [Next: Security →](./08-security.md)
