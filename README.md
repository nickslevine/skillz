# Skillz

A Claude Code plugin for structured feature development with continuous improvement.

## Quick Start

```bash
# 1. Add the plugin (run in Claude Code)
/plugin marketplace add nickslevine/skillz

# 2. Restart Claude Code to load the plugin

# 3. Start using skills
/generate-spec my new feature
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
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  /generate-spec │ ──▶ │   /implement    │ ──▶ │    /commit      │
│                 │     │                 │     │                 │
│  Create spec +  │     │  Execute plan,  │     │  Stage, commit, │
│  impl plans     │     │  run tests      │     │  push           │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                               │
                               ▼ (if context fills)
                        ┌─────────────────┐     ┌─────────────────┐
                        │    /handoff     │ ──▶ │    /resume      │
                        │                 │     │                 │
                        │  Save context   │     │  Continue work  │
                        └─────────────────┘     └─────────────────┘
                               │
                               ▼ (captures learnings)
                        ┌─────────────────┐
                        │  /self-improve  │
                        │                 │
                        │  Create skills, │
                        │  scripts, docs  │
                        └─────────────────┘
```

## Skills

### Planning & Specs

| Skill | Usage | Purpose |
|-------|-------|---------|
| `/generate-spec` | `/generate-spec user auth with OAuth` | Interview → spec + implementation plans |
| `/map` | `/map ./src` | Create MAP.md files for codebase navigation |

### Implementation

| Skill | Usage | Purpose |
|-------|-------|---------|
| `/implement` | `/implement docs/plans/auth-db.md` | Execute plan tasks, run tests, commit |
| `/commit` | `/commit` or `/commit --map` | Stage, commit, push with conventional messages |

### Session Management

| Skill | Usage | Purpose |
|-------|-------|---------|
| `/handoff` | `/handoff` | Save context for next session |
| `/resume` | `/resume docs/handoffs/HANDOFF-...md` | Continue from handoff |

### Continuous Improvement

| Skill | Usage | Purpose |
|-------|-------|---------|
| `/self-improve` | `/self-improve` | Review learnings, create skills/scripts/docs |

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
/generate-spec user authentication with OAuth

# 2. Implement the first plan
/implement docs/plans/user-auth-oauth-db-schema.md --map

# 3. If context fills up
/handoff

# 4. In a new session
/resume docs/handoffs/HANDOFF-2026-01-18-auth.md

# 5. Periodically improve the system
/self-improve
```

## Key Features

- **Structured workflow** - Specs → Plans → Implementation → Commit
- **Context management** - Handoff/resume for long tasks
- **Continuous learning** - Agents capture insights, `/self-improve` acts on them
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
