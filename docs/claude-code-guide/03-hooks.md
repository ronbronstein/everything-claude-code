# 3. Hooks: Trigger-Based Automation

[← Back to Index](./README.md) | [Previous: Permissions](./02-permissions.md) | [Next: MCP Servers →](./04-mcp-servers.md)

---

## Overview

Hooks are **deterministic "must-do" rules** that complement CLAUDE.md's "should-do" suggestions. They execute shell commands automatically at specific workflow points.

---

## All 10 Hook Events

**Verified:** Official documentation lists these event types.

| Event | When | Common Use Cases |
|-------|------|------------------|
| `PreToolUse` | Before tool execution | Block dangerous commands, validate inputs |
| `PostToolUse` | After tool completes | Auto-format, run linters, type checks |
| `Stop` | When Claude finishes responding | Final audits, cleanup |
| `PermissionRequest` | When Claude requests permission (v2.0.45+) | Auto-approve trusted patterns |
| `UserPromptSubmit` | When user submits a prompt | Add context, validate input |
| `Notification` | When Claude sends notifications | Custom notification handling |
| `SubagentStop` | When a subagent finishes (v1.0.41+) | Subagent result processing |
| `SessionStart` | When session starts or resumes | Load context, setup environment |
| `SessionEnd` | When session terminates | Cleanup, state persistence |
| `PreCompact` | Before compaction operations | Save important context |

---

## Hook Configuration Structure

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "#!/bin/bash\nyour-command-here"
          }
        ],
        "description": "What this hook does"
      }
    ]
  }
}
```

---

## Matcher Syntax

**Verified:** Official documentation confirms these patterns.

| Matcher | Matches |
|---------|---------|
| `"Write"` | Exact tool name |
| `"Write\|Edit"` | Multiple tools (pipe-separated) |
| `"*"` or `""` | All tools |
| `tool == "Bash" && tool_input.command matches "pattern"` | Conditional with regex |

**Important:** Matchers are case-sensitive.

---

## Exit Codes

**Verified:** Official docs confirm this behavior.

| Code | Behavior |
|------|----------|
| `0` | Success, proceed with action |
| `2` | **Blocking error** - stops the tool, shows error to Claude |
| Other | Non-blocking warning, continues execution |

---

## Production Hook Examples

### PreToolUse Hooks

**1. Block Dev Servers Outside tmux:**
```json
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"(npm run dev|yarn dev|pnpm dev)\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\necho '[Hook] BLOCKED: Dev server must run in tmux' >&2\necho '[Hook] Use: tmux new-session -d -s dev \"npm run dev\"' >&2\nexit 2"
  }],
  "description": "Ensure dev servers run in tmux for log access"
}
```

**2. Lint Before Commit:**
```json
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"^git commit\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\necho '[Hook] Running lint check...' >&2\nnpm run lint --silent || { echo '[Hook] Lint failed. Fix errors first.' >&2; exit 2; }"
  }],
  "description": "Run linter before allowing git commit"
}
```

**3. Block Lock File Reads:**
```json
{
  "matcher": "tool == \"Read\" && tool_input.file_path matches \"package-lock\\.json$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\necho '[Hook] BLOCKED: Lock files waste tokens' >&2\nexit 2"
  }],
  "description": "Prevent reading large lock files"
}
```

**4. Git Push Review Gate:**
```json
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"^git push\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\necho '[Hook] Review changes before push...' >&2\ngit log --oneline -5 >&2\necho '[Hook] Press Enter to continue or Ctrl+C to abort' >&2\nread -r"
  }],
  "description": "Pause before git push to review"
}
```

### PostToolUse Hooks

**5. Auto-Format with Prettier:**
```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ninput=$(cat)\nfile_path=$(echo \"$input\" | jq -r '.tool_input.file_path // \"\"')\nif [ -n \"$file_path\" ] && [ -f \"$file_path\" ]; then\n  prettier --write \"$file_path\" 2>&1 | head -5 >&2\nfi\necho \"$input\""
  }],
  "description": "Auto-format JS/TS files after edits"
}
```

**6. TypeScript Check:**
```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\.(ts|tsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\nnpx tsc --noEmit --pretty false 2>&1 | head -20 >&2"
  }],
  "description": "TypeScript check after editing"
}
```

**7. Console.log Warning:**
```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ninput=$(cat)\nfile_path=$(echo \"$input\" | jq -r '.tool_input.file_path // \"\"')\nif [ -n \"$file_path\" ] && [ -f \"$file_path\" ]; then\n  grep -n \"console\\.log\" \"$file_path\" && echo '[Hook] WARNING: console.log found' >&2\nfi\necho \"$input\""
  }],
  "description": "Warn about console.log statements"
}
```

### Stop Hooks

**8. Final Console.log Audit:**
```json
{
  "matcher": "*",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\nif git rev-parse --git-dir > /dev/null 2>&1; then\n  modified=$(git diff --name-only HEAD 2>/dev/null | grep -E '\\.(ts|tsx|js|jsx)$' || true)\n  for file in $modified; do\n    [ -f \"$file\" ] && grep -q \"console\\.log\" \"$file\" && echo \"[Hook] console.log in $file\" >&2\n  done\nfi"
  }],
  "description": "Final audit for console.log before session ends"
}
```

### SessionStart Hooks

**9. Load Project Context:**
```json
{
  "matcher": "*",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\nif [ -f docs/architecture/current-state.md ]; then\n  echo '[Hook] Project state loaded' >&2\nfi"
  }],
  "description": "Confirm project state is available"
}
```

---

## New in v2.1.0: Frontmatter Hooks

Hooks can now be defined directly in agent and skill frontmatter:

```markdown
---
name: code-reviewer
hooks:
  PostToolUse:
    - matcher: "Edit"
      command: "npm run lint"
