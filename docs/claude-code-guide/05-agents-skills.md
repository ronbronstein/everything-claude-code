# 5. Agents, Skills & Custom Commands

[← Back to Index](./README.md) | [Previous: MCP Servers](./04-mcp-servers.md) | [Next: Workflow Patterns →](./06-workflow-patterns.md)

---

## Overview

Claude Code supports specialized subagents, reusable skills, and custom slash commands to extend functionality and enable delegation.

---

## Agents: Specialized Subagents

Agents are focused problem-solvers with limited scope and specific tools.

### Agent Frontmatter Format

**Verified:** From official plugins repository and community examples.

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability
tools: Read, Grep, Glob, Bash
model: opus
---

You are a senior code reviewer. Your role is to...
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Identifier for the agent |
| `description` | Yes | What the agent does |
| `tools` | Yes | Comma-separated: Read, Write, Edit, Grep, Glob, Bash, WebFetch, etc. |
| `model` | No | `sonnet`, `opus`, `haiku`, or full model ID |

### Agent Examples

**Planner Agent** (`~/.claude/agents/planner.md`):

```markdown
---
name: planner
description: Expert planning specialist for complex features
tools: Read, Grep, Glob
model: opus
---

You are an expert planning specialist. Create comprehensive, actionable plans.

## Planning Process

1. **Requirements Analysis** - Understand the request completely
2. **Architecture Review** - Analyze existing code structure
3. **Step Breakdown** - Create detailed implementation steps
4. **Risk Assessment** - Identify potential issues

## Output Format

# Implementation Plan: [Feature Name]

## Overview
[2-3 sentence summary]

## Steps
1. [Step with file path and specific action]
2. [Step with file path and specific action]

## Risks
- [Risk and mitigation]
```

**Code Reviewer Agent** (`~/.claude/agents/code-reviewer.md`):

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, maintainability
tools: Read, Grep, Glob, Bash
model: opus
---

When invoked:
1. Run `git diff` to see recent changes
2. Focus on modified files
3. Begin review immediately

## Review Checklist
- [ ] Code is simple and readable
- [ ] No exposed secrets or API keys
- [ ] Input validation implemented
- [ ] Error handling present
- [ ] Tests cover changes
```

**TDD Guide Agent** (`~/.claude/agents/tdd-guide.md`):

```markdown
---
name: tdd-guide
description: Test-Driven Development specialist
tools: Read, Write, Edit, Bash, Grep
model: sonnet
---

## TDD Workflow

1. **RED** - Write a failing test first
2. **Verify** - Run test, confirm it FAILS
3. **GREEN** - Write minimal code to pass
4. **Verify** - Run test, confirm it PASSES
5. **REFACTOR** - Improve code, keep tests passing
6. **Coverage** - Verify 80%+ coverage
```

---

## Skills: Reusable Workflows

Skills are workflow definitions that can be invoked by commands or agents.

### Skill Format

**Verified:** From official anthropics/skills repository.

Skills are stored in `.claude/skills/{skill-name}/SKILL.md`:

```markdown
---
name: tdd-workflow
description: Test-driven development workflow
---

# TDD Workflow

## When to Use
- New features
- Bug fixes
- Refactoring with safety net

## Steps
1. Write test describing expected behavior
2. Run test - should FAIL
3. Write minimal implementation
4. Run test - should PASS
5. Refactor if needed
6. Verify coverage >= 80%

## Example
```typescript
// 1. Write test first
test('validates email format', () => {
  expect(isValidEmail('test@example.com')).toBe(true)
  expect(isValidEmail('invalid')).toBe(false)
})

// 2. Then implement
function isValidEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}
```
```

---

## Custom Slash Commands

> **Important Correction:** Commands like `/plan`, `/tdd`, `/code-review` are **NOT built-in**. They are custom commands you create.

### Built-in Commands (Official)

| Command | Purpose |
|---------|---------|
| `/help` | Show help |
| `/config` | Configuration |
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
| `/doctor` | Diagnostics |
| `/tasks` | Background tasks |
| `/vim` | Vim mode |

### Creating Custom Commands

Place Markdown files in `.claude/commands/`:

**`/plan` Command** (`.claude/commands/plan.md`):

```markdown
---
description: Create an implementation plan before coding
---

# Planning Mode

