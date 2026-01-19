---
name: implement
description: Execute an implementation plan from start to finish. Takes a plan file, works through tasks, runs tests/lints, and commits at logical points. Use when user says "implement", "execute plan", "build this", or has a plan file to work through.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Task
  - TodoWrite
  - AskUserQuestion
  - Skill
---

# Implement

Execute an implementation plan, working through tasks systematically until complete.

## Arguments

- `/implement <plan-path>` - Execute the plan at the given path
- `/implement <plan-path> --map` - Execute plan, include map updates in commits

**Examples:**
```
/implement docs/plans/user-auth-oauth-db-schema.md
/implement docs/plans/notifications-email.md --map
```

## Process

### Step 1: Load and Validate Plan

1. Read the plan file at the provided path
2. Check plan status:
   - If `pending`: Update to `in_progress`, begin from first task
   - If `in_progress`: Resume from first unchecked task
   - If `completed`: Inform user, ask if they want to re-run
3. Check for blockers:
   - If plan has `Blocked by: X`, check if X is complete
   - If blocked: warn user and ask whether to proceed anyway
4. Read the linked spec (from the `Spec:` field) for context

### Step 2: Explore Codebase via Code Maps

Use `MAP.md` files for efficient codebase exploration. This minimizes context usage and speeds up navigation.

**Exploration strategy:**

1. **Start at the root** - Read the top-level `MAP.md` (or `src/MAP.md`) to understand project structure
2. **Drill down as needed** - Follow the hierarchy into relevant folders
3. **Read files only when necessary** - MAP.md tells you what's in each file; only read the actual file when you need implementation details

**When exploring:**

| Need | Action |
|------|--------|
| Understand folder structure | Read `MAP.md` at that level |
| Find where something lives | Check MAP.md folder descriptions |
| Understand a file's exports | Check MAP.md file entry (exports, types) |
| See actual implementation | Read the source file |

**If MAP.md files don't exist:**
- Navigate manually (Glob, Grep, Read)
- Consider running `/map` on relevant folders to create them for future agents

**Benefits:**
- Lower context usage (MAP.md is much smaller than reading all files)
- Faster orientation (hierarchical overview vs. file-by-file exploration)
- Better decisions (understand existing patterns before writing code)

**External library documentation:**

If the `context7` skill/tool is available, use it to fetch documentation for external libraries and APIs:

- Use context7 when you need to understand a library's API
- Use context7 when implementing integrations with external services
- Use context7 when unsure about correct usage patterns for dependencies

This provides up-to-date documentation without relying on training data.

**Frontend and UI work:**

If the `frontend-design` skill is available, use it for any frontend or interface tasks:

- Use frontend-design when creating UI components
- Use frontend-design when building pages or layouts
- Use frontend-design when implementing user-facing features
- Use frontend-design for styling and visual design work

This ensures high-quality, production-grade frontend code that avoids generic patterns.

### Step 3: Discover Project Checks

Discover what validation is available in this project. Check for:

| File/Pattern | Check Command | Purpose |
|--------------|---------------|---------|
| `package.json` with `test` script | `npm test` | Tests |
| `package.json` with `lint` script | `npm run lint` | Linting |
| `package.json` with `typecheck` script | `npm run typecheck` | Type checking |
| `tsconfig.json` | `npx tsc --noEmit` | TypeScript types |
| `.eslintrc*` or `eslint.config.*` | `npx eslint .` | ESLint |
| `pyproject.toml` or `pytest.ini` | `pytest` | Python tests |
| `Makefile` with `test` target | `make test` | Tests |
| `Makefile` with `lint` target | `make lint` | Linting |
| `.github/workflows/*.yml` | (reference) | CI checks to mirror |

Store discovered checks for use before commits. Note: not all projects have all checks - only run what exists.

### Step 4: Implement Tasks

Work through tasks in order. For each task:

1. **Mark task in progress** in both:
   - The plan file: `- [ ]` → `- [~]` (optional, for visibility)
   - Your TodoWrite list

2. **Implement the task**
   - Use MAP.md files to navigate and understand existing code
   - Write code, create files, make changes
   - Follow the spec for context and requirements
   - Match existing patterns and conventions found in the codebase

3. **Handle unexpected work**
   - If new tasks arise during implementation, ADD them to the plan file
   - Insert them in logical order (after current task or at end)
   - Add them to your TodoWrite list too

4. **Mark task complete**
   - Plan file: `- [ ]` → `- [x]`
   - TodoWrite: mark completed

5. **Capture learnings**
   - If you discover gotchas, edge cases, or important notes
   - Append them to `docs/LEARNINGS.md` (see Learnings Format below)
   - Also note in the plan's `## Notes` section for immediate context

6. **Commit at logical points**
   - After completing a coherent chunk of work
   - Before moving to a different area/component
   - When tests pass for a meaningful unit

### Step 5: Pre-Commit Validation

Before each commit:

1. **Run discovered checks** (in order):
   - Type checking (fast feedback)
   - Linting
   - Tests

