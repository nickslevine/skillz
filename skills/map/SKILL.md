---
name: map
description: Create or update hierarchical MAP.md files for codebase navigation. Documents folder structure, file purposes, exports, and types. Use when user says "map", "document codebase", "create map", or before committing to update code maps.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Task
---

# Map

Create hierarchical MAP.md files throughout a codebase for efficient agent exploration. Each folder gets a MAP.md with just enough context for that level - broad overviews at top, detailed exports/types at the code level.

## Arguments

- `/map` - **Default (pre-commit):** Update maps for folders containing changed files (from `git status`)
- `/map <folder>` - Create/update maps for this folder and all children recursively
- `/map <file>` - Update the MAP.md for the file's enclosing folder only

## Execution Strategy

Use subagents for parallel processing of large codebases.

### For `/map <folder>` (recursive):

1. Scan the target folder structure
2. Identify top-level children (immediate subfolders)
3. **Spawn one Task agent per child subtree** (parallel)
   - Each subagent recursively maps its entire subtree
   - Subagent writes MAP.md files for its folder and all descendants
   - Subagent returns a summary (folder name, purpose, key subfolders/files)
4. Collect summaries from all subagents
5. Write the root MAP.md for the target folder using summaries

### For `/map` (default, pre-commit):

1. Run `git status --porcelain` to find changed/added/deleted files
2. Group files by their enclosing folder
3. **Spawn subagents per affected folder** (parallel)
4. Each subagent updates its folder's MAP.md
5. Determine if parent folders need updates (structural changes only):
   - New file added → update parent MAP.md
   - File deleted → update parent MAP.md
   - File content changed → no parent update needed
6. Update parent MAP.md files if needed (cascade up one level only, unless significant structural change warrants root update)

### For `/map <file>`:

1. Identify the file's enclosing folder
2. Read all files in that folder
3. Update or create the folder's MAP.md

## MAP.md Format

Every MAP.md includes a metadata comment on line 1:

```markdown
<!-- mapped: 2026-01-18 a1b2c3d -->
# folder-name/

One-line description of this folder's purpose.

## Subfolders
- `subfolder1/` - Brief description
- `subfolder2/` - Brief description

## Files

### filename.ts
One-line description of this file.

**Exports:** `func1()`, `func2()`, `func3()`
**Types:** `TypeA { field1, field2 }`, `TypeB { field1, field2, field3 }`
```

### Content Guidelines by Level

**Root/Top-level MAP.md:**
- Project purpose (1-2 sentences)
- Tech stack if relevant
- Key entry points ("start here" pointers)
- Subfolder descriptions (one line each)

**Mid-level folder MAP.md:**
- Folder's responsibility (one sentence)
- Subfolder descriptions
- Notable files with brief purposes

**Code-level folder MAP.md (leaf folders with source files):**
- Folder purpose
- Per-file entries with:
  - One-line description
  - **Exports:** Function/method names (no signatures)
  - **Types:** Type/interface names with key fields abbreviated

### Type Field Abbreviation

Include 3-5 most important fields. Use `...` for complex types:

```markdown
**Types:** `User { id, email, role, createdAt }`, `Config { apiUrl, timeout, ... }`
```

## Exclusions

**Always skip these (do not map or include in MAP.md):**

| Pattern | Reason |
|---------|--------|
| `node_modules/` | Dependencies |
| `.git/` | Git internals |
| `dist/`, `build/`, `out/` | Build artifacts |
| `coverage/`, `.nyc_output/` | Test coverage |
| `.cache/`, `.turbo/` | Cache directories |
| `*.test.ts`, `*.spec.ts` | Test files |
| `__tests__/`, `__mocks__/` | Test directories |
| `*.test.js`, `*.spec.js` | Test files |
| `*.d.ts` | Type declaration files (unless primary source) |
| `MAP.md` | The maps themselves |
| Everything in `.gitignore` | Already ignored |

**Include but keep brief:**
- Config files (`tsconfig.json`, `.eslintrc`, `package.json`) - just note their presence
- Lock files (`package-lock.json`, `yarn.lock`) - skip entirely, no value

## Metadata

Every MAP.md starts with a comment containing:
- **Timestamp:** ISO date (YYYY-MM-DD)
- **Git SHA:** Short hash (7 chars) of HEAD when mapped

```markdown
<!-- mapped: 2026-01-18 a1b2c3d -->
```

Get the SHA with:
```bash
git rev-parse --short HEAD
```

## Example Output

### Root: `src/MAP.md`

```markdown
<!-- mapped: 2026-01-18 a1b2c3d -->
# src/

Backend API server for the task management application. Express + TypeScript.

## Subfolders
- `routes/` - API route handlers
- `services/` - Business logic and external integrations
- `models/` - Database models and schemas
- `middleware/` - Express middleware (auth, logging, errors)
- `utils/` - Shared utilities

## Files
- `index.ts` - Application entry point, server setup
- `config.ts` - Environment configuration
```

### Mid-level: `src/services/MAP.md`

```markdown
<!-- mapped: 2026-01-18 a1b2c3d -->
# services/

Business logic layer - orchestrates data access and external APIs.

## Subfolders
- `auth/` - Authentication, sessions, OAuth
- `tasks/` - Task CRUD and business rules
- `notifications/` - Email and push notifications

## Files
- `index.ts` - Re-exports all services
```

### Code-level: `src/services/auth/MAP.md`

```markdown
<!-- mapped: 2026-01-18 a1b2c3d -->
# auth/

Authentication and session management.

## Files

### auth.ts
Core authentication logic - login, logout, token refresh.

**Exports:** `login()`, `logout()`, `refreshToken()`, `validateCredentials()`
**Types:** `Credentials { email, password }`, `AuthResult { user, token, expiresAt }`

### session.ts
Session storage and validation using Redis.

**Exports:** `createSession()`, `getSession()`, `destroySession()`, `extendSession()`
**Types:** `Session { id visibleTo, userId, expiresAt, metadata }`

### oauth.ts
OAuth2 provider integrations (Google, GitHub).

**Exports:** `initiateOAuth()`, `handleCallback()`, `linkAccount()`
**Types:** `OAuthProvider { name, clientId, authUrl, tokenUrl }`, `OAuthToken { accessToken, refreshToken, expiresAt }`
```

## Subagent Prompt Template

When spawning subagents, use this prompt structure:

```
Map the folder at `{path}` and all its children recursively.

For each folder:
1. Read all source files (skip exclusions)
2. Extract: file purpose, exports, types with key fields
3. Write MAP.md with timestamp and git SHA: {sha}

Return a summary: folder purpose (1 sentence), key subfolders, key files.
```

## Error Handling

- **Empty folder:** Create MAP.md noting it's empty or skip
- **Binary files:** Note presence but don't analyze (`assets/logo.png` - just list it)
- **Very large files:** Note presence, don't read fully
- **Permission errors:** Skip and note in output

## Output

After mapping completes, report:
1. Number of MAP.md files created/updated
2. Folders that were mapped
3. Any files or folders that were skipped (and why)
