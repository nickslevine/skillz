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

## Command-Skill Architecture: Skills Are Auto-Discovered

### Problem
When running `/skillz:handoff`, the agent said "The skill file doesn't exist in this repo" and searched aimlessly instead of creating a handoff document.

### Initial Wrong Assumption
We initially thought commands needed to embed full skill content because "the agent can't find skill files." This led us to duplicate all skill content into command files—which is wrong.

### Correct Understanding (from Claude Code docs)

**Skills and Commands are separate mechanisms:**

| Feature | Skills | Commands (Slash) |
|---------|--------|------------------|
| **Trigger** | Claude auto-discovers based on description | User explicitly types `/command` |
| **Location** | `skills/*/SKILL.md` | `commands/*.md` |
| **Discovery** | Auto-loaded when context matches description | Listed in slash command menu |
| **Purpose** | Knowledge Claude should know about | Reusable prompts/workflows |

**How skill discovery works:**
1. At startup, Claude loads only skill **name and description** (fast startup)
2. When a request matches a skill's description, Claude asks permission to use it
3. Only then does the full `SKILL.md` content load

**Commands should be thin dispatchers:**
```markdown
---
description: Create a handoff document capturing context for the next session
argument-hint: "[short-slug] or [focus guidance]"
---

Invoke the skillz:handoff skill and follow it exactly as presented to you
```

The skill in `skills/handoff/SKILL.md` gets auto-discovered and loaded when the command references it.

### Root Cause of Original Problem
The skill wasn't being discovered, likely due to:
1. Plugin not properly installed (local vs remote installation issue)
2. Skill description not matching context well enough
3. Some other discovery issue

The fix is NOT to embed content, but to:
1. Ensure plugin is properly installed from remote
2. Write good skill descriptions with trigger keywords
3. Keep commands thin and skills comprehensive

### Key Insight
**Don't duplicate content between commands and skills.** Commands dispatch to skills. Skills contain all the logic. The skill's `description` field is critical for auto-discovery—include what the skill does AND trigger keywords for when Claude should use it.

### Description Best Practice
```yaml
description: Create a handoff document when running out of context, ending a session,
or saving work for later. Use when context window is filling up, handing off to another
agent, or user says "handoff", "save context", or "wrap up".
```

Include:
1. What it does (capabilities)
2. When to use it (trigger conditions)
3. Trigger keywords users might say
