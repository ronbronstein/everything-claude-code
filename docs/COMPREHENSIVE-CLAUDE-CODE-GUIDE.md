# The Comprehensive Guide to Claude Code Configuration

> A complete reference guide synthesized from production-ready configurations and official Anthropic documentation.

---

## Table of Contents

**Part A: Best Practices Guide**
1. [Context Strategy: Writing Effective CLAUDE.md Files](#1-context-strategy-writing-effective-claudemd-files)
2. [Safety & Permissions: Configuring Secure Autonomy](#2-safety--permissions-configuring-secure-autonomy)
3. [Workflow Integration: Planning vs Coding Modes](#3-workflow-integration-planning-vs-coding-modes)
4. [Cost & Efficiency: Optimizing Token Usage](#4-cost--efficiency-optimizing-token-usage)
5. [Hooks: Trigger-Based Automation](#5-hooks-trigger-based-automation)
6. [Agents & Skills: Delegation Architecture](#6-agents--skills-delegation-architecture)
7. [MCP Servers: Extending Capabilities](#7-mcp-servers-extending-capabilities)

**Part B: Greenfield Implementation Guide**
1. [Directory Setup](#part-b-greenfield-implementation-guide)
2. [The "God File" Template](#2-the-god-file-template-claudemd)
3. [Config Initialization](#3-config-initialization)
4. [First Run Instructions](#4-first-run-instructions)

---

# Part A: The Comprehensive Guide to Claude Code Best Practices

## 1. Context Strategy: Writing Effective CLAUDE.md Files

### What is CLAUDE.md?

CLAUDE.md is a special file that Claude automatically reads at the start of every conversation. Think of it as **permanent project memory** - the instructions you'd give a new senior developer joining your team.

### The Three-File Strategy

Claude Code supports hierarchical context files that combine to form complete project understanding:

| File Location | Scope | Purpose |
|--------------|-------|---------|
| `~/.claude/CLAUDE.md` | User-level | Personal preferences across all projects |
| `.claude/CLAUDE.md` or `CLAUDE.md` | Project-level | Shared team context, checked into git |
| `frontend/CLAUDE.md`, `backend/CLAUDE.md` | Subfolder-level | Domain-specific context |

### The WHAT-WHY-HOW Framework

Structure your CLAUDE.md using this proven framework:

#### WHAT Section (Technical Context)
```markdown
## Project Overview

**Tech Stack:** Next.js 15, TypeScript, Tailwind, Supabase
**Architecture:** Monorepo with frontend/backend separation
**Database:** PostgreSQL via Supabase

## File Structure

src/
├── app/           # Next.js app router
├── components/    # Reusable UI components
├── hooks/         # Custom React hooks
├── lib/           # Utility functions
└── types/         # TypeScript definitions
```

#### WHY Section (Rationale)
```markdown
## Architectural Decisions

- Monorepo keeps frontend and backend synchronized
- Supabase handles auth and realtime updates
- Prisma ORM ensures type safety with database
```

#### HOW Section (Workflow)
```markdown
## Build & Test Commands

- `npm run dev` - Start dev server
- `npm run test` - Run all tests
- `npm run lint` - Run ESLint and Prettier
- `npm run build` - Production build

## Code Style Guidelines

- No emojis in code or comments
- Always immutable (never mutate objects/arrays)
- Max 400 lines per file (800 absolute max)
```

### Real Example from Production

From the repository's `examples/CLAUDE.md`:

```markdown
## Critical Rules

### 1. Code Organization
- Many small files over few large files
- High cohesion, low coupling
- 200-400 lines typical, 800 max per file
- Organize by feature/domain, not by type

### 2. Code Style
- No emojis in code, comments, or documentation
- Immutability always - never mutate objects or arrays
- No console.log in production code
- Proper error handling with try/catch
- Input validation with Zod or similar

### 3. Testing
- TDD: Write tests first
- 80% minimum coverage
- Unit tests for utilities
- Integration tests for APIs
- E2E tests for critical flows
```

### The Router Pattern: Keep CLAUDE.md Lean

**Critical Insight:** CLAUDE.md should function as a **router**, not an encyclopedia. Keep it under 50-80 lines and link to detailed resources.

**Why This Matters:**
- Every line in CLAUDE.md consumes tokens on every conversation
- Detailed workflows belong in `.claude/skills/` and `.claude/rules/`
- Claude reads linked files on-demand, saving thousands of tokens per session

**Router Pattern Example:**
```markdown
## Specialized Workflows
*Read these only when performing the specific task:*

| Task | Resource |
|------|----------|
| Refactoring | Read `.claude/skills/refactor-guide.md` |
| Testing | Read `.claude/rules/testing.md` |
| Database | Read `.claude/skills/db-migration.md` |
| Security | Read `.claude/rules/security.md` |
```

### Key Principles

1. **Keep CLAUDE.md lean (50-80 lines)** - Use it as a router to detailed resources
2. **Link, don't embed** - Put detailed workflows in `.claude/skills/` and `.claude/rules/`
3. **Start simple, add friction-based** - Only add rules when you encounter actual problems
4. **Be specific, not generic** - "Use immutable patterns" is better than "write good code"
5. **Include file paths** - Reference actual project locations
6. **Document commands** - List the exact commands to build, test, and deploy

---

## 2. Safety & Permissions: Configuring Secure Autonomy

### Permission Configuration Structure

Claude Code uses a hierarchical settings system with three levels:

```
Precedence (lowest to highest):
1. ~/.claude/settings.json        (User defaults)
2. .claude/settings.json          (Project shared)
3. .claude/settings.local.json    (Project personal, git-ignored)
```

### The Three Permission Tiers

Claude Code supports three permission levels for fine-grained control:

| Tier | Behavior | Use Case |
|------|----------|----------|
| `allow` | Auto-approved, no prompt | High-frequency, safe operations |
| `ask` | Requires user confirmation | Potentially impactful operations |
| `deny` | Blocked entirely | Dangerous or sensitive operations |

### Permission Syntax

```json
{
  "$schema": "https://json-schema.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Skill",
      "Bash(npm run lint)",
      "Bash(npm run test:*)",
      "Bash(git status)",
      "Bash(git diff)",
      "Read"
    ],
    "ask": [
      "Bash(git push)",
      "Bash(npm install:*)",
      "Edit(package.json)",
      "Write(.github/workflows/*)"
    ],
    "deny": [
      "Bash(curl:*)",
      "Bash(rm -rf:*)",
      "Bash(git push --force:*)",
      "Read(./.env*)",
      "Read(./secrets/**)"
    ],
    "defaultMode": "default"
  }
}
```

### Permission Modes (`defaultMode`)

| Mode | Behavior |
|------|----------|
| `default` | Prompts for permission on first use of each tool |
| `acceptEdits` | Auto-accepts file edits, prompts for others |
| `plan` | Planning mode - restricts code modifications |
| `bypassPermissions` | Auto-accepts all (use with caution) |

### Permission Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| `Tool` | Allow/deny entire tool | `"Bash"` |
| `Tool(exact)` | Exact command match | `"Bash(npm run test)"` |
| `Tool(prefix:*)` | Prefix wildcard | `"Bash(npm run:*)"` |
| `Tool(path/**)` | Path glob | `"Read(./src/**)"` |

### Best Practices for Permissions

**Allowlist Critical Operations:**
```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test)",
      "Bash(npm run build)",
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(git add .)",
      "Bash(git commit:*)"
    ]
  }
}
```

**Denylist Dangerous Operations:**
```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(git push --force:*)",
      "Read(./.env*)",
      "Read(./secrets/**)",
      "Write(./node_modules/**)"
    ]
  }
}
```

### Security Checklist

From `rules/security.md`:

```markdown
## Mandatory Security Checks

Before ANY commit:
- [ ] No hardcoded secrets (API keys, passwords, tokens)
- [ ] All user inputs validated
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS prevention (sanitized HTML)
- [ ] CSRF protection enabled
- [ ] Authentication/authorization verified
- [ ] Rate limiting on all endpoints
- [ ] Error messages don't leak sensitive data
```

---

## 3. Workflow Integration: Planning vs Coding Modes

### The Plan-Then-Execute Pattern

From the repository's `/plan` command:

```markdown
## What This Command Does

1. **Restate Requirements** - Clarify what needs to be built
2. **Identify Risks** - Surface potential issues and blockers
3. **Create Step Plan** - Break down implementation into phases
4. **Wait for Confirmation** - MUST receive user approval before proceeding
```

### Four-Step Planning Workflow

1. **Ask for a plan first**
   ```
   User: /plan I need to add real-time notifications
   ```

2. **Prevent premature coding**
   ```
   "Do not write code yet. Just give me the plan."
   ```

3. **Review and refine**
   - Question assumptions
   - Identify missing requirements
   - Validate approach

4. **Give explicit approval**
   ```
   "Proceed with Phase 1"
   ```

### Plan Output Format

From `agents/planner.md`:

```markdown
# Implementation Plan: [Feature Name]

## Overview
[2-3 sentence summary]

## Requirements
- [Requirement 1]
- [Requirement 2]

## Architecture Changes
- [Change 1: file path and description]
- [Change 2: file path and description]

## Implementation Steps

### Phase 1: [Phase Name]
1. **[Step Name]** (File: path/to/file.ts)
   - Action: Specific action to take
   - Why: Reason for this step
   - Dependencies: None / Requires step X
   - Risk: Low/Medium/High

## Testing Strategy
- Unit tests: [files to test]
- Integration tests: [flows to test]
- E2E tests: [user journeys to test]

## Risks & Mitigations
- **Risk**: [Description]
  - Mitigation: [How to address]
```

### TDD Integration

After planning, use test-driven development:

```
RED → GREEN → REFACTOR → REPEAT

RED:      Write a failing test
GREEN:    Write minimal code to pass
REFACTOR: Improve code, keep tests passing
REPEAT:   Next feature/scenario
```

### Command Sequence

```
/plan    → Create implementation plan
/tdd     → Implement with tests first
/build-fix → Fix any build errors
/code-review → Review implementation
```

---

## 4. Cost & Efficiency: Optimizing Token Usage

### Context Window Management

**Critical Warning:** Your 200k context window can shrink to 70k with too many MCP tools enabled.

From `rules/performance.md`:

```markdown
## Context Window Management

Avoid last 20% of context window for:
- Large-scale refactoring
- Feature implementation spanning multiple files
- Debugging complex interactions

Lower context sensitivity tasks:
- Single-file edits
- Independent utility creation
- Documentation updates
- Simple bug fixes
```

### MCP Tool Budget

**Rule of Thumb:**
- Configure: 20-30 MCPs total
- Keep under 10 enabled per project
- Keep total active tools under 80

**Disable Unused MCPs Per Project:**
```json
{
  "disabledMcpServers": [
    "cloudflare-docs",
    "railway",
    "supabase"
  ]
}
```

### Model Selection Strategy

From `rules/performance.md`:

| Model | Use Case | Cost |
|-------|----------|------|
| **Haiku 4.5** | Lightweight agents, pair programming, frequent tasks | 3x cheaper |
| **Sonnet 4.5** | Main development, orchestration, complex coding | Standard |
| **Opus 4.5** | Architectural decisions, deep reasoning, research | Premium |

### Output Style Configuration

Control Claude's verbosity with the `outputStyle` setting:

```json
{
  "outputStyle": "concise"
}
```

| Style | Behavior |
|-------|----------|
| `default` | Standard conversational output |
| `concise` | Minimal output, less filler text |
| `Explanatory` | More detailed explanations |
| `Learning` | Educational mode with extra context |

**Recommendation:** Use `"concise"` in production to reduce token waste from conversational filler like "I will now proceed to..."

### Files to Exclude from Context

**Important:** The `ignorePatterns` setting is **deprecated**. Use these alternatives:

**1. `.gitignore` (Primary Method)**

Files in `.gitignore` are typically excluded from Claude's context:

```gitignore
# Large generated files
node_modules/
.next/
dist/
build/

# Lock files (huge token waste)
package-lock.json
yarn.lock
pnpm-lock.yaml

# Large assets
*.png
*.jpg
*.svg
*.woff2

# Logs
*.log
```

**2. PreToolUse Hooks (For Enforcement)**

Use hooks to block reads of specific files:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "tool == \"Read\" && tool_input.file_path matches \"package-lock\\.json$\"",
      "hooks": [{
        "type": "command",
        "command": "#!/bin/bash\necho '[Hook] BLOCKED: Lock files waste tokens' >&2\nexit 2"
      }],
      "description": "Block reading lock files"
    }]
  }
}
```

### Session Management Tips

1. **Use `/clear` often** - Start fresh between unrelated tasks
2. **Don't accumulate history** - Old context wastes tokens
3. **Compact conversations** - Claude will auto-summarize if needed
4. **Use subagents** - Delegate to focused agents with limited scope

### Self-Maintenance Pattern: External Long-Term Memory

Create a `docs/architecture/` directory that serves as **persistent memory** across sessions. This pattern ensures architectural decisions and project state survive session resets.

**Directory Structure:**
```
docs/
└── architecture/
    ├── current-state.md      # Current system state (auto-updated)
    ├── decisions/            # Architecture Decision Records (ADRs)
    │   ├── 001-auth-system.md
    │   └── 002-database-choice.md
    └── plans/                # Implementation plans
        └── active-plan.md
```

**The `current-state.md` Template:**
```markdown
# Current System State

*Last updated: [DATE] by Claude*

## Active Features
- [Feature 1]: [Status - Complete/In Progress/Planned]
- [Feature 2]: [Status]

## Recent Changes
- [DATE]: [Change description]
- [DATE]: [Change description]

## Known Issues
- [ ] [Issue 1]
- [ ] [Issue 2]

## Next Steps
1. [Next task]
2. [Following task]
```

**Enforcement Rule (add to CLAUDE.md):**
```markdown
## Self-Maintenance Rule

After completing ANY significant change:
1. Update `docs/architecture/current-state.md` with what changed
2. If architectural decision was made, create ADR in `docs/architecture/decisions/`
3. Update "Recent Changes" section with date and description
```

This creates an external "brain" that persists across session boundaries, allowing Claude to quickly understand project state without re-analyzing the entire codebase.

---

## 5. Hooks: Trigger-Based Automation

### What Are Hooks?

Hooks are **deterministic "must-do" rules** that complement CLAUDE.md's "should-do" suggestions. They execute automatically at specific workflow points.

### Hook Events

| Event | When | Use Case |
|-------|------|----------|
| `PreToolUse` | Before tool execution | Block dangerous commands, validate inputs |
| `PostToolUse` | After tool succeeds | Auto-format, run linters, type checks |
| `PermissionRequest` | When permission requested | Auto-approve trusted operations |
| `UserPromptSubmit` | When user submits prompt | Add context, validate input |
| `Stop` | When session ends | Final audits, cleanup |
| `SessionStart` | Session initialization | Load context, setup |

### Hook Configuration Structure

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "tool == \"Bash\" && tool_input.command matches \"pattern\"",
        "hooks": [
          {
            "type": "command",
            "command": "#!/bin/bash\n# Your script here"
          }
        ],
        "description": "What this hook does"
      }
    ]
  }
}
```

### Production Hook Examples

From `hooks/hooks.json`:

**1. Block Dev Servers Outside tmux:**
```json
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"(npm run dev|pnpm( run)? dev)\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\necho '[Hook] BLOCKED: Dev server must run in tmux' >&2\necho '[Hook] Use: tmux new-session -d -s dev \"npm run dev\"' >&2\nexit 1"
  }],
  "description": "Block dev servers outside tmux - ensures log access"
}
```

**2. Auto-Format After Edit:**
```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\nfile_path=$(cat | jq -r '.tool_input.file_path')\nprettier --write \"$file_path\" 2>&1 >&2"
  }],
  "description": "Auto-format JS/TS files with Prettier after edits"
}
```

**3. TypeScript Check After Edit:**
```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\n# Find project root and run tsc\nnpx tsc --noEmit --pretty false 2>&1 | grep \"$file_path\" | head -10 >&2"
  }],
  "description": "TypeScript check after editing .ts/.tsx files"
}
```

**4. Warn About console.log:**
```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\nfile_path=$(cat | jq -r '.tool_input.file_path')\ngrep -n \"console\\.log\" \"$file_path\" && echo '[Hook] Remove console.log' >&2"
  }],
  "description": "Warn about console.log statements after edits"
}
```

**5. Git Push Review Gate:**
```json
{
  "matcher": "tool == \"Bash\" && tool_input.command matches \"git push\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\necho '[Hook] Review changes before push...' >&2\necho '[Hook] Press Enter to continue or Ctrl+C to abort' >&2\nread -r"
  }],
  "description": "Pause before git push to review changes"
}
```

**6. Final Console.log Audit (Stop Hook):**
```json
{
  "matcher": "*",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\nmodified=$(git diff --name-only HEAD | grep -E '\\.(ts|tsx|js|jsx)$')\nfor file in $modified; do\n  grep -q 'console.log' \"$file\" && echo \"[Hook] console.log in $file\" >&2\ndone"
  }],
  "description": "Final audit for console.log before session ends"
}
```

### Hook Exit Codes

| Code | Behavior |
|------|----------|
| `0` | Success, proceed with action |
| `2` | Blocking error, stop action |
| Other | Non-blocking warning, continue |

### Security Best Practices for Hooks

- Always quote shell variables: `"$VAR"`
- Use absolute paths for scripts
- Validate and sanitize inputs
- Check for path traversal (`..`)
- Never read `.env` or sensitive files

---

## 6. Agents & Skills: Delegation Architecture

### What Are Agents?

Agents are **specialized subagents** with limited scope and specific tools. They handle delegated tasks efficiently.

### Agent Definition Format

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, maintainability
tools: Read, Grep, Glob, Bash
model: opus
---

You are a senior code reviewer. Your role is to...
```

### Production Agent Examples

From `agents/`:

**1. Planner Agent:**
```markdown
---
name: planner
description: Expert planning specialist for complex features
tools: Read, Grep, Glob
model: opus
---

You are an expert planning specialist focused on creating
comprehensive, actionable implementation plans.

## Planning Process

1. Requirements Analysis - Understand completely
2. Architecture Review - Analyze existing code
3. Step Breakdown - Create detailed steps
4. Implementation Order - Prioritize by dependencies
```

**2. Code Reviewer Agent:**
```markdown
---
name: code-reviewer
description: Reviews code for quality, security, maintainability
tools: Read, Grep, Glob, Bash
model: opus
---

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code is simple and readable
- No exposed secrets or API keys
- Input validation implemented
- Good test coverage
```

**3. TDD Guide Agent:**
```markdown
---
name: tdd-guide
description: Test-Driven Development specialist
tools: Read, Write, Edit, Bash, Grep
model: opus
---

## TDD Workflow

1. Write Test First (RED)
2. Run Test (Verify it FAILS)
3. Write Minimal Implementation (GREEN)
4. Run Test (Verify it PASSES)
5. Refactor (IMPROVE)
6. Verify Coverage (80%+)
```

### When to Use Which Agent

From `rules/agents.md`:

| Agent | When to Use |
|-------|-------------|
| `planner` | Complex features, architectural changes |
| `architect` | System design decisions |
| `tdd-guide` | New features, bug fixes |
| `code-reviewer` | After writing code |
| `security-reviewer` | Before commits |
| `build-error-resolver` | When build fails |
| `e2e-runner` | Critical user flows |

### Skills (Workflow Definitions)

Skills are reusable workflow patterns. Example structure:

```markdown
---
name: coding-standards
description: Universal coding standards for TypeScript/React
---

# Coding Standards

## Variable Naming
- Descriptive names: `marketSearchQuery` not `q`
- Boolean prefix: `isUserAuthenticated` not `flag`

## Immutability Pattern (CRITICAL)
const updated = { ...original, newField: value }
const newArray = [...items, newItem]
```

### Parallel Agent Execution

From `rules/agents.md`:

```markdown
## Parallel Task Execution

ALWAYS use parallel Task execution for independent operations:

# GOOD: Parallel execution
Launch 3 agents in parallel:
1. Agent 1: Security analysis of auth.ts
2. Agent 2: Performance review of cache system
3. Agent 3: Type checking of utils.ts

# BAD: Sequential when unnecessary
First agent 1, then agent 2, then agent 3
```

---

## 7. MCP Servers: Extending Capabilities

### What is MCP?

Model Context Protocol (MCP) allows Claude Code to connect to external tools and data sources.

### Configuration Methods

**Method 1: CLI (Recommended)**
```bash
claude mcp add github \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=your_token \
  -- npx @modelcontextprotocol/server-github
```

**Method 2: JSON Configuration**

In `~/.claude.json` or `.mcp.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "YOUR_TOKEN"
      },
      "description": "GitHub operations"
    }
  }
}
```

### Recommended MCP Servers

From `mcp-configs/mcp-servers.json`:

| Server | Purpose | Type |
|--------|---------|------|
| `github` | PRs, issues, repos | npx |
| `supabase` | Database operations | npx |
| `vercel` | Deployments | HTTP |
| `railway` | Railway platform | npx |
| `firecrawl` | Web scraping | npx |
| `memory` | Persistent memory | npx |
| `sequential-thinking` | Chain-of-thought | npx |
| `cloudflare-docs` | CF documentation | HTTP |
| `clickhouse` | Analytics queries | HTTP |
| `context7` | Live doc lookup | npx |

### Full MCP Configuration Example

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "YOUR_GITHUB_PAT_HERE"
      }
    },
    "supabase": {
      "command": "npx",
      "args": ["-y", "@supabase/mcp-server-supabase@latest", "--project-ref=YOUR_REF"]
    },
    "vercel": {
      "type": "http",
      "url": "https://mcp.vercel.com"
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

### Context Budget Warning

**Critical:** MCP tools consume your context window.

- MCP output > 10,000 tokens triggers warning
- Too many MCPs can reduce 200k to 70k available
- Rule: Keep under 10 enabled per project, under 80 total tools

**Disable per project:**
```json
{
  "disabledMcpServers": ["cloudflare-docs", "railway"]
}
```

---

# Part B: Greenfield Implementation Guide

## 1. Directory Setup: The Ideal AI-Native Structure

### Step 1: Create the .claude Directory Structure

```bash
# Create user-level config directory
mkdir -p ~/.claude/{agents,skills,commands,rules,hooks}

# Create project-level config directory
mkdir -p .claude/{commands,skills,rules}

# Create external long-term memory directory
mkdir -p docs/architecture/{decisions,plans}

# Initialize current-state.md
cat > docs/architecture/current-state.md << 'EOF'
# Current System State

*Last updated: [DATE] by Claude*

## Active Features
- [Feature]: [Status]

## Recent Changes
- [DATE]: Initial setup

## Known Issues
- [ ] None yet

## Next Steps
1. [First task]
EOF
```

### Complete Directory Structure

```
~/.claude/                          # User-level (global)
├── settings.json                   # User permissions & hooks
├── CLAUDE.md                       # Personal preferences
├── agents/                         # Reusable agents
│   ├── planner.md
│   ├── architect.md
│   ├── code-reviewer.md
│   ├── security-reviewer.md
│   ├── tdd-guide.md
│   ├── build-error-resolver.md
│   ├── e2e-runner.md
│   ├── refactor-cleaner.md
│   └── doc-updater.md
├── skills/                         # Workflow definitions
│   ├── coding-standards.md
│   ├── backend-patterns.md
│   ├── frontend-patterns.md
│   └── tdd-workflow/
│       └── SKILL.md
├── commands/                       # Slash commands
│   ├── tdd.md
│   ├── plan.md
│   ├── code-review.md
│   ├── build-fix.md
│   └── e2e.md
└── rules/                          # Always-follow guidelines
    ├── security.md
    ├── coding-style.md
    ├── testing.md
    ├── git-workflow.md
    ├── agents.md
    └── performance.md

your-project/                       # Project-level
├── .claude/
│   ├── settings.json               # Project shared config
│   ├── settings.local.json         # Personal (git-ignored)
│   ├── commands/                   # Project-specific commands
│   ├── skills/                     # Project-specific workflows
│   └── rules/                      # Project-specific rules
├── CLAUDE.md                       # Project context (LEAN ROUTER)
├── docs/
│   └── architecture/               # External long-term memory
│       ├── current-state.md        # Auto-updated project state
│       ├── decisions/              # Architecture Decision Records
│       └── plans/                  # Implementation plans
├── src/
│   ├── app/                        # Application code
│   ├── components/                 # UI components
│   ├── hooks/                      # Custom hooks
│   ├── lib/                        # Utilities
│   └── types/                      # TypeScript types
└── tests/                          # Test files
```

---

## 2. The "God File" Template (CLAUDE.md)

### User-Level CLAUDE.md (~/.claude/CLAUDE.md)

```markdown
# User-Level Configuration

You are Claude Code. I use specialized agents and skills for complex tasks.

## Core Philosophy

1. **Agent-First**: Delegate to specialized agents for complex work
2. **Parallel Execution**: Use Task tool with multiple agents when possible
3. **Plan Before Execute**: Use Plan Mode for complex operations
4. **Test-Driven**: Write tests before implementation
5. **Security-First**: Never compromise on security

## Modular Rules

Detailed guidelines are in `~/.claude/rules/`:

| Rule File | Contents |
|-----------|----------|
| security.md | Security checks, secret management |
| coding-style.md | Immutability, file organization |
| testing.md | TDD workflow, 80% coverage |
| git-workflow.md | Commit format, PR workflow |
| agents.md | Agent orchestration |
| performance.md | Model selection, context management |

## Available Agents

Located in `~/.claude/agents/`:

| Agent | Purpose |
|-------|---------|
| planner | Feature implementation planning |
| architect | System design and architecture |
| tdd-guide | Test-driven development |
| code-reviewer | Code review for quality/security |
| security-reviewer | Security vulnerability analysis |
| build-error-resolver | Build error resolution |

## Personal Preferences

### Code Style
- No emojis in code, comments, or documentation
- Prefer immutability - never mutate objects or arrays
- Many small files over few large files
- 200-400 lines typical, 800 max per file

### Git
- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`
- Always test locally before committing
- Small, focused commits

### Testing
- TDD: Write tests first
- 80% minimum coverage

## Success Metrics

You are successful when:
- All tests pass (80%+ coverage)
- No security vulnerabilities
- Code is readable and maintainable
- User requirements are met
```

### Project-Level CLAUDE.md Template (Lean Router Pattern)

This template follows the router pattern - keep it under 50-80 lines and link to detailed resources.

```markdown
# Project: [YOUR PROJECT NAME]

## Overview
[One sentence: what this project does and why]

**Stack:** [Framework], [Language], [Database], [Key Libraries]

## Architecture Map
*Do not hallucinate files outside this structure:*
```
src/
├── app/           # [Purpose]
├── components/    # [Purpose]
├── lib/           # [Purpose]
└── types/         # [Purpose]
```

## Commands (The "Truth")
- **Dev:** `npm run dev` (use tmux!)
- **Test:** `npm test` (MUST pass before commit)
- **Lint:** `npm run lint`
- **Build:** `npm run build`

## Specialized Workflows
*Read these only when performing the specific task:*

| Task | Resource |
|------|----------|
| Testing/TDD | Read `.claude/rules/testing.md` |
| Security | Read `.claude/rules/security.md` |
| Refactoring | Read `.claude/skills/refactor-guide.md` |
| Database | Read `.claude/skills/db-migration.md` |

## Imperative Rules
1. **No broken builds** - Fix compilation errors immediately
2. **TDD required** - Write tests before implementation
3. **Plan first** - If touching >2 files, update `docs/architecture/plans/`
4. **Update state** - After significant changes, update `docs/architecture/current-state.md`

## Project State
Current state and decisions: `docs/architecture/current-state.md`
```

### Detailed Project CLAUDE.md (Alternative)

For projects that need more inline context, use this expanded version:

```markdown
# Project: [YOUR PROJECT NAME]

## Overview
[Brief description - 2-3 sentences about what this project does]

**Tech Stack:**
- Frontend: [e.g., Next.js 15, TypeScript, Tailwind]
- Backend: [e.g., Node.js, FastAPI, etc.]
- Database: [e.g., PostgreSQL, Supabase]
- Testing: [e.g., Jest, Playwright]

## Commands
- `npm run dev` - Start dev server (use tmux!)
- `npm run test` - Run unit tests
- `npm run lint` - Run linter
- `npm run build` - Production build

## Critical Rules
- Many small files (200-400 lines, 800 max)
- No emojis in code or comments
- Immutability always - never mutate
- TDD: Write tests before implementation
- 80% minimum coverage
- No hardcoded secrets

## Specialized Workflows
| Task | Resource |
|------|----------|
| Testing | `.claude/rules/testing.md` |
| Security | `.claude/rules/security.md` |
| Code Style | `.claude/rules/coding-style.md` |

## Self-Maintenance
After significant changes, update `docs/architecture/current-state.md`
```

---

## 3. Config Initialization

### Step 1: Create User Settings (~/.claude/settings.json)

```json
{
  "$schema": "https://json-schema.org/claude-code-settings.json",
  "outputStyle": "concise",
  "permissions": {
    "allow": [
      "Skill",
      "Bash(npm run lint)",
      "Bash(npm run test)",
      "Bash(npm run test:*)",
      "Bash(npm run build)",
      "Bash(git status)",
      "Bash(git diff)",
      "Bash(git add .)",
      "Bash(git commit:*)",
      "Bash(git log:*)",
      "Read"
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
    ],
    "defaultMode": "default"
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "tool == \"Bash\" && tool_input.command matches \"(npm run dev|pnpm( run)? dev|yarn dev)\"",
        "hooks": [
          {
            "type": "command",
            "command": "#!/bin/bash\necho '[Hook] BLOCKED: Dev server must run in tmux' >&2\necho '[Hook] Use: tmux new-session -d -s dev \"npm run dev\"' >&2\nexit 1"
          }
        ],
        "description": "Block dev servers outside tmux"
      },
      {
        "matcher": "tool == \"Bash\" && tool_input.command matches \"git push\"",
        "hooks": [
          {
            "type": "command",
            "command": "#!/bin/bash\necho '[Hook] Review changes before push...' >&2\necho '[Hook] Press Enter to continue or Ctrl+C to abort' >&2\nread -r"
          }
        ],
        "description": "Pause before git push"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
        "hooks": [
          {
            "type": "command",
            "command": "#!/bin/bash\ninput=$(cat)\nfile_path=$(echo \"$input\" | jq -r '.tool_input.file_path // \"\"')\nif [ -n \"$file_path\" ] && [ -f \"$file_path\" ]; then\n  if command -v prettier >/dev/null 2>&1; then\n    prettier --write \"$file_path\" 2>&1 | head -5 >&2\n  fi\nfi\necho \"$input\""
          }
        ],
        "description": "Auto-format JS/TS with Prettier"
      },
      {
        "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
        "hooks": [
          {
            "type": "command",
            "command": "#!/bin/bash\ninput=$(cat)\nfile_path=$(echo \"$input\" | jq -r '.tool_input.file_path // \"\"')\nif [ -n \"$file_path\" ] && [ -f \"$file_path\" ]; then\n  console_logs=$(grep -n \"console\\.log\" \"$file_path\" 2>/dev/null || true)\n  if [ -n \"$console_logs\" ]; then\n    echo \"[Hook] WARNING: console.log found in $file_path\" >&2\n    echo \"$console_logs\" | head -5 >&2\n  fi\nfi\necho \"$input\""
          }
        ],
        "description": "Warn about console.log"
      }
    ],
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "#!/bin/bash\nif git rev-parse --git-dir > /dev/null 2>&1; then\n  modified=$(git diff --name-only HEAD 2>/dev/null | grep -E '\\.(ts|tsx|js|jsx)$' || true)\n  for file in $modified; do\n    [ -f \"$file\" ] && grep -q \"console\\.log\" \"$file\" && echo \"[Hook] console.log in $file\" >&2\n  done\nfi"
          }
        ],
        "description": "Final console.log audit"
      }
    ]
  }
}
```

### Step 2: Create MCP Configuration (~/.claude.json)

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "YOUR_GITHUB_PAT_HERE"
      },
      "description": "GitHub operations - PRs, issues, repos"
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"],
      "description": "Persistent memory across sessions"
    },
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"],
      "description": "Chain-of-thought reasoning"
    }
  }
}
```

### Step 3: Create Project Settings (.claude/settings.json)

```json
{
  "$schema": "https://json-schema.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run dev)",
      "Bash(npm run build)",
      "Bash(npm run test:*)"
    ]
  },
  "disabledMcpServers": []
}
```

### Step 4: Create Core Rule Files

**~/.claude/rules/security.md:**
```markdown
# Security Guidelines

