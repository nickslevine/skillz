# Claude Code Plugins and Skills Research

This document contains research on how to create Claude Code plugins with agent skills.

## Plugin Structure

A Claude Code plugin should be structured like this:

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Required manifest
├── skills/                   # Agent skills
│   └── my-skill/
│       └── SKILL.md
├── commands/                 # Optional slash commands
│   └── my-command.md
├── hooks/                    # Optional event handlers
│   └── hooks.json
├── .mcp.json                 # Optional MCP servers
├── .lsp.json                 # Optional LSP servers
└── README.md
```

**Important:** Only `plugin.json` goes inside `.claude-plugin/`. All other directories (skills, commands, hooks) must be at the plugin root level.

## plugin.json (Required Manifest)

### Required Fields

```json
{
  "name": "my-plugin",
  "description": "Description of what your plugin does",
  "version": "1.0.0"
}
```

### All Available Fields

| Field | Purpose | Required |
|-------|---------|----------|
| `name` | Unique identifier and slash command namespace. Commands are prefixed with this (e.g., `/my-plugin:hello`) | Yes |
| `description` | Shown in plugin manager when browsing or installing | Yes |
| `version` | Track releases using semantic versioning | Yes |
| `author` | Attribution information (object with `name` field) | No |
| `homepage` | Link to plugin documentation/website | No |
| `repository` | Git repository URL | No |
| `license` | License type | No |

### Full Example

```json
{
  "name": "my-plugin",
  "description": "A plugin that does useful things",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  },
  "homepage": "https://github.com/username/my-plugin",
  "repository": "https://github.com/username/my-plugin",
  "license": "MIT"
}
```

## SKILL.md Format

Skills are the core of teaching Claude specialized tasks. Each skill needs a folder under `skills/` containing a `SKILL.md` file.

### Basic Structure

```markdown
---
name: my-skill-name
description: Clear description of what this skill does and when to use it. Include keywords users would naturally say.
---

# My Skill Name

Instructions for Claude to follow when this skill is active.

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Lowercase letters, numbers, hyphens only (max 64 chars) |
| `description` | Yes | When to use it (max 1024 chars). Claude reads this to decide activation |
| `allowed-tools` | No | Comma-separated tools Claude can use without permission |
| `model` | No | Specific Claude model to use when this skill is active |
| `context` | No | Set to `fork` to run in isolated sub-agent context |
| `agent` | No | Agent type when `context: fork` (e.g., `Explore`, `Plan`, `general-purpose`) |
| `hooks` | No | Define lifecycle hooks (`PreToolUse`, `PostToolUse`, `Stop`) |
| `user-invocable` | No | Controls slash command visibility (default: `true`) |

### String Substitutions

Available variables for dynamic content:

| Variable | Description |
|----------|-------------|
| `$ARGUMENTS` | Arguments passed when invoking the skill |
| `${CLAUDE_SESSION_ID}` | Current session ID for logging/correlation |

### Advanced Example with All Options

```markdown
---
name: code-review
description: Reviews code for best practices, security issues, and potential bugs. Use when reviewing code, checking PRs, or analyzing code quality.
allowed-tools:
  - Read
  - Grep
  - Glob
context: fork
agent: Explore
user-invocable: true
---

# Code Review

When reviewing code, check for:

1. **Code organization and structure**
2. **Error handling**
3. **Security concerns**
4. **Test coverage**

## Process

1. First, understand the context of the changes
2. Read through all modified files
3. Identify potential issues
4. Provide constructive feedback

## Examples

See [examples.md](examples.md) for detailed examples.
```

## Multi-File Skills

Skills can include additional files for complex functionality:

```
my-skill/
├── SKILL.md              # Required - main instructions
├── reference.md          # Detailed documentation (loaded when needed)
├── examples.md           # Usage examples
└── scripts/
    └── helper.py         # Utility scripts (executed, not loaded)
```

Reference additional files from SKILL.md:

```markdown
For complete API details, see [reference.md](reference.md)
For usage examples, see [examples.md](examples.md)
```

## Slash Commands

