---
name: self-improve
description: Review accumulated learnings and take actions to improve the agent system. Creates new skills, scripts, or documentation based on patterns discovered during work.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

# Self-Improve

Review the learnings captured in `docs/LEARNINGS.md` and take actions to make the agent system smarter and more efficient over time.

## When to Use

- Periodically (e.g., end of week, end of sprint)
- After completing a major feature
- When the learnings file has grown significantly
- When user requests system improvement

## Arguments

- `/self-improve` - Review all unprocessed learnings
- `/self-improve --dry-run` - Show what would be done without making changes
- `/self-improve skills` - Focus only on skill opportunities
- `/self-improve scripts` - Focus only on script opportunities
- `/self-improve docs` - Focus only on documentation updates

## Process

### Step 1: Read Learnings

Read `docs/LEARNINGS.md` and identify entries marked as **Actionable: yes** that haven't been processed yet.

Group actionable items by type:
- `skill-opportunity` - Repeated processes that could become skills
- `script-opportunity` - Manual work that could be automated
- `docs-update` - Documentation that should be added/updated
- `pattern` - Patterns to document in CLAUDE.md
- `gotcha` - Gotchas to document in CLAUDE.md

### Step 2: Prioritize Actions

For each actionable learning, assess:

| Factor | Question |
|--------|----------|
| **Frequency** | How often does this come up? |
| **Impact** | How much time/effort does it save? |
| **Complexity** | How hard is it to implement? |
| **Risk** | Could this cause problems if wrong? |

Prioritize: High frequency + High impact + Low complexity = Do first

### Step 3: Propose Actions

Present a summary to the user:

```markdown
## Self-Improvement Opportunities

### Skills to Create
1. **[skill-name]** - [description]
   - Based on: [learning reference]
   - Estimated effort: [low/medium/high]

### Scripts to Create
1. **[script-name]** - [description]
   - Based on: [learning reference]
   - Would automate: [what manual work]

### Documentation Updates
1. **Update CLAUDE.md** - [what to add]
   - Section: [where it goes]
   - Based on: [learning reference]

### Patterns to Document
1. **[pattern-name]** - [description]
   - Where: [CLAUDE.md or specific MAP.md]

Proceed with these improvements? [Yes / Select specific ones / Skip]
```

### Step 4: Execute Approved Actions

For each approved action:

#### Creating a New Skill

1. Create `skills/[skill-name]/SKILL.md`
2. Follow the standard skill format:
   ```markdown
   ---
   name: [skill-name]
   description: [when to use this skill]
   allowed-tools:
     - [tools needed]
   ---

   # [Skill Name]

   [Instructions for the skill]

   ## Process
   [Step-by-step process]

   ## Examples
   [Usage examples]
   ```
3. Update README.md skills table
4. Mark learning as processed

#### Creating a Script

1. Create script in `scripts/` directory (or project-appropriate location)
2. Make it executable if bash/shell
3. Document usage in CLAUDE.md:
   ```markdown
   ## Project Scripts

   ### [script-name]
   [Description and usage]
   ```bash
   ./scripts/[script-name] [args]
   ```
   ```
4. Mark learning as processed

#### Updating Documentation

1. Read the target file (CLAUDE.md or specific doc)
2. Add the new content in the appropriate section
3. If section doesn't exist, create it
4. Mark learning as processed

### Step 5: Mark Learnings as Processed

After acting on a learning, update its entry in `docs/LEARNINGS.md`:

```markdown
## [2026-01-18] Skill Opportunity: Boilerplate generation

...existing content...

**Actionable:** yes → **processed**
**Processed:** 2026-01-20
**Action taken:** Created `/boilerplate` skill in skills/boilerplate/SKILL.md

---
```

### Step 6: Report Summary

After completing, report:

```markdown
## Self-Improvement Summary

**Learnings reviewed:** X
**Actions taken:** Y

### Created
- Skill: `/new-skill` - [description]
- Script: `scripts/helper.sh` - [description]

### Documentation Updated
- CLAUDE.md: Added API conventions section
- MAP.md (src/services/): Updated with new patterns

### Skipped (need more info)
- [learning that needs clarification]

### Remaining Actionable
- X learnings still pending action
```

## Skill Creation Guidelines

When creating new skills:

1. **Keep it focused** - One skill, one purpose
2. **Clear trigger** - Description should make it obvious when to use
3. **Minimal tools** - Only request tools actually needed
4. **Step-by-step** - Clear process the agent can follow
5. **Examples** - Show how to invoke and what to expect

## Script Creation Guidelines

When creating scripts:

1. **Idempotent** - Safe to run multiple times
2. **Documented** - Usage info in comments and CLAUDE.md
3. **Error handling** - Fail gracefully with helpful messages
4. **No side effects** - Don't modify things unexpectedly

## Documentation Guidelines

When updating docs:

1. **Right location** - CLAUDE.md for project-wide, MAP.md for local
2. **Concise** - Agents have limited context
3. **Actionable** - Tell agents what to do, not just what exists
4. **Examples** - Show, don't just tell

## LEARNINGS.md Format Reference

Expected format for entries:

```markdown
## [YYYY-MM-DD] Category: Brief Title

**Context:** [What you were working on]
**Plan:** [Link to plan if applicable]

**Learning:**
[Description of the insight, gotcha, or pattern discovered]

**Actionable:** [yes/no/processed]
**Action type:** [skill-opportunity | script-opportunity | docs-update | pattern | gotcha]
**Suggested action:** [If actionable, what should be done]
**Processed:** [Date if processed]
**Action taken:** [What was done if processed]

---
```

## Safety

- Always get user approval before creating skills or scripts
- Don't modify critical project files without confirmation
- Use `--dry-run` to preview changes first
- Keep backups of modified documentation