---
```

---

## Environment Variables in Hooks

| Variable | Value |
|----------|-------|
| `$CLAUDE_PROJECT_DIR` | Project root absolute path |
| `$CLAUDE_CODE_REMOTE` | `"true"` for web, empty for CLI |

---

## Security Best Practices

1. **Always quote shell variables:** `"$VAR"` not `$VAR`
2. **Use absolute paths** for scripts
3. **Validate and sanitize inputs** from `tool_input`
4. **Check for path traversal** (`..`)
5. **Never read `.env`** or sensitive files in hooks
6. **Test hooks thoroughly** before deployment

---

## Complete settings.json with Hooks

```json
{
  "$schema": "https://json-schema.org/claude-code-settings.json",
  "permissions": {
    "allow": ["Read", "Bash(npm run:*)"],
    "deny": ["Read(./.env*)"]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "tool == \"Bash\" && tool_input.command matches \"^git commit\"",
        "hooks": [{
          "type": "command",
          "command": "#!/bin/bash\nnpm run lint --silent || exit 2"
        }],
        "description": "Lint before commit"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [{
          "type": "command",
          "command": "#!/bin/bash\nprettier --write \"$(cat | jq -r '.tool_input.file_path')\" 2>/dev/null || true"
        }],
        "description": "Auto-format after edit"
      }
    ],
    "Stop": [
      {
        "matcher": "*",
        "hooks": [{
          "type": "command",
          "command": "#!/bin/bash\necho '[Session ended]' >&2"
        }],
        "description": "Session end notification"
      }
    ]
  }
}
```

---

## Verification Status

| Claim | Status | Source |
|-------|--------|--------|
| 10 hook events | ✅ Verified | Official docs |
| Exit code 2 blocks | ✅ Verified | Official docs |
| Matcher syntax | ✅ Verified | Official docs |
| Frontmatter hooks | ✅ Verified | v2.1.0 changelog |

---

[← Back to Index](./README.md) | [Previous: Permissions](./02-permissions.md) | [Next: MCP Servers →](./04-mcp-servers.md)