## Mandatory Checks Before ANY Commit

- [ ] No hardcoded secrets (API keys, passwords, tokens)
- [ ] All user inputs validated
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS prevention (sanitized HTML)
- [ ] CSRF protection enabled

## Secret Management

```typescript
// NEVER: Hardcoded secrets
const apiKey = "sk-proj-xxxxx"

// ALWAYS: Environment variables
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

## Security Response Protocol

If security issue found:
1. STOP immediately
2. Use **security-reviewer** agent
3. Fix CRITICAL issues before continuing
4. Rotate any exposed secrets
```

**~/.claude/rules/coding-style.md:**
```markdown
# Coding Style

## Immutability (CRITICAL)

ALWAYS create new objects, NEVER mutate:

```javascript
// WRONG: Mutation
user.name = name

// CORRECT: Immutability
return { ...user, name }
```

## File Organization

- Many small files > few large files
- 200-400 lines typical, 800 max
- Organize by feature/domain, not by type

## Error Handling

ALWAYS handle errors:

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('Operation failed:', error)
  throw new Error('User-friendly message')
}
```
```

**~/.claude/rules/testing.md:**
```markdown
# Testing Requirements

## Minimum Coverage: 80%

Test Types (ALL required):
1. **Unit Tests** - Individual functions, utilities
2. **Integration Tests** - API endpoints, database ops
3. **E2E Tests** - Critical user flows (Playwright)

