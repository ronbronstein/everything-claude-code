# 2. Permissions: Configuring Secure Autonomy

[← Back to Index](./README.md) | [Previous: Context Strategy](./01-context-strategy.md) | [Next: Hooks →](./03-hooks.md)

---

## Overview

Claude Code's permission system controls what tools and commands Claude can execute autonomously vs. requiring confirmation.

---

## Settings File Hierarchy

**Verified:** Official documentation confirms this precedence (highest to lowest):

1. **Enterprise managed policies** (`managed-settings.json`)
2. **Command line arguments**
3. **Local project settings** (`.claude/settings.local.json`) - git-ignored
4. **Shared project settings** (`.claude/settings.json`) - checked into git
5. **User settings** (`~/.claude/settings.json`)

---

## The Three Permission Tiers

**Verified:** Official settings documentation confirms these three arrays.

| Tier | Behavior | Use Case |
|------|----------|----------|
| `allow` | Auto-approved, no prompt | High-frequency, safe operations |
| `ask` | Requires user confirmation | Potentially impactful operations |
| `deny` | Blocked entirely | Dangerous or sensitive operations |

### Basic Example

```json
{
  "$schema": "https://json-schema.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Read",
      "Bash(npm run lint)",
      "Bash(npm run test:*)",
      "Bash(git status)",
      "Bash(git diff)"
    ],
    "ask": [
      "Bash(git push)",
      "Bash(npm install:*)",
      "Edit(package.json)",
      "Write(.github/workflows/*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(git push --force:*)",
      "Read(./.env*)",
      "Read(./secrets/**)"
    ]
  }
}
```

---

## Permission Patterns

**Verified:** Official docs confirm pattern syntax.

| Pattern | Description | Example |
|---------|-------------|---------|
| `Tool` | Allow/deny entire tool | `"Bash"` |
| `Tool(exact)` | Exact command match | `"Bash(npm run test)"` |
| `Tool(prefix:*)` | Wildcard prefix match | `"Bash(npm run:*)"` |
| `Tool(path)` | File path match | `"Read(./.env)"` |
| `Tool(glob/**)` | Recursive glob | `"Read(./secrets/**)"` |

**Important:** From official docs:
> "Bash rules use prefix matching, not regex."

### Enhanced Wildcards (v2.1.0+)

As of v2.1.0, enhanced wildcard support:
```json
"Bash(*-h*)"  // Now works for patterns containing wildcards
```

---

## defaultMode Setting

**Verified:** Official documentation confirms these options.

> **Correction:** Previous versions of this guide incorrectly mixed `defaultMode` with `outputStyle`. These are separate settings.

| Mode | Behavior |
|------|----------|
| `"default"` | Prompts for permission on first use of each tool |
| `"acceptEdits"` | Auto-accepts file edits, prompts for others |
| `"plan"` | Claude can analyze but not modify files or execute commands |
| `"bypassPermissions"` | Skips all permission prompts (use with caution) |

```json
{
  "permissions": {
    "defaultMode": "default",
    "allow": [...],
    "deny": [...]
  }
}
```

---

## Output Style (Separate Setting)

> **Correction:** `outputStyle` is NOT a permission mode. It controls response verbosity.

**Verified options:**

| Style | Behavior |
|-------|----------|
| `"Default"` | Efficient, concise responses |
| `"Explanatory"` | Explains implementation choices and patterns |
| `"Learning"` | Collaborative mode with `TODO(human)` markers |

```json
{
  "outputStyle": "Default"
}
```

**Note:** "concise" is NOT a separate option. Concise behavior is part of Default mode.

**Custom styles:** Create Markdown files in `.claude/output-styles/` for custom output behaviors.

---

## Recommended Configurations

### Development (Balanced)

```json
{
  "permissions": {
    "defaultMode": "default",
    "allow": [
      "Read",
      "Skill",
      "Bash(npm run lint)",
      "Bash(npm run test)",
      "Bash(npm run test:*)",
      "Bash(npm run build)",
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(git log:*)"
    ],
    "ask": [
      "Bash(git push)",
      "Bash(git commit:*)",
      "Bash(npm install:*)",
      "Edit(package.json)",
      "Edit(tsconfig.json)",
      "Write(.github/**)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(git push --force:*)",
      "Read(./.env*)",
      "Read(./secrets/**)"
    ]
  }
}
```

### High Autonomy (Trusted Projects)

```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": [
      "Read",
      "Write",
      "Edit",
      "Bash(npm:*)",
      "Bash(git:*)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(git push --force:*)",
      "Read(./.env*)"
    ]
  }
}
```

### Restricted (Review Mode)

```json
{
  "permissions": {
    "defaultMode": "plan",
    "allow": [
      "Read",
      "Bash(git status)",
      "Bash(git diff)"
    ],
    "deny": [
      "Write",
      "Edit",
      "Bash(rm:*)"
    ]
  }
}
```

---

## Deprecated: ignorePatterns

**Verified Deprecated:** Official docs state this is replaced.

```json
// OLD (deprecated)
{ "ignorePatterns": ["tests/**", ".env"] }

// NEW (current)
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  }
}
```

---

## Additional Settings

| Setting | Purpose |
|---------|---------|
| `additionalDirectories` | Grant access beyond project root |
| `disableBypassPermissionsMode` | Prevent bypass mode activation |

---

## Verification Status

| Claim | Status | Source |
|-------|--------|--------|
| Three-tier system | ✅ Verified | Official docs |
| Pattern syntax | ✅ Verified | Official docs |
| defaultMode options | ✅ Verified | Official docs |
| outputStyle options | ✅ Verified | Official docs |
| ignorePatterns deprecated | ✅ Verified | Official docs |

---

[← Back to Index](./README.md) | [Previous: Context Strategy](./01-context-strategy.md) | [Next: Hooks →](./03-hooks.md)
