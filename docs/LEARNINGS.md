# Plugin Development Learnings

Lessons learned while developing and publishing the skillz plugin for Claude Code.

## Remote vs Local Plugin Installation

### Problem
When trying to update the plugin with `/plugin update skillz`, we got:
```
Local plugins cannot be updated remotely. To update, modify the source at: ./
```

### Root Causes

1. **Installing from within the plugin directory**: Running `/plugin install` from within the local plugin source directory (`/Users/.../skillz/`) causes Claude Code to detect the local `.claude-plugin` folder and install from there instead of from GitHub.

2. **Relative source path interpreted as local**: Using `"source": "./"` in marketplace.json is interpreted as a local path, even when the marketplace itself is fetched from GitHub.

### Solution

Use an explicit remote URL format in marketplace.json:

```json
{
  "plugins": [
    {
      "name": "skillz",
      "source": {
        "source": "url",
        "url": "https://github.com/nickslevine/skillz.git"
      }
    }
  ]
}
```

This tells Claude Code definitively that the plugin source is remote.

### Installation Best Practice

Always install from a different directory than the plugin source:

```bash
cd ~
/plugin install skillz@nickslevine-skillz
```

## Command Namespacing

### Problem
Commands were appearing without namespacing (e.g., `/commit` instead of `/skillz:commit`), which could conflict with other plugins.

### Root Cause
The command files included an explicit `name:` field in the YAML frontmatter:

```yaml
---
name: commit
description: Commit and push all changes
---
```

### Solution
Remove the `name:` field. Commands should derive their name from the filename, and Claude Code will namespace them based on the plugin name:

```yaml
---
description: Commit and push all changes
argument-hint: "[--map] [message guidance]"
---
```

This results in properly namespaced commands like `skillz:commit`.

## Clean Reinstall Procedure

When plugin state gets confused, do a full clean reinstall:

```bash
cd ~  # Important: be outside the plugin source directory
/plugin uninstall skillz
/plugin marketplace remove nickslevine-skillz
/plugin marketplace add nickslevine/skillz
/plugin install skillz@nickslevine-skillz
```

## Marketplace.json Schema

Key fields for a properly configured marketplace.json:

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "marketplace-name",
  "version": "0.1.0",
  "description": "Marketplace description",
  "owner": {
    "name": "owner-name"
  },
  "plugins": [
    {
      "name": "plugin-name",
      "description": "Plugin description",
      "version": "0.1.0",
      "author": {
        "name": "author-name"
      },
      "source": {
        "source": "url",
        "url": "https://github.com/owner/repo.git"
      },
      "category": "development"
    }
  ]
}
```

## Command Frontmatter

Supported fields in command YAML frontmatter:

- `description` (required): Short description shown in command list
- `argument-hint`: Shows usage hint (e.g., `"<file> [options]"`)
- `allowed-tools`: Restricts which tools the command can use (e.g., `Bash(git add:*)`)

Do NOT include:
- `name`: Derived from filename, causes namespacing issues if specified

## Enabling Subagent Exploration in Skills

### Problem
Skills were not using subagents to explore the codebase, even when exploration would improve quality (e.g., generate-spec asking better questions after understanding the codebase).

### Root Cause
The `Task` tool was not included in the skill's `allowed-tools` list. Without it, the model cannot spawn subagents for parallel exploration.

### Solution
1. **Add Task to allowed-tools** in the skill's YAML frontmatter:
   ```yaml
   allowed-tools:
     - Read
     - Write
     - Task  # Enables subagent spawning
   ```

2. **Explicitly instruct the skill to use subagents** in the prompt. Just having the tool available isn't enough—the skill instructions should tell the model when and how to spawn agents:
   ```markdown
   **MANDATORY**: Use the Task tool to spawn exploration agents:

   Task(subagent_type="Explore", prompt="Find [specific thing]...")
   ```

3. **Spawn multiple agents in parallel** for efficiency when exploring different aspects:
   - Architecture explorer
   - Related code explorer
   - Convention explorer

### Key Insight
Adding a tool to `allowed-tools` gives the model the *capability*, but explicit instructions in the skill prompt drive *usage*. For reliable subagent exploration, you need both.
