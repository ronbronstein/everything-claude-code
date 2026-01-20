# 8. Security: Safe Autonomy Practices

[← Back to Index](./README.md) | [Previous: Advanced Features](./07-advanced-features.md) | [Next: Greenfield Setup →](./09-greenfield-setup.md)

---

## Overview

Granting Claude Code autonomy requires careful security configuration. This section covers known risks and mitigations based on official documentation and community experience.

---

## Critical Risks & Mitigations

### 1. Destructive Commands

**Risk:** Claude may execute `rm -rf` or similar destructive commands.

**Community Reports:** Users have reported Claude executing destructive commands without fully understanding consequences.

**Mitigation:**

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(rm -r:*)",
      "Bash(rmdir:*)",
      "Bash(DROP TABLE:*)",
      "Bash(DELETE FROM:*)",
      "Bash(TRUNCATE:*)"
    ]
  }
}
```

### 2. Force Push to Main

**Risk:** Irreversible git history modification.

**Mitigation:**

```json
{
  "permissions": {
    "deny": [
      "Bash(git push --force:*)",
      "Bash(git push -f:*)",
      "Bash(git push origin main --force:*)",
      "Bash(git push origin master --force:*)"
    ],
    "ask": [
      "Bash(git push:*)"
    ]
  }
}
```

### 3. Secret Exposure

**Risk:** Claude reading or committing sensitive files.

**Mitigation:**

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./**/*.pem)",
      "Read(./**/*.key)",
      "Read(./**/credentials.*)",
      "Read(./**/.git/config)"
    ]
  }
}
```

**Also in CLAUDE.md:**

```markdown
## Security Rules
- NEVER read or commit .env files
- NEVER hardcode secrets in code
- ALWAYS use environment variables for sensitive data
```

### 4. Config File Corruption

**Risk:** Claude may overwrite config files with blank or invalid values during "testing."

**Mitigation:**

```json
{
  "permissions": {
    "ask": [
      "Edit(package.json)",
      "Edit(tsconfig.json)",
      "Edit(.eslintrc.*)",
      "Write(.github/workflows/*)"
    ]
  }
}
```

**Best Practice:** Always maintain version control and commit before autonomous runs.

### 5. Network Exfiltration

**Risk:** Claude using curl/wget to send data externally.

**Mitigation:**

```json
{
  "permissions": {
    "deny": [
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(nc:*)",
      "Bash(netcat:*)"
    ]
  }
}
```

### 6. Windows WebDAV Vulnerability

**Risk:** Enabling WebDAV allows Claude to trigger network requests bypassing permissions.

**Official Recommendation:**
> Don't enable WebDAV or allow `\\*` paths.

```json
{
  "permissions": {
    "deny": [
      "Read(\\\\*)",
      "Write(\\\\*)"
    ]
  }
}
```

---

## MCP Server Security

> **Official Warning:** "Anthropic does not audit third-party MCP servers."

### Best Practices

1. **Only use servers from trusted providers**
2. **Review server source code** when possible
3. **Use environment variables** for tokens, not hardcoded values
4. **Configure MCP permissions** separately
5. **Monitor with `/mcp`** for unexpected behavior

### Token Security