## TDD Workflow (MANDATORY)

1. Write test first (RED)
2. Run test - it should FAIL
3. Write minimal implementation (GREEN)
4. Run test - it should PASS
5. Refactor (IMPROVE)
6. Verify coverage (80%+)
```

---

## 4. First Run Instructions

### Step 1: Verify Installation

```bash
# Check Claude Code is installed
claude --version

# Should output version number
```

### Step 2: Initialize Configuration

```bash
# Create directories
mkdir -p ~/.claude/{agents,skills,commands,rules}

# Copy your settings.json (from above)
# Copy your CLAUDE.md (from above)

# Verify structure
ls -la ~/.claude/
```

### Step 3: Test the Configuration

```bash
# Start Claude Code in your project
cd your-project
claude

# Test that context is loaded
# Ask: "What do you know about this project?"
# Claude should reference your CLAUDE.md content

# Test a hook
# Try: "Run npm run dev"
# Should be blocked with tmux message
```

### Step 4: Verify MCP Servers

```bash
# Within Claude Code session
/mcp

# Should list your configured servers
# Verify github, memory, etc. are available
```

### Step 5: Test Slash Commands

```bash
# Test planning
/plan Add a user authentication feature

# Test TDD
/tdd Create a function to validate email addresses

# Test code review
/code-review
```

### Verification Checklist

- [ ] `claude --version` returns version
- [ ] `~/.claude/settings.json` exists and is valid JSON
- [ ] `~/.claude/CLAUDE.md` exists with your preferences
- [ ] `~/.claude.json` has MCP servers configured
- [ ] Project has `CLAUDE.md` in root
- [ ] `/mcp` shows configured servers
- [ ] Hooks trigger correctly (test with `npm run dev`)
- [ ] Slash commands work (`/plan`, `/tdd`)

---

## Quick Reference Card

### Essential Commands

| Command | Purpose |
|---------|---------|
| `/plan` | Create implementation plan |
| `/tdd` | Test-driven development |
| `/code-review` | Review code quality |
| `/build-fix` | Fix build errors |
| `/clear` | Clear conversation history |
| `/mcp` | Check MCP server status |
| `/context` | View context usage |

### File Locations

| File | Purpose |
|------|---------|
| `~/.claude/settings.json` | User permissions & hooks |
| `~/.claude/CLAUDE.md` | Personal preferences |
| `~/.claude.json` | MCP server config |
| `.claude/settings.json` | Project config |
| `CLAUDE.md` | Project context |

### Hook Exit Codes

| Code | Behavior |
|------|----------|
| `0` | Success, proceed |
| `2` | Block action |
| Other | Warn, continue |

---

## Appendix: Quick Setup Script

Create `setup-claude-code.sh`:

```bash
#!/bin/bash

