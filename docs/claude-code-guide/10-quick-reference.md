# 10. Quick Reference

[← Back to Index](./README.md) | [Previous: Greenfield Setup](./09-greenfield-setup.md)

---

## Built-in Slash Commands

| Command | Purpose |
|---------|---------|
| `/help` | Show help |
| `/config` | Configuration settings |
| `/init` | Initialize project |
| `/memory` | Memory management |
| `/compact` | Compact conversation |
| `/clear` | Clear conversation |
| `/permissions` | Permission settings |
| `/mcp` | MCP server status |
| `/hooks` | Hook configuration |
| `/rewind` | Checkpoint restore |
| `/output-style` | Change output style |
| `/model` | Switch model |
| `/context` | View context usage |
| `/stats` | Usage statistics |
| `/doctor` | Run diagnostics |
| `/tasks` | Background tasks |
| `/vim` | Enable vim mode |
| `/teleport` | Transfer to web |

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` | Toggle thinking mode (sticky) |
| `Shift+Tab` | Toggle Plan Mode |
| `Alt+P` / `Option+P` | Switch model mid-prompt |
| `Esc` (double-tap) | Access checkpoints |
| `Ctrl+C` | Cancel current operation |

---

## Thinking Triggers

| Phrase | Effect |
|--------|--------|
| `"think"` | Standard thinking |
| `"think hard"` | Extended thinking |
| `"think harder"` | More extended |
| `"ultrathink"` | Maximum reasoning |

---

## Permission Tiers

| Tier | Behavior |
|------|----------|
| `allow` | Auto-approved |
| `ask` | Requires confirmation |
| `deny` | Blocked |

---

## Permission Patterns

```json
"Bash"                    // Entire tool
"Bash(npm run test)"      // Exact match
"Bash(npm run:*)"         // Prefix wildcard
"Read(./.env)"            // File path
"Read(./secrets/**)"      // Recursive glob
```

---

## defaultMode Options

| Mode | Behavior |
|------|----------|
| `"default"` | Prompts on first use |
| `"acceptEdits"` | Auto-accepts edits |
| `"plan"` | Analysis only, no modifications |
| `"bypassPermissions"` | Auto-accepts all |

---

## outputStyle Options

| Style | Behavior |
|-------|----------|
| `"Default"` | Efficient, concise |
| `"Explanatory"` | Explains choices |
| `"Learning"` | Collaborative with TODOs |

---

## Hook Events

| Event | When |
|-------|------|
| `PreToolUse` | Before tool execution |
| `PostToolUse` | After tool completes |
| `Stop` | Claude finishes responding |
| `PermissionRequest` | Permission requested |
| `UserPromptSubmit` | User submits prompt |
| `Notification` | Notification sent |
| `SubagentStop` | Subagent finishes |
| `SessionStart` | Session starts |
| `SessionEnd` | Session terminates |
| `PreCompact` | Before compaction |

---

## Hook Exit Codes

| Code | Behavior |
|------|----------|
| `0` | Success, proceed |
| `2` | Block action |
| Other | Warn, continue |

---

## File Locations

| File | Purpose |
|------|---------|
| `~/.claude/settings.json` | User permissions |
| `~/.claude/CLAUDE.md` | User preferences |
| `~/.claude.json` | MCP configuration |
| `.claude/settings.json` | Project config |
| `.claude/settings.local.json` | Personal project config |
| `CLAUDE.md` | Project context |
| `CLAUDE.local.md` | Personal project context |

---

## Model Selection

| Model | Best For | Cost |
|-------|----------|------|
| Haiku 4.5 | Quick tasks, agents | Cheapest |
| Sonnet 4.5 | Main development | Standard |
| Opus 4.5 | Deep reasoning | Premium |

---

## Context Management

| Level | Action |
|-------|--------|
| < 50% | Continue |
| 50-70% | Consider `/compact` |
| 70-85% | Manual `/compact` |
| > 85% | `/compact` or `/clear` |

---

## Common Deny Patterns

```json
{
  "deny": [
    "Bash(rm -rf:*)",
    "Bash(curl:*)",
    "Bash(wget:*)",
    "Bash(git push --force:*)",
    "Read(./.env*)",
    "Read(./secrets/**)",
    "Read(./**/*.pem)",
    "Read(./**/*.key)"
  ]
}
```

---

## MCP Commands

```bash
claude mcp add <name> -- <command>
claude mcp list
claude mcp remove <name>
/mcp                      # In session
```

---

## Agent Frontmatter

```markdown
---
name: agent-name
description: What it does
tools: Read, Grep, Glob, Bash
model: opus
---
```

---

## Workflow Quick Reference

### Explore → Plan → Code → Commit

```
1. Shift+Tab (Plan Mode)
2. "Read codebase, don't code yet"
3. "Create plan with ultrathink"
4. Review plan
5. Shift+Tab (exit Plan Mode)
6. "Proceed with Phase 1"
7. "Create commit"
```

### TDD Loop

```
1. Write test (RED)
2. Verify FAILS
3. Implement (GREEN)
4. Verify PASSES
5. Refactor
6. Check coverage
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| MCP not working | Check `/mcp`, use `--mcp-debug` |
| Hook not triggering | Verify matcher syntax |
| Permission denied | Check `deny` rules |
| Context running out | `/compact` or `/clear` |
| Claude less effective | Start fresh with `/clear` |

---

## Quick Setup Commands

```bash
# User-level
mkdir -p ~/.claude/{agents,skills,commands,rules}

# Project-level
mkdir -p .claude/{commands,skills,rules}
mkdir -p docs/architecture/{decisions,plans}
touch CLAUDE.md docs/architecture/current-state.md
```

---

## Verification Commands

```bash
claude --version          # Check installation
/mcp                      # Check MCP servers
/context                  # Check context usage
/doctor                   # Run diagnostics
/stats                    # View statistics
```

---

## Deprecated Features

| Feature | Replacement |
|---------|-------------|
| `ignorePatterns` | `permissions.deny` |
| `.claude.json` | `settings.json` hierarchy |
| `allowedTools` | `permissions.allow` |
| `@-mention MCP` | `/mcp enable` |
| `#` memory shortcut | Edit CLAUDE.md directly |
| `includeCoAuthoredBy` | `attribution` |

---

## Sources

- [Official Documentation](https://code.claude.com/docs/en/overview)
- [Hooks Reference](https://code.claude.com/docs/en/hooks)
- [Settings Reference](https://code.claude.com/docs/en/settings)
- [MCP Documentation](https://code.claude.com/docs/en/mcp)
- [Anthropic Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)

---

[← Back to Index](./README.md) | [Previous: Greenfield Setup](./09-greenfield-setup.md)
