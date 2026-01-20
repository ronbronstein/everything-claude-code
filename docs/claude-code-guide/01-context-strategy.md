# 1. Context Strategy: CLAUDE.md Files

[← Back to Index](./README.md) | [Next: Permissions →](./02-permissions.md)

---

## Overview

CLAUDE.md is a special file that Claude automatically reads at the start of every conversation. It provides persistent project context - the instructions you'd give a senior developer joining your team.

---

## The Four-File Hierarchy

**Verified:** Official documentation confirms this hierarchy at code.claude.com/docs/en/memory.

| File Location | Scope | Purpose |
|--------------|-------|---------|
| `~/.claude/CLAUDE.md` | User-level | Personal preferences across all projects |
| `./CLAUDE.md` or `./.claude/CLAUDE.md` | Project-level | Shared team context, checked into git |
| `./CLAUDE.local.md` | Project-local | Personal project overrides, git-ignored |
| `./subdir/CLAUDE.md` | Subfolder-level | Domain-specific context (loaded when Claude reads files in that directory) |

### Precedence Rules

From official docs:
> "User-level rules are loaded before project rules, giving project rules higher priority."

Claude Code recurses up from the current directory to (but not including) root `/`, reading all CLAUDE.md files found.

---

## What to Include

### Essential Elements

1. **Project Overview** - Stack, architecture, purpose (2-3 sentences)
2. **Build Commands** - The exact commands to build, test, lint, deploy
3. **Critical Rules** - Non-negotiable constraints (security, style)
4. **File Structure** - Architecture map to prevent hallucination

### Keep It Lean

Official guidance suggests keeping CLAUDE.md files focused. Community consensus recommends:
- **~200 lines optimal** for project-level
- A 2,800-line file wastes ~62% of context budget

---

## Community Patterns

> **Note:** The following patterns are community best practices, not official Anthropic guidance.

### Router Pattern *(Community Pattern)*

Keep CLAUDE.md minimal and link to detailed resources:

```markdown
# Project: MyApp

## Overview
Real-time collaboration tool. Stack: Next.js, TypeScript, Supabase.

## Commands
- `npm run dev` - Dev server (use tmux!)
- `npm test` - Run tests (MUST pass)
- `npm run build` - Production build

## Workflows
*Read these only when performing the specific task:*

| Task | Resource |
|------|----------|
| Testing | `.claude/rules/testing.md` |
| Security | `.claude/rules/security.md` |
| Database | `.claude/skills/db-migration.md` |

## Rules
1. TDD required - tests before code
2. No hardcoded secrets
3. Update `docs/architecture/current-state.md` after changes
```

**Why This Works:**
- Every line in CLAUDE.md consumes tokens on every conversation
- Detailed workflows in `.claude/skills/` and `.claude/rules/` load on-demand
- Saves thousands of tokens per session

### Self-Maintenance Pattern *(Community Pattern)*

Create external "long-term memory" that survives session resets:

```
docs/architecture/
├── current-state.md      # Auto-updated by Claude
├── decisions/            # Architecture Decision Records
│   ├── 001-auth-system.md
│   └── 002-database-choice.md
└── plans/                # Implementation plans
```

**current-state.md Template:**

```markdown
# Current System State

*Last updated: [DATE] by Claude*

## Active Features
- [Feature 1]: [Status - Complete/In Progress/Planned]

## Recent Changes
- [DATE]: [Change description]

## Known Issues
- [ ] [Issue 1]

## Next Steps
1. [Next task]
```

**Enforcement (add to CLAUDE.md):**
```markdown
## Self-Maintenance Rule
After completing ANY significant change:
1. Update `docs/architecture/current-state.md`
2. Create ADR in `docs/architecture/decisions/` if architectural
```

---

## Example: Minimal Project CLAUDE.md

```markdown
# Project: [NAME]

## Overview
[One sentence description]

**Stack:** [Framework], [Language], [Database]

## Commands
- `npm run dev` - Dev server (use tmux!)
- `npm test` - Run tests (MUST pass before commit)
- `npm run lint` - Linter
- `npm run build` - Build

## Architecture
```
src/
├── app/           # App router
├── components/    # UI components
├── lib/           # Utilities
└── types/         # TypeScript types
```

## Rules
1. TDD required
2. No hardcoded secrets
3. 80% test coverage minimum

## Workflows
| Task | Resource |
|------|----------|
| Testing | `.claude/rules/testing.md` |
| Security | `.claude/rules/security.md` |
```

---

## Example: User-Level CLAUDE.md

Place at `~/.claude/CLAUDE.md`:

```markdown
# User Preferences

## Code Style
- No emojis in code or comments
- Prefer immutability - never mutate objects/arrays
- Many small files (200-400 lines, 800 max)

## Git
- Conventional commits: feat:, fix:, refactor:, docs:, test:
- Test locally before committing
- Small, focused commits

## Testing
- TDD: Write tests first
- 80% minimum coverage

## Workflows
| Rule | Location |
|------|----------|
| Security | `~/.claude/rules/security.md` |
| Code Style | `~/.claude/rules/coding-style.md` |
| Testing | `~/.claude/rules/testing.md` |
```

---

## Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad | Better Approach |
|--------------|--------------|-----------------|
| 1000+ line CLAUDE.md | Wastes context on every message | Use router pattern, link to files |
| Duplicate rules in user + project | Confusion, conflicts | User = defaults, project = overrides |
| Vague instructions | "Write good code" | Be specific: "Use immutable patterns" |
| No build commands | Claude guesses wrong | Document exact commands |

---

## Verification Status

| Claim | Status | Source |
|-------|--------|--------|
| Four-file hierarchy | ✅ Verified | Official docs |
| Precedence rules | ✅ Verified | Official docs |
| Router pattern | ⚠️ Community | Not in official docs |
| Self-maintenance | ⚠️ Community | Not in official docs |

---

[← Back to Index](./README.md) | [Next: Permissions →](./02-permissions.md)