echo "Setting up Claude Code configuration..."

# Create user-level directories
mkdir -p ~/.claude/{agents,skills,commands,rules,hooks}

# Create settings.json with three-tier permissions
cat > ~/.claude/settings.json << 'EOF'
{
  "$schema": "https://json-schema.org/claude-code-settings.json",
  "outputStyle": "concise",
  "permissions": {
    "allow": ["Skill", "Bash(npm run:*)", "Bash(git status)", "Bash(git diff)", "Read"],
    "ask": ["Bash(git push)", "Bash(npm install:*)", "Edit(package.json)"],
    "deny": ["Bash(rm -rf:*)", "Bash(git push --force:*)", "Read(./.env*)"],
    "defaultMode": "default"
  }
}
EOF

# Create lean CLAUDE.md (router pattern)
cat > ~/.claude/CLAUDE.md << 'EOF'
# Claude Code Configuration

## Core Principles
- Plan before executing
- Write tests first (TDD)
- Never compromise on security
- Prefer immutability

## Modular Rules
| Rule | Location |
|------|----------|
| Security | `~/.claude/rules/security.md` |
| Code Style | `~/.claude/rules/coding-style.md` |
| Testing | `~/.claude/rules/testing.md` |

## Self-Maintenance
After significant changes, update `docs/architecture/current-state.md`
EOF

