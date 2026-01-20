# 6. Workflow Patterns for Senior-Level Development

[← Back to Index](./README.md) | [Previous: Agents & Skills](./05-agents-skills.md) | [Next: Advanced Features →](./07-advanced-features.md)

---

## Overview

These workflow patterns are based on official Anthropic engineering posts and validated community practices for using Claude Code as a senior development partner.

---

## The Explore → Plan → Code → Commit Workflow

**Source:** Anthropic Engineering best practices

This is the recommended workflow for complex features:

### 1. Explore Phase

```
"Read the codebase to understand how [feature area] works.
Don't write any code yet."
```

Use `Shift+Tab` to enter **Plan Mode** (restricts code modifications).

### 2. Plan Phase

```
"Create a detailed plan for implementing [feature].
Use ultrathink for deeper analysis.
Don't write code until I approve."
```

**Planning triggers:**
- `"think"` - Standard thinking
- `"think hard"` - Extended thinking
- `"think harder"` - More extended
- `"ultrathink"` - Maximum reasoning depth

### 3. Review & Refine

- Question assumptions in the plan
- Identify missing requirements
- Validate the approach
- Request alternatives if needed

### 4. Code Phase

```
"Proceed with Phase 1 of the plan."
```

Exit Plan Mode (`Shift+Tab` again) to enable code writing.

### 5. Commit Phase

```
"Create a commit with these changes."
```

Or for full PR workflow:

```
"Create a PR with these changes."
```

---

## TDD Workflow

**Verified:** From Anthropic engineering recommendations

### The RED → GREEN → REFACTOR Loop

```
1. RED    - Write a failing test
2. Verify - Confirm test FAILS
3. GREEN  - Write minimal code to pass
4. Verify - Confirm test PASSES
5. REFACTOR - Improve code
6. Verify - Confirm tests still pass
```

### Effective TDD Prompts

**Starting TDD:**
```
"Use TDD to implement [feature].
Write tests first, don't create mocks.
Confirm each test fails before implementing."
```

**Preventing Over-Implementation:**
```
"Write ONLY enough code to make this specific test pass.
Don't anticipate future requirements."
```

**Verification:**
```
"Run the tests. If any fail, fix them before continuing."
```

### TDD Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad | Better Approach |
|--------------|--------------|-----------------|
| Writing implementation first | Defeats TDD purpose | Insist on test-first |
| Creating mocks by default | Tests become brittle | Use real implementations |
| Testing implementation details | Tight coupling | Test behavior, not internals |
| Skipping the RED phase | Missing validation | Always verify failure first |

---

## Multi-Claude Pattern

**Source:** Community pattern validated by power users

Run multiple Claude Code sessions in parallel on different features:

### Setup with Git Worktrees

```bash
# Create worktrees for parallel development
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b
git worktree add ../project-bugfix bugfix-branch

# Run Claude in each
cd ../project-feature-a && claude
cd ../project-feature-b && claude
cd ../project-bugfix && claude
```

### Coordination Strategy

1. **Keep a tracking document** for active sessions
2. **Assign clear scope** to each Claude instance
3. **Merge frequently** to avoid conflicts
4. **Use one Claude** for final integration review

### When to Use Multi-Claude

| Scenario | Single Claude | Multi-Claude |
|----------|---------------|--------------|
| Sequential tasks | ✅ | ❌ |
| Independent features | ❌ | ✅ |
| Large refactoring | ✅ | ❌ |
| Parallel bug fixes | ❌ | ✅ |
| Code review + coding | ❌ | ✅ |

---

## Context Management Discipline

**Critical:** From community consensus on maintaining Claude effectiveness

### The Golden Rule

> **Use `/clear` frequently between unrelated tasks.**

### Context Monitoring

```
/context    # Check current context usage
```

### Compaction Strategy

| Context Level | Action |
|---------------|--------|
| < 50% | Continue working |
| 50-70% | Consider `/compact` at logical breakpoint |
| 70-85% | Manual `/compact` recommended |
| > 85% | `/compact` or `/clear` required |

**Manual compaction with instructions:**
```
/compact only keep the authentication decisions and current task
```

### Why This Matters

Community reports consistently indicate:
- Claude becomes less effective after auto-compaction
- Manual compaction at logical breakpoints preserves more relevant context
- Starting fresh often produces better results than continuing degraded context

---

## Token Efficiency Patterns

### Lean CLAUDE.md

| Size | Impact |
|------|--------|
| ~200 lines | Optimal |
| ~500 lines | Acceptable |
| 1000+ lines | Significant waste |
| 2800 lines | ~62% context wasted |

### Specific References

```
# BAD: Vague
"Find where authentication happens"

# GOOD: Specific
"Read src/lib/auth/session.ts to understand session handling"
```

### Batch Related Changes

```
# BAD: Multiple small requests
"Add error handling to login"
"Add error handling to logout"
"Add error handling to refresh"

# GOOD: Single batched request
"Add consistent error handling to all auth functions in src/lib/auth/"
```

### Disable Unused MCPs

Before starting a session:
```json
{
  "disabledMcpServers": ["unused-server-1", "unused-server-2"]
}
```

---

## Writer/Reviewer Separation Pattern

**Source:** Anthropic engineering recommendation

Use separate Claude instances for writing and reviewing:

### Workflow

1. **Claude A** (Writer): Implements the feature
2. **Claude B** (Reviewer): Reviews the implementation in separate terminal
3. **Claude C** (Integrator): Reads code + feedback, makes refinements

### Why It Works

- Prevents confirmation bias
- Fresh perspective catches more issues
- Simulates real team code review

### Implementation

```bash
# Terminal 1: Writer
cd project && claude
> "Implement user authentication"

# Terminal 2: Reviewer (after writer finishes)
cd project && claude
> "Review the recent auth changes. Be critical."

# Terminal 1: Integrator
> "Here's the review feedback: [paste]. Address these issues."
```

---

## Subagent Verification Pattern

After Claude implements something, use a subagent to verify:

```
"Launch a security-reviewer agent to analyze the auth implementation.
Then launch a code-reviewer agent to check code quality.
Run these in parallel."
```

This prevents Claude from being blind to its own mistakes.

---

## Recommended Command Sequence

For complex features:

```
/plan          → Create implementation plan
[review plan]
/tdd           → Implement with tests first
/build-fix     → Fix any build errors
/code-review   → Review implementation
[commit]
```

---

## Summary: Workflow Selection Guide

| Task Type | Recommended Workflow |
|-----------|---------------------|
| Complex feature | Explore → Plan → Code → Commit |
| Bug fix | TDD (write failing test first) |
| Refactoring | Plan Mode analysis, then targeted changes |
| Multiple independent features | Multi-Claude with worktrees |
| Security-sensitive | Writer/Reviewer separation |
| Performance optimization | Plan with ultrathink, measure before/after |

---

[← Back to Index](./README.md) | [Previous: Agents & Skills](./05-agents-skills.md) | [Next: Advanced Features →](./07-advanced-features.md)