2. **If checks fail:**
   - Fix the issues
   - Re-run checks until passing
   - If unable to fix: note in plan, ask user

3. **If checks pass:**
   - Invoke `/commit` (or `/commit --map` if `--map` flag was passed)

### Step 6: Update Plan Metadata

Keep the plan file updated:

```markdown
**Status:** in_progress → completed
**Started:** 2026-01-18
**Completed:** 2026-01-18
```

Update `## Files to Create/Modify` if you touched files not originally listed.

### Step 7: Monitor Context Usage

**At ~90% context usage:**
1. Stop implementation at the current logical point
2. Ensure current work is committed
3. Update plan with progress (checked tasks, notes)
4. Suggest user run `/handoff` to continue in a new session

Message: "I'm approaching context limits. Current progress is committed and the plan is updated. Run `/handoff` to create a handoff document, then `/resume` in a new session to continue."

## Stopping Conditions

Stop implementation when ANY of these occur:

| Condition | Action |
|-----------|--------|
| **Plan complete** | Update status to `completed`, report summary |
| **90% context** | Commit, update plan, suggest handoff |
| **Blocked** | Note blocker in plan, ask user (use sparingly) |
| **Tests failing & can't fix** | Commit working code, note issue, ask user |
| **Unclear requirement** | Check spec first, then ask user if still unclear |

## Commit Frequency Guidelines

Commit after:
- Each task that represents a complete, testable unit
- Before switching to a different component/area
- After fixing a complex issue (preserve the fix)
- When you have passing tests for new functionality

Don't commit:
- Half-finished features that break tests
- Code that doesn't compile/typecheck
- After every tiny change (batch related small changes)

## Asking Questions

Use `AskUserQuestion` sparingly. Before asking:

1. Check the spec - answer might be there
2. Check the plan's notes - might have context
3. Make a reasonable decision if it's low-risk
4. Only ask if the decision significantly affects the implementation

Good reasons to ask:
- Ambiguous requirement with multiple valid interpretations
- Discovered a design issue that affects the spec
- External dependency or access needed
- Significant scope change discovered

## Output

When implementation completes (or stops), report:

1. **Summary:** What was accomplished
2. **Tasks completed:** Count and list
3. **Tasks remaining:** If any
4. **Commits made:** Short list with messages
5. **Issues/notes:** Anything discovered during implementation
6. **Next steps:** What to do next (if incomplete)

## Example Session

```
/implement docs/plans/auth-db-schema.md --map
```

Agent:
1. Reads plan, updates status to `in_progress`
2. Reads linked spec for context
3. Reads root MAP.md to understand project structure
4. Discovers project has `npm test`, `npm run lint`, TypeScript
5. Works through tasks:
   - Creates migration file ✓
   - Creates User model ✓
   - Creates Session model ✓
   - Runs lint, typecheck, tests - all pass
   - Commits: "feat(auth): add user and session database schema"
   - Continues with remaining tasks...
6. All tasks complete
7. Updates plan status to `completed`
8. Reports summary

## Learnings Format

Append learnings to `docs/LEARNINGS.md`. Create the file if it doesn't exist.

**Format for each learning entry:**

```markdown
## [YYYY-MM-DD] Category: Brief Title

**Context:** [What you were working on]
**Plan:** [Link to plan if applicable]

**Learning:**
[Description of the insight, gotcha, or pattern discovered]

**Actionable:** [yes/no]
**Action type:** [skill-opportunity | script-opportunity | docs-update | pattern | gotcha]
**Suggested action:** [If actionable, what should be done]

---
```

**Categories:**
| Category | When to use |
|----------|-------------|
| `Gotcha` | Trap or pitfall to avoid |
| `Pattern` | Reusable approach discovered |
| `Skill Opportunity` | Repeated multi-step process that could be a skill |
| `Script Opportunity` | Manual work that could be automated |
| `Docs Gap` | Missing documentation that caused friction |

**Example entry:**

```markdown
## [2026-01-18] Gotcha: API returns null not undefined

**Context:** Implementing user fetch in auth service
**Plan:** [auth-db-schema](plans/user-auth-oauth-db-schema.md)

**Learning:**
The user API returns `null` for missing optional fields, not `undefined`.
This breaks our standard `user?.email` pattern. Need explicit null checks.

**Actionable:** yes
**Action type:** docs-update
**Suggested action:** Add to CLAUDE.md API conventions section

---
```

## Plan File Format Reference

Expected plan structure:

```markdown
# Plan: [Name]

**Spec:** [link to spec](../specs/feature.md)
**Status:** pending | in_progress | completed
**Depends on:** other-plan (optional)
**Blocked by:** other-plan (optional)
**Started:** (added by agent)
**Completed:** (added by agent)

## Overview
Context for this plan.

## Tasks
- [ ] First task
- [ ] Second task
- [x] Completed task

## Files to Create/Modify
- `path/to/file.ts` - description

## Notes
Implementation notes, gotchas, learnings.
```