echo "User-level setup complete at ~/.claude/"
echo ""
echo "For each project, run:"
echo "  mkdir -p .claude/{commands,skills,rules}"
echo "  mkdir -p docs/architecture/{decisions,plans}"
echo "  touch CLAUDE.md docs/architecture/current-state.md"
```

### Project Setup Script

Create `setup-project-claude.sh` for new projects:

```bash
#!/bin/bash

echo "Setting up Claude Code for this project..."

# Create project directories
mkdir -p .claude/{commands,skills,rules}
mkdir -p docs/architecture/{decisions,plans}

# Create lean project CLAUDE.md
cat > CLAUDE.md << 'EOF'
# Project: [PROJECT NAME]

## Overview
[One sentence description]

**Stack:** [Framework], [Language], [Database]

## Commands
- `npm run dev` - Dev server (use tmux!)
- `npm test` - Run tests (MUST pass)
- `npm run lint` - Linter
- `npm run build` - Build

## Workflows
| Task | Resource |
|------|----------|
| Testing | `.claude/rules/testing.md` |
| Security | `.claude/rules/security.md` |

## Rules
1. TDD required - tests before code
2. Plan first if touching >2 files
3. Update `docs/architecture/current-state.md` after changes
EOF

# Create current-state.md
cat > docs/architecture/current-state.md << 'EOF'
# Current System State

*Last updated: [DATE]*

## Active Features
- [ ] Initial setup

## Recent Changes
- [DATE]: Project initialized

## Next Steps
1. Define core architecture
EOF

echo "Project setup complete!"
echo "Edit CLAUDE.md and docs/architecture/current-state.md"
```

---

## Sources & References

- [Claude Code Official Documentation](https://code.claude.com/docs/en/overview)
- [Claude Code Hooks Reference](https://code.claude.com/docs/en/hooks)
- [Claude Code MCP Integration](https://code.claude.com/docs/en/mcp)
- [Anthropic Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [everything-claude-code Repository](https://github.com/affaanmustafa/everything-claude-code)

---

*This guide synthesizes patterns from production configurations and official documentation. Customize based on your workflow and project needs.*
