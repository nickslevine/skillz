---
name: resume
description: Resume work from a handoff document. Reads the handoff file and continues where the previous agent left off. Use when starting a new session to continue previous work.
allowed-tools:
  - Read
  - Glob
  - Bash
  - TodoWrite
---

# Resume

Resume work from a handoff document created by a previous agent session.

## Arguments

`$ARGUMENTS` should be the path to the handoff document:

```
/resume ./docs/handoffs/HANDOFF-2026-01-18-auth-flow.md
```

If no path is provided, look for the most recent handoff in `./docs/handoffs/`.

## Process

### Step 1: Read the Handoff Document

Read the handoff file at the provided path (or find the most recent one).

### Step 2: Internalize Context

Parse and understand:
- **Task Overview** - What is the overall goal?
- **Source Documents** - Read any linked specs/plans
- **Current Status** - What's done, in progress, remaining?
- **Key Files** - Note the important files to work with
- **Decisions Made** - Respect these, don't re-litigate
- **Learnings & Gotchas** - Avoid known pitfalls
- **Blockers & Issues** - Be aware of problems
- **Open Questions** - May need to ask user
- **Next Steps** - What to do first

### Step 3: Set Up Todo List

Create a todo list from the handoff's status sections:
- Mark completed items as completed
- Add in-progress and remaining items as pending
- Add the immediate next step as the first pending item

### Step 4: Restore Environment (if needed)

If the handoff includes environment commands, offer to run them:
- Dev servers
- Build commands
- Test setup

### Step 5: Confirm and Begin

Briefly summarize to the user:
1. What task you're resuming
2. What's already done
3. What you'll do next

Then begin working on the next step.

## Finding Recent Handoffs

If no path provided, find the most recent:

```bash
ls -t ./docs/handoffs/HANDOFF-*.md | head -1
```

## Example Output

```
Resuming from: ./docs/handoffs/HANDOFF-2026-01-18-auth-flow.md

Task: Implement OAuth2 authentication flow

Completed:
- Set up OAuth provider configuration
- Created login endpoint

Next up: Implement callback handler and token storage

Proceeding with the callback handler...
```
