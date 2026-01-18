# Skillz

A Claude Code plugin with custom agent skills.

## Installation

### From GitHub

```bash
# Add the marketplace
/plugin marketplace add nlevine/skillz

# Install the plugin
/plugin install skillz@nlevine-skillz
```

### Local Development

```bash
claude --plugin-dir /path/to/skillz
```

## Skills

| Skill | Description |
|-------|-------------|
| `generate-spec` | Interview user to create a complete spec (`docs/specs/`) and implementation plans (`docs/plans/`) |
| `implement` | Execute an implementation plan - works through tasks, runs tests/lints, commits at logical points |
| `self-improve` | Review learnings and improve the agent system - creates skills, scripts, or documentation |
| `handoff` | Create a handoff document when running out of context, capturing everything the next agent needs to continue |
| `resume` | Resume work from a handoff document, restoring context and continuing where the previous session left off |
| `commit` | Commit and push all changes - handles gitignore, staging, conventional commit messages, and push |
| `map` | Create hierarchical MAP.md files for codebase exploration - documents folder structure, file purposes, exports, and types |

## Creating New Skills

1. Create a new folder under `skills/`:
   ```
   skills/my-new-skill/SKILL.md
   ```

2. Add frontmatter with `name` and `description`:
   ```markdown
   ---
   name: my-new-skill
   description: What this skill does and when to use it
   ---
   ```

3. Add instructions for Claude below the frontmatter

4. Test locally:
   ```bash
   claude --plugin-dir ./
   ```

## Documentation

See [docs/research/PLUGINS_AND_SKILLS.md](docs/research/PLUGINS_AND_SKILLS.md) for detailed documentation on creating plugins and skills.

## License

MIT