Commands are Markdown files in the `commands/` directory. The filename becomes the command name.

### Basic Command

**File:** `commands/hello.md`

```markdown
---
description: Greet the user with a friendly message
---

# Hello Command

Greet the user warmly and ask how you can help them today.
```

**Usage:** `/my-plugin:hello`

### Command with Arguments

```markdown
---
description: Greet the user with a personalized message
---

# Hello Command

Greet the user named "$ARGUMENTS" warmly and ask how you can help them today.
```

**Usage:** `/my-plugin:hello Alex`

## Hooks Configuration

Hooks enable event handling and automation.

**File:** `hooks/hooks.json`

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{
          "type": "command",
          "command": "npm run lint:fix $FILE"
        }]
      }
    ]
  }
}
```

## Testing Plugins Locally

Load your plugin during development without installation:

```bash
# Single plugin
claude --plugin-dir ./my-plugin

# Multiple plugins
claude --plugin-dir ./plugin-one --plugin-dir ./plugin-two
```

**Note:** Restart Claude Code after making changes to pick up updates.

### Testing Checklist

- Try commands with `/plugin-name:command-name`
- Check that skills are discovered (they activate based on context)
- Verify hooks work as expected
- Use `claude --debug` to see skill loading errors

## Installing Plugins

### From GitHub

1. Add the repository as a marketplace:
   ```
   /plugin marketplace add username/repo-name
   ```

2. Install the plugin:
   ```
   /plugin install plugin-name@username-repo-name
   ```

### From Official Directory

```
/plugin install plugin-name@claude-plugin-directory
```

### Direct Installation

```
/plugin install username/repo-name
```

## Publishing Plugins

1. Create a GitHub repository with the proper structure
2. Ensure `.claude-plugin/plugin.json` is valid
3. Add components (commands, skills) at the plugin root (not inside `.claude-plugin`)
4. Include a README with usage instructions
5. Push to GitHub

Plugin discovery services like ClaudePluginHub will find your plugin automatically.

## Best Practices

### Writing Skill Descriptions

Since Claude reads descriptions to find relevant skills, write descriptions that:

- Include keywords users would naturally say
- Clearly state when the skill should be used
- Be specific with trigger terms

**Good:**
```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**Bad:**
```yaml
description: Handles documents.
```

### Progressive Disclosure

- Keep `SKILL.md` under 500 lines
- Move detailed docs to separate files linked from `SKILL.md`
- Claude loads referenced files only when needed

### When to Use What

| Use | When |
|-----|------|
| **Skills** | Claude chooses when relevant (model-invoked) |
| **Slash Commands** | User types `/command` to run explicitly |
| **CLAUDE.md** | Project-wide instructions always loaded |
| **Hooks** | Run scripts on specific events |
| **MCP Servers** | Connect to external tools/APIs |

## Skill Scope by Location

| Location | Path | Applies To |
|----------|------|-----------|
| Enterprise | Managed settings | All org users |
| Personal | `~/.claude/skills/` | You, all projects |
| Project | `.claude/skills/` | Team, this repo |
| Plugin | Inside plugin directory | Plugin users |

**Priority:** Managed > Personal > Project > Plugin

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Skill not triggering | Improve description with specific keywords users would say |
| Skill doesn't load | Check file path matches `SKILL.md` (case-sensitive) and YAML syntax |
| Multiple skills conflict | Differentiate descriptions with specific trigger terms |
| Plugin skills missing | Clear cache: `rm -rf ~/.claude/plugins/cache` then reinstall |
| Invalid YAML | Ensure frontmatter starts with `---` on line 1, uses spaces (not tabs) |

## Sources

- [Agent Skills - Claude Code Docs](https://code.claude.com/docs/en/skills)
- [Create Plugins - Claude Code Docs](https://code.claude.com/docs/en/plugins)
- [GitHub - anthropics/skills](https://github.com/anthropics/skills)
- [Claude Code Plugins README](https://github.com/anthropics/claude-code/blob/main/plugins/README.md)
- [Skills Specification](https://github.com/anthropics/skills/tree/main/spec)
