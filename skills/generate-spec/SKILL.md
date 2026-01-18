---
name: generate-spec
description: Interview the user in-depth about a feature to create a complete specification and implementation plans. Use when the user wants to design a feature, write a spec, or plan an implementation thoroughly.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Bash
  - AskUserQuestion
---

# Generate Spec

Conduct an in-depth interview to create a complete specification document and break it into actionable implementation plans.

## Output Structure

```
docs/
├── specs/
│   └── <feature-slug>.md          # The specification
└── plans/
    ├── <feature-slug>-<plan1>.md  # Implementation plan 1
    ├── <feature-slug>-<plan2>.md  # Implementation plan 2
    └── ...
```

## Arguments

`$ARGUMENTS` can be one of:

1. **File path** (e.g., `./docs/specs/auth-flow.md`) - Read and update this existing spec
2. **Description string** (e.g., `"user authentication with OAuth"`) - Create a new spec

## Process

### Step 1: Determine Input Type

Examine `$ARGUMENTS`:

- **If it's a file path** (contains `/` or ends with `.md`):
  - Read the file at that path
  - This is the spec to flesh out
  - Derive the feature slug from the filename

- **If it's a description string**:
  - This is a new spec to create
  - Derive a filename slug from the description:
    - Lowercase, hyphenated, max 50 chars
    - Example: `"User authentication with OAuth"` → `user-auth-oauth`
  - Ensure `docs/specs/` directory exists

### Step 2: Conduct In-Depth Interview

Interview the user thoroughly using the AskUserQuestion tool. Your goal is to uncover every detail, edge case, and consideration needed for both the spec AND the implementation plans.

**Interview Guidelines:**

- Ask about things that are NOT obvious from any existing document
- Go deep, not broad - follow up on answers with more specific questions
- Challenge assumptions and explore alternatives
- Cover ALL of these dimensions:

#### Technical Implementation
- Architecture decisions and tradeoffs
- Data models and relationships
- API design and contracts
- Performance considerations and bottlenecks
- Scalability concerns
- Error handling and failure modes
- Security implications
- Testing strategy

#### UI & UX
- User flows and journeys
- Edge cases in the interface
- Loading states, empty states, error states
- Accessibility requirements
- Mobile/responsive considerations
- Interaction patterns and feedback

#### Business & Product
- Success metrics and KPIs
- User personas and use cases
- Priority and scope boundaries
- Dependencies on other systems/teams
- Rollout strategy and feature flags
- Backwards compatibility

#### Concerns & Risks
- What could go wrong?
- What are the unknowns?
- What assumptions are being made?
- What's the rollback plan?

#### Implementation Breakdown
- What are the major components/subsystems?
- What can be built in parallel vs sequentially?
- What are the natural boundaries between work streams?
- Are there any parts that should be done first?

**Question Quality Rules:**

1. NEVER ask questions already answered in the document
2. NEVER ask generic questions - be specific to THIS feature
3. Ask questions that reveal hidden complexity
4. Ask "what if" and "have you considered" questions
5. Ask about the boundaries and limits
6. Ask about things the user hasn't thought about yet

**Interview Flow:**

- Use AskUserQuestion with 2-4 focused questions at a time
- Group related questions together
- After each answer, go deeper on interesting points
- Continue until you've covered all dimensions thoroughly
- Let the user know when you're moving to a new topic area

### Step 3: Write the Specification

Write the complete specification to `docs/specs/<feature-slug>.md`:

```markdown
# [Feature Name]

## Overview
Brief description of what this feature does and why.

## Goals
- Primary objectives
- Success metrics

## Non-Goals
- What this feature explicitly does NOT do

## User Stories
- As a [user], I want [action] so that [benefit]

## Technical Design

### Architecture
[Diagrams, component relationships]

### Data Model
[Schemas, relationships]

### API Design
[Endpoints, contracts]

## Edge Cases & Error Handling
[Detailed coverage of edge cases]

## Security Considerations
[Auth, permissions, data protection]

## Testing Strategy
[Unit, integration, e2e approach]

## Rollout Plan
[Phases, feature flags, rollback]

## Implementation Plans
- [plan-name-1](../plans/<feature-slug>-<plan1>.md)
- [plan-name-2](../plans/<feature-slug>-<plan2>.md)

## Open Questions
[Unresolved items for future discussion]
```

### Step 4: Create Implementation Plans

Break the implementation into independent plans. Each plan should be:

- **Self-contained**: An agent can pick it up and execute without waiting
- **Parallelizable**: Can be worked on simultaneously with other plans
- **Appropriately sized**: Completable in a focused work session

**When to merge vs separate:**

| Situation | Action |
|-----------|--------|
| Tightly coupled + must be sequential | Merge into one plan |
| Largely independent + logical ordering | Separate plans with soft dependency |
| Completely independent | Separate plans, no dependency |

**Plan format** (`docs/plans/<feature-slug>-<plan-slug>.md`):

```markdown
# Plan: [Plan Name]

**Spec:** [feature-name](../specs/<feature-slug>.md)
**Status:** pending

<!-- Optional: only include if there's a dependency -->
**Depends on:** <other-plan> (brief reason)
<!-- Or if hard blocked: -->
**Blocked by:** <other-plan> (cannot start until X exists)

## Overview
Brief context - what this plan accomplishes and why it's a coherent unit.

## Tasks
- [ ] First task with enough detail to execute
- [ ] Second task
- [ ] Third task
  - [ ] Subtask if needed
- [ ] Write tests for X
- [ ] Update documentation

## Files to Create/Modify
- `src/path/to/file.ts` - description
- `src/path/to/other.ts` - description

## Notes
Implementation notes, gotchas, decisions, or context for the agent.
```

**Dependency guidance:**

| Phrasing | Meaning |
|----------|---------|
| `Depends on: X` | Should do X first for cleaner implementation, but can start without |
| `Blocked by: X` | Cannot start until X is complete (hard dependency) |
| *(no dependency)* | Fully independent, start anytime |

### Step 5: Report Output

After completing, report:
1. Spec file path
2. List of plan files created
3. Dependency graph (if any dependencies exist)
4. Any open questions that surfaced

## Example Interview Questions

Instead of: "What technology will you use?"
Ask: "You mentioned using PostgreSQL - have you considered how you'll handle the eventual consistency between the search index and the primary database when updates happen?"

Instead of: "What's the user flow?"
Ask: "When a user is mid-way through the flow and loses connectivity, should we persist their partial progress? If so, for how long, and how do we handle conflicts if they start fresh on another device?"

Instead of: "How will you handle errors?"
Ask: "If the third-party payment API returns a timeout but actually processed the charge, how will you detect and reconcile this state?"

Instead of: "How should we break this up?"
Ask: "The OAuth integration and the session management seem related but distinct - could someone build the session layer using mock auth while another person builds the OAuth flow? Or are they too intertwined?"

## Examples

**New spec from description:**
```
/generate-spec real-time collaborative editing
```
→ Interviews user
→ Creates `docs/specs/real-time-collaborative-editing.md`
→ Creates plans like:
  - `docs/plans/real-time-collaborative-editing-crdt-engine.md`
  - `docs/plans/real-time-collaborative-editing-websocket-sync.md`
  - `docs/plans/real-time-collaborative-editing-presence-ui.md`

**Update existing spec:**
```
/generate-spec ./docs/specs/notifications.md
```
→ Reads existing spec
→ Interviews user for gaps
→ Updates spec
→ Creates/updates implementation plans

## Begin

Start by determining the input type from `$ARGUMENTS`, then proceed with the interview.
