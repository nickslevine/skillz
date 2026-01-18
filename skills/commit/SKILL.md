---
name: commit
description: Commit and push all changes from the current unit of work. Handles gitignore, staging, commit message, and push. Use when work is ready to be committed.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Task
  - AskUserQuestion
---

# Commit

Commit and push all changes from the current unit of work with a well-crafted commit message.

## Process

### Step 0: Update Code Maps (if `--map` flag present)

If `$ARGUMENTS` contains `--map`:

1. Invoke the `/map` skill in default mode (no arguments)
2. This updates MAP.md files for all folders containing changed files
3. The updated MAP.md files will be included in this commit
4. Continue to Step 1 after mapping completes

### Step 1: Check for Files to Gitignore

Run `git status` to see all changed and untracked files.

**Auto-add to .gitignore** if any of these are present:

| Pattern | Reason |
|---------|--------|
| `.env`, `.env.*` | Environment secrets |
| `.DS_Store` | macOS metadata |
| `node_modules/` | Dependencies |
| `__pycache__/`, `*.pyc` | Python bytecode |
| `.venv/`, `venv/` | Python virtual environments |
| `dist/`, `build/`, `out/` | Build artifacts |
| `*.log` | Log files |
| `.cache/` | Cache directories |
| `coverage/`, `.coverage` | Test coverage |
| `*.sqlite`, `*.db` | Local databases |
| `.idea/`, `.vscode/` | IDE settings (unless intentionally tracked) |
| `tmp/`, `temp/` | Temporary files |

**Also scan for secrets** - flag and gitignore any files that appear to contain:
- API keys, tokens, passwords
- Private keys (`.pem`, `.key`)
- Credentials files (`credentials.json`, `secrets.yaml`)
- Files with `secret`, `password`, `token`, `apikey` in the name

If `.gitignore` doesn't exist, create it. If it exists, append new patterns (don't duplicate existing ones).

### Step 2: Check for Large Files

Scan for potentially large files that shouldn't be in git:

**File types to check:**
- Data files: `.jsonl`, `.parquet`, `.csv`, `.tsv`, `.json` (when large)
- ML artifacts: `.pt`, `.pth`, `.ckpt`, `.safetensors`, `.bin`, `.h5`, `.pkl`, `.pickle`
- Archives: `.zip`, `.tar`, `.gz`, `.7z`
- Media: `.mp4`, `.mov`, `.avi`, `.mp3`, `.wav`
- Other: any file that looks like training data or model weights

**Size thresholds:**
| Size | Action |
|------|--------|
| > 10MB | Strongly recommend gitignore, ask user to confirm |
| 1MB - 10MB | Ask user if it should be gitignored |
| < 1MB | Include without asking |

**To check file sizes:**
```bash
find . -type f \( -name "*.jsonl" -o -name "*.parquet" -o -name "*.csv" -o -name "*.pt" -o -name "*.ckpt" -o -name "*.safetensors" -o -name "*.pkl" \) -size +1M -exec ls -lh {} \;
```

**When asking the user**, use AskUserQuestion with:
- The filename and size
- Options: "Gitignore this file", "Gitignore all files of this type", "Include in commit"
- For files >10MB, make "Gitignore" the recommended option

### Step 3: Stage Relevant Files

Stage all files relevant to the current unit of work:

```bash
git add -A
```

If you added files to `.gitignore` in Steps 1-2, those will automatically be excluded.

**Do NOT stage:**
- Files unrelated to the current work (use judgement based on session context)
- If unsure whether a file is related, include it

### Step 4: Review Changes

Run `git diff --cached --stat` to see what will be committed.

Run `git log --oneline -5` to see recent commit message style for consistency.

### Step 5: Write Commit Message

Use **Conventional Commits** format:

```
<type>(<scope>): <description>

[optional body]

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Types:**
| Type | Use When |
|------|----------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no code change |
| `refactor` | Code change that neither fixes nor adds |
| `test` | Adding or updating tests |
| `chore` | Maintenance, dependencies, config |

**Scope:** Optional, indicates area (e.g., `auth`, `api`, `ui`)

**Description guidelines:**
- Use imperative mood ("add" not "added")
- Lowercase, no period at end
- Be specific but concise
- Focus on the "what" and "why"

**Examples:**
```
feat(auth): add OAuth2 login flow

fix: prevent crash when user has no email

chore: update dependencies to fix security vulnerabilities

refactor(api): extract validation logic into middleware
```

### Step 6: Commit

```bash
git commit -m "<message>"
```

Use a HEREDOC for multi-line messages:

```bash
git commit -m "$(cat <<'EOF'
feat(skills): add commit skill for automated git workflow

Handles gitignore detection, staging, conventional commit messages,
and push in a single command.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### Step 7: Push

```bash
git push
```

If the branch has no upstream, set it:

```bash
git push -u origin HEAD
```

If push fails due to remote changes, inform the user rather than force pushing.

## Arguments

Optional: `$ARGUMENTS` can provide guidance on the commit scope or message focus.

**Flags:**
| Flag | Effect |
|------|--------|
| `--map` | Run `/map` first to update MAP.md files for changed code |

**Examples:**
- `/commit` - Auto-detect everything
- `/commit --map` - Update code maps, then commit everything
- `/commit auth changes only` - Focus on auth-related files
- `/commit --map refactored auth` - Update maps and commit with context
- `/commit WIP save point` - Use a WIP-style message

## Output

After completing, report:
1. Files that were added to `.gitignore` (if any)
2. Files that were committed
3. The commit message used
4. Push status (success or any issues)

## Safety

- **Never** force push
- **Never** commit files that look like secrets (gitignore them instead)
- **Never** amend existing commits
- If something looks wrong, stop and inform the user