Create a detailed implementation plan. **Do not write code yet.**

## Process

1. **Restate Requirements** - Clarify what needs to be built
2. **Identify Files** - List files that will be modified/created
3. **Create Steps** - Break down into specific steps
4. **Identify Risks** - Surface potential issues
5. **Wait for Approval** - MUST receive user confirmation before proceeding

## Output Format

# Plan: [Feature Name]

## Requirements
- [Requirement 1]

## Files to Modify
- `path/to/file.ts` - [what changes]

## Implementation Steps
1. [Step 1]
2. [Step 2]

## Risks
- [Risk and mitigation]

---
**Awaiting approval to proceed.**
```

**`/tdd` Command** (`.claude/commands/tdd.md`):

```markdown
---
description: Implement feature using test-driven development
---

# TDD Implementation

Use strict test-driven development:

1. **Write test first** (RED)
   - Describe expected behavior
   - Test should FAIL initially

2. **Verify failure**
   - Run: `npm test`
   - Confirm test fails for the right reason

3. **Implement minimally** (GREEN)
   - Write just enough code to pass
   - No extra features

4. **Verify success**
   - Run: `npm test`
   - All tests should pass

5. **Refactor** (IMPROVE)
   - Clean up code
   - Keep tests passing

6. **Check coverage**
   - Run: `npm run test:coverage`
   - Must be >= 80%

**Do not skip steps. Do not write mocks unless explicitly asked.**
```

**`/code-review` Command** (`.claude/commands/code-review.md`):

```markdown
---
description: Review recent code changes
---

# Code Review

Review the most recent changes:

1. Run `git diff HEAD~1` to see changes
2. Analyze each modified file

## Review Checklist

- [ ] Code is readable and simple
- [ ] No hardcoded secrets
- [ ] Error handling is present
- [ ] Input validation where needed
- [ ] Tests cover the changes
- [ ] No console.log in production code

## Output Format

# Code Review: [files reviewed]

## Summary
[Overall assessment]

## Issues Found
- **[Severity]**: [Issue description] in `file:line`

## Suggestions
- [Improvement suggestion]

## Verdict
[APPROVED / NEEDS CHANGES]
```

### Command Arguments

Use `$ARGUMENTS` placeholder:

```markdown
---
description: Generate tests for a specific file
---

Generate comprehensive tests for: $ARGUMENTS

Include:
- Unit tests for all functions
- Edge cases
- Error scenarios
```

Usage: `/generate-tests src/utils/auth.ts`

---

## Skills + Commands Merged (v2.1.0)

As of v2.1.0, skills can be invoked using `/` prefix directly.

---

## Directory Structure

```
~/.claude/
├── agents/
│   ├── planner.md
│   ├── code-reviewer.md
│   ├── tdd-guide.md
│   └── security-reviewer.md
├── skills/
│   ├── tdd-workflow/
│   │   └── SKILL.md
│   └── coding-standards.md
└── commands/
    ├── plan.md
    ├── tdd.md
    └── code-review.md

.claude/                    # Project-level
├── commands/
│   └── deploy.md          # Project-specific commands
└── skills/
    └── db-migration.md    # Project-specific skills
```

---

## Parallel Agent Execution

Launch multiple agents simultaneously for independent tasks:

```
# GOOD: Parallel execution
Launch 3 agents in parallel:
1. Security review of auth module
2. Performance review of cache system
3. Type checking of API routes

# BAD: Sequential when unnecessary
First agent 1, then agent 2, then agent 3
```

---

## When to Use Each

| Need | Use |
|------|-----|
| Specialized task delegation | Agent |
| Reusable workflow pattern | Skill |
| Quick invokable action | Command |
| One-off complex analysis | Direct prompt with planning |

---

## Verification Status

| Claim | Status | Source |
|-------|--------|--------|
| Agent frontmatter format | ✅ Verified | Official plugins repo |
| Skill format | ✅ Verified | Official skills repo |
| Built-in commands list | ✅ Verified | Official docs |
| Custom commands in .claude/commands/ | ✅ Verified | Official docs |
| /plan, /tdd as built-in | ❌ Incorrect | These are custom, not built-in |

---

[← Back to Index](./README.md) | [Previous: MCP Servers](./04-mcp-servers.md) | [Next: Workflow Patterns →](./06-workflow-patterns.md)
