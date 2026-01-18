# Handoff: Skillz Plugin Initial Setup

**Date:** 2026-01-18
**Session:** Created Claude Code plugin structure with initial skills

## Task Overview

Set up a Claude Code plugin called "skillz" with custom agent skills for workflow automation.

## Source Documents

- `docs/research/PLUGINS_AND_SKILLS.md` - Comprehensive documentation on creating Claude Code plugins and skills

## Current Status

### Completed
- [x] Research Claude Code plugin and skill documentation
- [x] Create plugin structure (`.claude-plugin/plugin.json`)
- [x] Create `plan-from-spec` skill - interviews user to flesh out specs
- [x] Create `handoff` skill - creates context handoff documents
- [x] Create `resume` skill - resumes work from handoff documents
- [x] Create `commit` skill - handles gitignore, staging, commit, push
- [x] Set up `docs/handoffs/` directory for handoff documents
- [x] Write README with installation and usage instructions

### In Progress
- [ ] Nothing in progress

### Remaining
- [ ] Test the plugin locally with `claude --plugin-dir ./`
- [ ] Verify each skill works as expected
- [ ] Commit and push to GitHub
- [ ] Install from GitHub as a marketplace plugin

## Key Files & Folders

| Path | Purpose |
|------|---------|
| `.claude-plugin/plugin.json` | Plugin manifest (name, version, description) |
| `skills/plan-from-spec/SKILL.md` | Skill for in-depth spec interviews |
| `skills/handoff/SKILL.md` | Skill for creating handoff documents |
| `skills/resume/SKILL.md` | Skill for resuming from handoff documents |
| `skills/commit/SKILL.md` | Skill for git commit workflow |
| `docs/research/PLUGINS_AND_SKILLS.md` | Reference documentation |
| `docs/handoffs/` | Where handoff documents are stored |
| `README.md` | Plugin installation and usage guide |

## Decisions Made

| Decision | Rationale |
|----------|-----------|
| Named plugin "skillz" | Matches repo name |
| Used conventional commits for commit skill | Industry standard, clear semantics |
| 1MB/10MB thresholds for large files | Balances repo bloat vs. small data files |
| Handoff prints `/resume` command | Frictionless copy-paste to continue |
| Agent auto-derives filename slugs | Minimize user friction |

## Learnings & Gotchas

- Plugin structure: only `plugin.json` goes in `.claude-plugin/`, all other dirs (skills, commands) at root
- SKILL.md frontmatter must start on line 1 with `---` (no blank lines before)
- `$ARGUMENTS` is the placeholder for skill arguments
- Skills are model-invoked (Claude decides when to use based on description)
- Use `claude --plugin-dir ./` for local testing, restart to pick up changes
- Slash commands are namespaced: `/skillz:commit` not `/commit`

## Blockers & Issues

None encountered.

## Open Questions

- Should additional skills be added? (e.g., PR review, code exploration)
- Any adjustments to existing skills after testing?

## Environment & Commands

```bash
# Test the plugin locally
claude --plugin-dir /Users/nlevine/Dev/skillz

# Or from the skillz directory
cd /Users/nlevine/Dev/skillz
claude --plugin-dir ./
```

## Next Steps

1. **Immediate:** Test plugin locally with `claude --plugin-dir ./`
2. **Then:** Try each skill (`/skillz:commit`, `/skillz:handoff`, etc.)
3. **Then:** Commit and push to GitHub
4. **Finally:** Install from GitHub: `/plugin marketplace add nlevine/skillz`

---

*To continue this work, read this document and proceed with the next steps.*
