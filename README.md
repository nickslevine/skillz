# Skillz

A Claude Code plugin for structured feature development with continuous improvement.

## Quick Start

```bash
# 1. Add the plugin (run in Claude Code)
/plugin marketplace add nickslevine/skillz

# 2. Restart Claude Code to load the plugin

# 3. Start using skills
/skillz:generate-spec my new feature
```

## Installation

**From GitHub (recommended):**
```bash
/plugin marketplace add nickslevine/skillz
```
Then restart Claude Code.

**Local development:**
```bash
claude --plugin-dir /path/to/skillz
```

## Workflow

```
┌───────────────────────┐     ┌───────────────────┐     ┌─────────────────────┐
│ /skillz:generate-spec │ ──▶ │ /skillz:implement │ ──▶ │   /skillz:commit    │
│                       │     │                   │     │                     │
│  Create spec +        │     │  Execute plan,    │     │  Stage, commit,     │
│  impl plans           │     │  run tests        │     │  push               │
└───────────────────────┘     └───────────────────┘     └─────────────────────┘
                                       │
                                       ▼ (if context fills)
                              ┌───────────────────┐     ┌─────────────────────┐
                              │  /skillz:handoff  │ ──▶ │   /skillz:resume    │
                              │                   │     │                     │
                              │  Save context     │     │  Continue work      │
                              └───────────────────┘     └─────────────────────┘
                                       │
                                       ▼ (captures learnings)
                              ┌───────────────────────┐
                              │  /skillz:self-improve │
                              │                       │
                              │  Create skills,       │
                              │  scripts, docs        │
                              └───────────────────────┘
```

## Skills

### Planning & Specs

| Skill | Usage | Purpose |
|-------|-------|---------|
| `/skillz:generate-spec` | `/skillz:generate-spec user auth with OAuth` | Interview → spec + implementation plans |
| `/skillz:map` | `/skillz:map ./src` | Create MAP.md files for codebase navigation |

### Implementation

| Skill | Usage | Purpose |
|-------|-------|---------|
| `/skillz:implement` | `/skillz:implement docs/plans/auth-db.md` | Execute plan tasks, run tests, commit |
| `/skillz:commit` | `/skillz:commit` or `/skillz:commit --map` | Stage, commit, push with conventional messages |

### Session Management

| Skill | Usage | Purpose |
|-------|-------|---------|
| `/skillz:handoff` | `/skillz:handoff` | Save context for next session |
| `/skillz:resume` | `/skillz:resume docs/handoffs/HANDOFF-...md` | Continue from handoff |

### Continuous Improvement

| Skill | Usage | Purpose |
|-------|-------|---------|
| `/skillz:self-improve` | `/skillz:self-improve` | Review learnings, create skills/scripts/docs |

## Project Structure

```
your-project/
├── docs/
│   ├── specs/           # Feature specifications
│   ├── plans/           # Implementation plans (todo lists)
│   ├── handoffs/        # Session continuity documents
│   └── LEARNINGS.md     # Accumulated insights
├── MAP.md               # Codebase navigation (per folder)
└── CLAUDE.md            # Project-specific agent instructions
```

## Example Session

```bash
# 1. Design a feature
/skillz:generate-spec user authentication with OAuth

# 2. Implement the first plan
/skillz:implement docs/plans/user-auth-oauth-db-schema.md --map

# 3. If context fills up
/skillz:handoff

# 4. In a new session
/skillz:resume docs/handoffs/HANDOFF-2026-01-18-auth.md

# 5. Periodically improve the system
/skillz:self-improve
```

## Key Features

- **Structured workflow** - Specs → Plans → Implementation → Commit
- **Context management** - Handoff/resume for long tasks
- **Continuous learning** - Agents capture insights, `/skillz:self-improve` acts on them
- **Code maps** - Hierarchical MAP.md for efficient codebase navigation
- **Auto-validation** - Runs tests/lint before commits

## Creating New Skills

```
skills/my-skill/SKILL.md
```

```markdown
---
name: my-skill
description: When to use this skill
allowed-tools:
  - Read
  - Write
---

# My Skill

Instructions for the agent...
```

## License

MIT
