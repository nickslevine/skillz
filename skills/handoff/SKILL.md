---
name: handoff
description: Create a handoff document when running out of context or ending a session. Captures all necessary context for the next agent to continue the work seamlessly.
allowed-tools:
  - Read
  - Write
  - Glob
  - Bash
  - AskUserQuestion
  - Task
---

# Handoff

Create a concise handoff document that gives the next agent everything they need to continue the work without losing context or repeating effort.

## When to Use

- Context window is filling up
- Ending a session with work in progress
- Handing off to a different agent or session
- User explicitly requests a handoff

## Arguments

The user may optionally provide guidance via `$ARGUMENTS`:

**Examples:**
- `/handoff` - No guidance, create a general handoff
- `/handoff focus on the sql integration subtask` - Emphasize SQL integration work
- `/handoff the auth flow is tricky, make sure to document it well` - Extra detail on auth
- `/handoff sql-integration` - Just a short description for the filename

If the argument looks like guidance/instructions (contains spaces and reads like a sentence), use it to:
1. Focus the handoff content on that area
2. Provide extra detail on the specified topic
3. Derive a short description for the filename from the guidance

If the argument is just a short kebab-case slug, use it directly as the filename description.

## Process

### Step 1: Parse Arguments

Check `$ARGUMENTS`:
- **Empty:** Derive a short slug from the main task/work in the current session (e.g., if working on user authentication, use `auth-flow`)
- **Short slug** (e.g., `sql-integration`): Use as filename description
- **Guidance sentence** (e.g., `focus on the sql integration subtask`):
  - Extract a short slug for the filename (e.g., `sql-integration`)
  - Use the full guidance to focus the handoff content

### Step 2: Determine Filename

Generate a filename with this format:
```
HANDOFF-YYYY-MM-DD-short-description.md
```

Example: `HANDOFF-2026-01-18-auth-refactor.md`

### Step 3: Check for Existing File

Before writing, check if the file already exists in `./docs/handoffs/`. If it does, append a number (e.g., `-2`, `-3`) to make it unique.

### Step 4: Gather Context

Review the current session to collect:

1. **Source documents** - Any specs, plans, or docs being worked from
2. **Files touched** - Files read, created, or modified
3. **Current todo state** - If a todo list exists, capture it
4. **Key learnings** - Insights that save the next agent time

**If guidance was provided:** Pay special attention to the area the user highlighted. Provide extra detail, context, and learnings for that specific topic.

### Step 5: Reflect and Capture Learnings

Before writing the handoff, reflect on the session and capture learnings to `docs/LEARNINGS.md`:

**Ask yourself:**
- What gotchas or pitfalls did I encounter?
- What patterns or approaches worked well?
- Did I repeat any manual work that could be automated?
- Was there a multi-step process that could become a skill?
- Was there missing documentation that caused friction?

**For each significant learning, append to `docs/LEARNINGS.md`:**

```markdown
## [YYYY-MM-DD] Category: Brief Title

**Context:** [What you were working on]
**Plan:** [Link to plan if applicable]

**Learning:**
[Description of the insight]

**Actionable:** [yes/no]
**Action type:** [skill-opportunity | script-opportunity | docs-update | pattern | gotcha]
**Suggested action:** [If actionable, what should be done]

---
```

This builds the knowledge base for `/self-improve` to act on later.

### Step 6: Write the Handoff Document

Write to `./docs/handoffs/[filename]` using the template below.

If the user provided focus guidance, ensure that topic gets prominent coverage in the relevant sections (especially Learnings & Gotchas, Next Steps, and any section where that work is documented).

## Handoff Document Template

```markdown
# Handoff: [Brief Title]

**Date:** [YYYY-MM-DD]
**Session:** [Brief description of what was being worked on]

## Task Overview

[1-2 sentence summary of the overall goal]

## Source Documents

- [Link to any specs, plans, or reference docs being used]
- [Include file paths relative to repo root]

## Current Status

### Completed
- [x] Task that was finished
- [x] Another completed task

### In Progress
- [ ] Task currently being worked on
- [ ] State of partial completion

### Remaining
- [ ] Task not yet started
- [ ] Another pending task

## Key Files & Folders

| Path | Purpose |
|------|---------|
| `path/to/file` | Why this file matters |
| `path/to/folder/` | What's in this folder |

## Decisions Made

| Decision | Rationale |
|----------|-----------|
| Chose X over Y | Because of Z reason |
| Used approach A | Due to constraint B |

## Learnings & Gotchas

- [Insight that will save the next agent time]
- [Non-obvious thing discovered during work]
- [Gotcha or trap to avoid]

## Blockers & Issues

- [Any problems encountered]
- [Things that didn't work and why]

## Open Questions

- [Thing that still needs clarification]
- [Decision that needs user input]

## Environment & Commands

```bash
# Commands to restore working state
[Any dev server, build, or test commands needed]
```

## Next Steps

1. **Immediate:** [Exactly what to do first]
2. **Then:** [What comes after]
3. **Finally:** [End goal]

---

*To continue this work, read this document and proceed with the next steps.*
```

## Writing Guidelines

1. **Be concise** - The next agent has limited context too
2. **Be specific** - Use exact file paths, not vague references
3. **Be actionable** - The next agent should know exactly what to do
4. **Preserve decisions** - Don't make the next agent re-discover rationale
5. **Capture learnings** - Non-obvious insights are the most valuable part

## After Writing

**Always conclude by printing the resume command:**

```
Handoff complete. To resume in a new session:

/resume ./docs/handoffs/[filename]
```

This gives the user a copy-paste command to continue the work.
