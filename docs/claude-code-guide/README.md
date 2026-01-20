# The Comprehensive Claude Code Configuration Guide

> A verified, modular reference guide for configuring Claude Code as a senior-level development partner.

**Last Updated:** January 2026
**Claude Code Version:** 2.1.x
**Verification Status:** Cross-referenced against official Anthropic documentation

---

## Table of Contents

### Core Configuration
| # | Section | Description |
|---|---------|-------------|
| 1 | [Context Strategy](./01-context-strategy.md) | CLAUDE.md files, hierarchy, community patterns |
| 2 | [Permissions](./02-permissions.md) | Three-tier system (allow/ask/deny), patterns |
| 3 | [Hooks](./03-hooks.md) | All 10 hook events, automation triggers |
| 4 | [MCP Servers](./04-mcp-servers.md) | Model Context Protocol, extending capabilities |
| 5 | [Agents & Skills](./05-agents-skills.md) | Subagents, skills, custom commands |

### Workflows & Patterns
| # | Section | Description |
|---|---------|-------------|
| 6 | [Workflow Patterns](./06-workflow-patterns.md) | Explore-Plan-Code-Commit, TDD, multi-Claude |
| 7 | [Advanced Features](./07-advanced-features.md) | Checkpoints, plugins, sandbox, thinking modes |
| 8 | [Security](./08-security.md) | Risk mitigation, safe autonomy practices |

### Getting Started
| # | Section | Description |
|---|---------|-------------|
| 9 | [Greenfield Setup](./09-greenfield-setup.md) | New project configuration, templates |
| 10 | [Quick Reference](./10-quick-reference.md) | Cheatsheet, commands, shortcuts |

---

## Quick Navigation by Task

### "I want to..."

| Goal | Go To |
|------|-------|
| Set up a new project | [Greenfield Setup](./09-greenfield-setup.md) |
| Configure permissions safely | [Permissions](./02-permissions.md) |
| Automate formatting/linting | [Hooks](./03-hooks.md) |
| Connect external tools | [MCP Servers](./04-mcp-servers.md) |
| Create custom agents | [Agents & Skills](./05-agents-skills.md) |
| Understand best workflows | [Workflow Patterns](./06-workflow-patterns.md) |
| Use checkpoints/rewind | [Advanced Features](./07-advanced-features.md) |
| Prevent dangerous commands | [Security](./08-security.md) |
| Find a command quickly | [Quick Reference](./10-quick-reference.md) |

---

## Key Corrections from Previous Versions

This guide incorporates corrections based on January 2026 verification:

| Topic | Correction |
|-------|------------|
| **outputStyle** | Options are `"Default"`, `"Explanatory"`, `"Learning"` (not lowercase, no "concise") |
| **Slash commands** | `/plan`, `/tdd`, `/code-review` are CUSTOM commands to add, not built-in |
| **Hook events** | 10 total events (not 6) |
| **ignorePatterns** | DEPRECATED - use permissions.deny instead |
| **Context claims** | "200k to 70k" is community observation, not official |
| **Router pattern** | Community best practice, not official Anthropic guidance |

---

## Attribution

This guide synthesizes information from:

- **Official Sources:**
  - [Claude Code Documentation](https://code.claude.com/docs/en/overview)
  - [Anthropic Engineering Blog](https://www.anthropic.com/engineering/claude-code-best-practices)
  - [Official Hooks Reference](https://code.claude.com/docs/en/hooks)
  - [Official Settings Reference](https://code.claude.com/docs/en/settings)

- **Community Sources:**
  - [everything-claude-code Repository](https://github.com/affaanmustafa/everything-claude-code)
  - Community patterns marked with *(Community Pattern)* throughout

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | Jan 2026 | Modular restructure, verification against official docs |
| 1.0 | Jan 2026 | Initial comprehensive guide |

---

*For the single-file version, see [COMPREHENSIVE-CLAUDE-CODE-GUIDE.md](../COMPREHENSIVE-CLAUDE-CODE-GUIDE.md)*
