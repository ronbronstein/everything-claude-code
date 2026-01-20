# 9. Greenfield Setup: New Project Configuration

[← Back to Index](./README.md) | [Previous: Security](./08-security.md) | [Next: Quick Reference →](./10-quick-reference.md)

---

## Overview

This guide walks through setting up Claude Code for a new project from scratch, with copy-paste ready configuration.

---

## Step 1: Directory Structure

### Create User-Level Configuration

```bash
# Create user-level directories
mkdir -p ~/.claude/{agents,skills,commands,rules,hooks}
```

### Create Project-Level Configuration

```bash
# In your project root
mkdir -p .claude/{commands,skills,rules}
mkdir -p docs/architecture/{decisions,plans}
```

### Complete Structure

```
~/.claude/                          # User-level (global)
├── settings.json                   # User permissions & hooks
├── CLAUDE.md                       # Personal preferences
├── agents/                         # Reusable agents
├── skills/                         # Workflow definitions
├── commands/                       # Custom slash commands
└── rules/                          # Always-follow guidelines

your-project/                       # Project-level
├── .claude/
│   ├── settings.json               # Project shared config
│   ├── settings.local.json         # Personal (git-ignored)
│   ├── commands/                   # Project-specific commands
│   ├── skills/                     # Project-specific workflows
│   └── rules/                      # Project-specific rules
├── CLAUDE.md                       # Project context (LEAN)
├── CLAUDE.local.md                 # Personal overrides (git-ignored)
├── docs/architecture/              # Long-term memory
│   ├── current-state.md
│   ├── decisions/
│   └── plans/
└── src/
```

---

## Step 2: User Settings (~/.claude/settings.json)

```json
{
  "$schema": "https://json-schema.org/claude-code-settings.json",
  "outputStyle": "Default",
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
      "Bash(git push:*)",
      "Bash(git commit:*)",
      "Bash(npm install:*)",
      "Edit(package.json)",
      "Edit(tsconfig.json)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(git push --force:*)",
      "Read(./.env*)",
      "Read(./secrets/**)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "tool == \"Bash\" && tool_input.command matches \"^git commit\"",
        "hooks": [{
          "type": "command",
          "command": "#!/bin/bash\nnpm run lint --silent || { echo '[Hook] Lint failed' >&2; exit 2; }"
        }],
        "description": "Lint before commit"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\.(ts|tsx|js|jsx)$\"",
        "hooks": [{
          "type": "command",
          "command": "#!/bin/bash\ncommand -v prettier >/dev/null && prettier --write \"$(cat | jq -r '.tool_input.file_path')\" 2>/dev/null || true"
        }],
        "description": "Auto-format after edit"
      }
    ]
  }
}
```

---

## Step 3: User CLAUDE.md (~/.claude/CLAUDE.md)

```markdown
# User Preferences

## Core Principles
- Plan before executing complex tasks
- Write tests first (TDD)
- Never compromise on security
- Prefer immutability

## Code Style
- No emojis in code or comments
- Many small files (200-400 lines, 800 max)
- Organize by feature, not by type

## Git
- Conventional commits: feat:, fix:, refactor:, docs:, test:
- Test locally before committing

## Modular Rules
| Rule | Location |
|------|----------|
| Security | `~/.claude/rules/security.md` |
| Testing | `~/.claude/rules/testing.md` |

## Self-Maintenance
After significant changes, update `docs/architecture/current-state.md`
```

---

## Step 4: Project CLAUDE.md

Create `CLAUDE.md` in your project root:

```markdown
# Project: [PROJECT NAME]

## Overview
[One sentence: what this does and why]

**Stack:** [Framework], [Language], [Database]

## Architecture
```
src/
├── app/           # [Purpose]
├── components/    # [Purpose]
├── lib/           # [Purpose]
└── types/         # [Purpose]
```

## Commands
- `npm run dev` - Dev server (use tmux!)
- `npm test` - Run tests (MUST pass before commit)
- `npm run lint` - Linter
- `npm run build` - Production build

## Workflows
| Task | Resource |
|------|----------|
| Testing | `.claude/rules/testing.md` |
| Security | `.claude/rules/security.md` |

## Rules
1. TDD required - tests before code
2. No hardcoded secrets
3. Update `docs/architecture/current-state.md` after changes

## Project State
See: `docs/architecture/current-state.md`
```

---

## Step 5: Project State File

Create `docs/architecture/current-state.md`:

```markdown
# Current System State

*Last updated: [DATE]*

## Active Features
- [ ] Initial setup

## Recent Changes
- [DATE]: Project initialized with Claude Code

## Architecture Decisions
See `./decisions/` for ADRs

## Known Issues
- [ ] None yet

## Next Steps
1. [First task to implement]
```

