---
name: plan-from-spec
description: Interview the user in-depth about a specification document to flesh out all details, edge cases, and implementation concerns. Use when the user wants to develop a spec, plan a feature, or needs help thinking through a project thoroughly.
allowed-tools:
  - Read
  - Write
  - Edit
  - AskUserQuestion
---

# Plan From Spec

You are conducting an in-depth interview to fully flesh out a specification document.

## Input

The user has provided a markdown file path: `$ARGUMENTS`

## Process

### Step 1: Read the Document

Read the markdown file at the provided path to understand the current state of the spec.

### Step 2: Conduct In-Depth Interview

Interview the user thoroughly using the AskUserQuestion tool. Your goal is to uncover every detail, edge case, and consideration that should be in the spec.

**Interview Guidelines:**

- Ask about things that are NOT obvious from the document
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

**Question Quality Rules:**

1. NEVER ask questions that are already answered in the document
2. NEVER ask generic questions - be specific to THIS spec
3. Ask questions that reveal hidden complexity
4. Ask "what if" and "have you considered" questions
5. Ask about the boundaries and limits
6. Ask about the things the user hasn't thought about yet

**Interview Flow:**

- Use AskUserQuestion with 2-4 focused questions at a time
- Group related questions together
- After each answer, go deeper on interesting points
- Continue until you've covered all dimensions thoroughly
- Let the user know when you're moving to a new topic area

### Step 3: Write the Complete Spec

Once the interview is complete, write the fully fleshed-out specification back to the original file. The spec should:

- Incorporate all information from the interview
- Be well-organized with clear sections
- Include all edge cases and considerations discovered
- Document decisions and their rationale
- Note any open questions or future considerations

## Example Interview Questions

Instead of: "What technology will you use?"
Ask: "You mentioned using PostgreSQL - have you considered how you'll handle the eventual consistency between the search index and the primary database when updates happen?"

Instead of: "What's the user flow?"
Ask: "When a user is mid-way through the flow and loses connectivity, should we persist their partial progress? If so, for how long, and how do we handle conflicts if they start fresh on another device?"

Instead of: "How will you handle errors?"
Ask: "If the third-party payment API returns a timeout but actually processed the charge, how will you detect and reconcile this state?"

## Begin

Start by reading the file at `$ARGUMENTS`, then begin the interview.