```json
{
  "mcpServers": {
    "github": {
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**Never:**
```json
{
  "mcpServers": {
    "github": {
      "env": {
        "GITHUB_TOKEN": "ghp_actualtoken123"  // NEVER DO THIS
      }
    }
  }
}
```

---

## Context Compaction Risks

**Community Observation:** Claude becomes less effective after compaction.

### Symptoms

- Forgetting file locations
- Re-introducing previously fixed bugs
- Losing track of architectural decisions
- Repeating already-completed work

### Mitigations

1. **Commit before context operations** - Safe restore point
2. **Manual `/compact` at logical breakpoints** - Better than auto-compact
3. **Use custom summarization instructions:**
   ```
   /compact only keep the authentication decisions and database schema
   ```
4. **Start fresh** when effectiveness degrades significantly

---

## Security Checklist for Autonomous Work

Before enabling high autonomy:

- [ ] `deny` rules configured for destructive commands
- [ ] Sensitive files protected (`deny` on .env, secrets, keys)
- [ ] Config files in `ask` tier (package.json, tsconfig, etc.)
- [ ] Git force push blocked
- [ ] Network tools blocked (curl, wget)
- [ ] MCP tokens in environment variables
- [ ] Version control in place
- [ ] Recent commit as restore point

---

## Recommended Security Configuration

```json
{
  "$schema": "https://json-schema.org/claude-code-settings.json",
  "permissions": {
    "defaultMode": "default",
    "allow": [
      "Read",
      "Bash(npm run lint)",
      "Bash(npm run test:*)",
      "Bash(npm run build)",
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(git log:*)"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(git commit:*)",
      "Bash(npm install:*)",
      "Edit(package.json)",
      "Edit(tsconfig.json)",
      "Write(.github/**)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(rm -r:*)",
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(git push --force:*)",
      "Bash(git push -f:*)",
      "Read(./.env*)",
      "Read(./secrets/**)",
      "Read(./**/*.pem)",
      "Read(./**/*.key)"
    ]
  }
}
```

---

## Hook-Based Security Enforcement

### Pre-Commit Security Check

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "tool == \"Bash\" && tool_input.command matches \"^git commit\"",
      "hooks": [{
        "type": "command",
        "command": "#!/bin/bash\n# Check for secrets in staged files\nif git diff --cached | grep -E '(api[_-]?key|password|secret|token)\\s*[=:]\\s*[\"\\x27][^\"\\x27]+' > /dev/null; then\n  echo '[Hook] BLOCKED: Potential secrets detected in commit' >&2\n  exit 2\nfi"
      }],
      "description": "Block commits containing potential secrets"
    }]
  }
}
```

### Block Dangerous Patterns

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "tool == \"Bash\"",
      "hooks": [{
        "type": "command",
        "command": "#!/bin/bash\ninput=$(cat)\ncmd=$(echo \"$input\" | jq -r '.tool_input.command // \"\"')\n# Block sudo commands\nif echo \"$cmd\" | grep -E '^sudo ' > /dev/null; then\n  echo '[Hook] BLOCKED: sudo commands not allowed' >&2\n  exit 2\nfi"
      }],
      "description": "Block sudo commands"
    }]
  }
}
```

---

## Incident Response

### If Secrets Were Exposed

1. **Revoke immediately** - Rotate all potentially exposed credentials
2. **Check git history** - `git log --all --full-history -- "**/.*"`
3. **Clean history if committed** - Use `git filter-branch` or BFG
4. **Audit access logs** - Check for unauthorized usage
5. **Update deny rules** - Prevent recurrence

### If Destructive Command Executed

1. **Stop Claude immediately** - `Ctrl+C`
2. **Assess damage** - `git status`, check filesystem
3. **Restore from checkpoint** - `/rewind`
4. **Restore from git** - `git checkout .`
5. **Restore from backup** - If needed
6. **Update deny rules** - Prevent recurrence

---

## Security vs. Productivity Balance

| Security Level | Autonomy | Best For |
|----------------|----------|----------|
| Paranoid | Low | Untrusted code, production systems |
| Balanced | Medium | Normal development |
| Trusting | High | Personal projects, sandboxed environments |

### Paranoid Configuration

```json
{
  "permissions": {
    "defaultMode": "default",
    "allow": ["Read"],
    "deny": ["Write", "Edit", "Bash"]
  }
}
```

### Balanced Configuration

See "Recommended Security Configuration" above.

### Trusting Configuration (Sandboxed Only)

```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": ["Read", "Write", "Edit", "Bash(npm:*)", "Bash(git:*)"],
    "deny": ["Bash(rm -rf:*)", "Read(./.env*)"]
  },
  "sandbox": {
    "enabled": true
  }
}
```

---

## Verification Status

| Topic | Source |
|-------|--------|
| Permission syntax | Official docs |
| MCP security warning | Official docs |
| WebDAV vulnerability | Official docs |
| Destructive command risks | Community reports |
| Compaction limitations | Community reports |

---

[← Back to Index](./README.md) | [Previous: Advanced Features](./07-advanced-features.md) | [Next: Greenfield Setup →](./09-greenfield-setup.md)