---

## Step 6: Core Rules

### ~/.claude/rules/testing.md

```markdown
# Testing Rules

## TDD Workflow (MANDATORY)
1. Write test first (RED)
2. Verify test FAILS
3. Write minimal implementation (GREEN)
4. Verify test PASSES
5. Refactor (IMPROVE)
6. Verify 80%+ coverage

## Test Types Required
- Unit tests for utilities
- Integration tests for APIs
- E2E tests for critical flows

## Rules
- Don't create mocks unless explicitly asked
- Test behavior, not implementation
- Every bug fix needs a regression test
```

### ~/.claude/rules/security.md

```markdown
# Security Rules

## Before ANY Commit
- [ ] No hardcoded secrets
- [ ] All user inputs validated
- [ ] SQL injection prevention
- [ ] XSS prevention
- [ ] Error messages don't leak data

## Secret Management
```typescript
// NEVER
const key = "sk-xxxxx"

// ALWAYS
const key = process.env.API_KEY
if (!key) throw new Error('API_KEY not configured')
```

## If Security Issue Found
1. STOP immediately
2. Fix before continuing
3. Rotate exposed secrets
```

---

## Step 7: Custom Commands

### .claude/commands/plan.md

```markdown
---
description: Create implementation plan before coding
---

Create a detailed plan. **Do not write code yet.**

1. Restate requirements
2. List files to modify
3. Create implementation steps
4. Identify risks
5. Wait for approval

Output format:
# Plan: [Feature]
## Requirements
## Files to Modify
## Steps
## Risks
---
**Awaiting approval.**
```

### .claude/commands/tdd.md

```markdown
---
description: Implement with test-driven development
---

Use strict TDD:
1. Write failing test (RED)
2. Verify it FAILS
3. Write minimal code (GREEN)
4. Verify it PASSES
5. Refactor (IMPROVE)
6. Check 80%+ coverage

Don't skip steps. Don't create mocks unless asked.
```

---

## Step 8: MCP Configuration (~/.claude.json)

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

---

## Step 9: Git Configuration

### .gitignore additions

```gitignore
# Claude Code local files
.claude/settings.local.json
CLAUDE.local.md

# Sensitive files
.env
.env.*
secrets/
*.pem
*.key
```

---

## Step 10: First Run Verification

### Start Claude Code

```bash
cd your-project
claude
```

### Verify Configuration

```
# Check context loaded
"What do you know about this project?"

# Should reference your CLAUDE.md content
```

### Verify Permissions

```
# Test allow rule
"Run npm run lint"

# Test ask rule (should prompt)
"Run git push"

# Test deny rule (should block)
"Read .env"
```

### Verify MCP

```
/mcp
# Should list configured servers
```

### Verify Commands

```
/plan Add user authentication
# Should output planning format without coding
```

---

## Quick Setup Script

Create `setup-claude-project.sh`:

```bash
#!/bin/bash
set -e

echo "Setting up Claude Code for this project..."

# Create directories
mkdir -p .claude/{commands,skills,rules}
mkdir -p docs/architecture/{decisions,plans}

# Create CLAUDE.md
cat > CLAUDE.md << 'EOF'
# Project: [NAME]

## Overview
[Description]

**Stack:** [Stack]

## Commands
- `npm run dev` - Dev server
- `npm test` - Tests
- `npm run lint` - Lint
- `npm run build` - Build

## Rules
1. TDD required
2. No hardcoded secrets
3. Update docs/architecture/current-state.md after changes
EOF

# Create current-state.md
cat > docs/architecture/current-state.md << 'EOF'
# Current System State

*Last updated: $(date +%Y-%m-%d)*

## Active Features
- [ ] Initial setup

## Recent Changes
- $(date +%Y-%m-%d): Project initialized

## Next Steps
1. Define architecture
EOF

# Create .gitignore entries
echo "" >> .gitignore
echo "# Claude Code" >> .gitignore
echo ".claude/settings.local.json" >> .gitignore
echo "CLAUDE.local.md" >> .gitignore

echo "Done! Edit CLAUDE.md and docs/architecture/current-state.md"
```

---

## Verification Checklist

- [ ] `~/.claude/settings.json` exists
- [ ] `~/.claude/CLAUDE.md` exists
- [ ] Project `CLAUDE.md` created
- [ ] `docs/architecture/current-state.md` created
- [ ] `.gitignore` updated
- [ ] `claude --version` works
- [ ] `/mcp` shows servers
- [ ] Custom commands available

---

[← Back to Index](./README.md) | [Previous: Security](./08-security.md) | [Next: Quick Reference →](./10-quick-reference.md)
