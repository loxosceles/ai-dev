---
name: implementation-planning
description: Directory convention, numbering system, and workflow for multi-session implementation plans. Follow when creating phased feature plans that span multiple sessions.
type: guideline
---

# Implementation Planning

**This is a strict guideline.** Follow these rules exactly.

Instructions for creating and managing implementation plans that persist across sessions.

---

## Structure

Implementation plans follow this directory structure:

```
docs/planning/implementation/
  {feature-name}/
    index.md              # Navigation hub with progress tracking
    phase-0-*.md          # Prerequisite/setup phase
    phase-1-*.md          # First implementation phase
    phase-2-*.md          # Second implementation phase
    ...
    post-implementation.md # Validation, handoff, troubleshooting
```

### Phases vs Steps

**Phase** = One document containing related work
- Each phase has its own file: `phase-N-descriptive-name.md`
- Keep phases under 500 lines (LLMs struggle with longer documents)
- Phases are executed in numerical order (0, 1, 2, ...)
- Renaming a phase file requires updating all references in index.md

**Step** = One unit of work within a phase
- Each step is numbered: Step N.1, Step N.2, etc. (where N is the phase number)
- Steps within a phase are executed in order
- Each step should be:
  - Completable in 15-60 minutes
  - Independently testable
  - Committable (leaves codebase in working state)

**Example:**
```
phase-1-api-endpoints.md contains:
  - Step 1.1: Define TypeScript interfaces
  - Step 1.2: Create Lambda handler
  - Step 1.3: Add API Gateway route
  - Step 1.4: Test endpoint

phase-2-database-setup.md contains:
  - Step 2.1: Define DynamoDB table
  - Step 2.2: Create table in CDK
  - Step 2.3: Add IAM permissions
```

### Numbering Rules

**Phase numbering:**
- Sequential: 0, 1, 2, 3, ...
- Phase 0 is always prerequisites/setup
- Phases must be executed in order

**Step numbering:**
- Format: `{phase}.{step}` — Phase 1 steps: 1.1, 1.2, 1.3
- Steps must be executed in order within a phase
- Step numbers indicate execution order within their phase

**General rules:**
- Phase numbers indicate execution order (must be sequential: 0, 1, 2, ...)
- Phase 0 is for prerequisites/setup
- Use `post-implementation.md` for validation and handoff
- Keep phase documents under 500 lines (LLMs struggle with longer files)

---

## Index File Structure

The `index.md` serves as the single navigation and progress tracking hub:

```markdown
# {Feature Name} - Implementation Guide

**Last Updated**: YYYY-MM-DD

---

## 🎯 Quick Navigation

| Phase | Document |
|-------|----------|
| Phase 0 | [Phase Name](./phase-0-*.md) |
| Phase 1 | [Phase Name](./phase-1-*.md) |
| Phase 2 | [Phase Name](./phase-2-*.md) |

---

## Overview

Brief description of what this implementation achieves.

### Prerequisites

Before starting:
- Requirement 1
- Requirement 2
- Requirement 3

### Implementation Principles

- Key principle 1
- Key principle 2
- Non-negotiable constraint

---

## Progress Tracking

**Current Step**: Phase X, Step X.X - {Step Name}

### Phase 0: {Name}
- [ ] Step 0.1: Description
- [ ] Step 0.2: Description

### Phase 1: {Name}
- [ ] Step 1.1: Description
- [ ] Step 1.2: Description

---

## Related Documents

- Links to related planning docs
- Links to architecture patterns
```

---

## Phase Document Structure

Each phase document should follow this structure:

```markdown
## Phase {N}: {Phase Name}

**Goal**: Single sentence describing what this phase achieves.

**Duration**: Estimated time

### Context

Why this phase is needed. Background information.

---

### Step {N}.1: {Step Name}

**Goal**: What this step accomplishes.

**Tasks**:
1. Specific task
2. Specific task

**Implementation**:

Detailed instructions, code examples, commands.

**Testing**:

How to verify this step worked.

**Commit**: `git commit -m "Step {N}.{X}: [brief description]"`

---

### Step {N}.2: {Step Name}

...
```

**Important:**
- Step numbers must match phase number: Phase 1 has steps 1.1, 1.2, 1.3, etc.
- Each step should be small enough to complete in 15-60 minutes
- Each step should be testable independently
- Each step should leave codebase in committable state
- Keep total phase document under 500 lines
  - If approaching 500 lines, the phase is too big and unmanageable
  - Don't arbitrarily split the document — reorganize the work
  - Split into multiple focused phases with clear goals
  - Each phase should have a single, focused goal

---

## Implementation Workflow

### The Golden Rule: ALWAYS FOLLOW THE PLAN

**Never write code without updating the plan first.**

The implementation plan is the single source of truth for what gets built and how. During implementation:

1. **Read the current step** in index.md
2. **Follow the step exactly** as documented in the phase file
3. **If you need to deviate** → STOP and enter planning mode

### Planning Mode

When you discover:
- The current step won't work as written
- A better approach exists
- Something needs refactoring
- A dependency is missing
- An assumption was wrong

**Immediately enter planning mode:**

1. **STOP writing code**
2. **Discuss the issue** with the developer
3. **Update the plan** with the new approach
4. **Check all other steps** — does this change affect them?
5. **Verify against principles** — does this follow core principles and patterns?
6. **Check dependencies** — does this break earlier steps or block later steps?
7. **Update index.md** — reflect any step changes in progress tracking
8. **Get confirmation** before resuming implementation

### Plan Updates

When updating the plan:
- **Be specific**: Document exactly what changes and why
- **Update dependencies**: Note which other steps are affected
- **Preserve history**: Don't delete old approaches, mark them as superseded
- **Update index.md**: Keep progress tracking current

---

## Phase Completion

When a phase is complete:
- [ ] All steps in phase executed successfully
- [ ] All tests passing
- [ ] Changes committed with clear messages
- [ ] Index.md updated to next phase
- [ ] Prerequisites for next phase verified

No separate completion checklist in phase documents — tracking happens only in index.md.

---

## Creating a New Implementation Plan

1. **Create index.md first** — Establish navigation and progress tracking
2. **Break into phases** — Each phase = one document, completable in 1-3 days
3. **Break phases into steps** — Each step completable in 15-60 minutes
4. **Number sequentially** — Phases: 0, 1, 2, ... | Steps: 1.1, 1.2, ... | 2.1, 2.2, ...
5. **Keep phases manageable** — If a phase document approaches 500 lines, it's too big
6. **Verify prerequisites** — Document what must exist before starting

## During Implementation

1. **Check index.md** — Know exactly which step you're on
2. **Read the step** — Understand what needs to be done
3. **Execute the step** — Follow instructions exactly
4. **Test the step** — Verify it worked
5. **Commit the step** — Save progress with clear commit message
6. **Update index.md** — Check off the completed step
7. **Move to next step** — Repeat

**Important:** Steps must be executed in order. Phases must be executed in order.

## Reordering Work

**To reorder steps within a phase:**
1. Update step numbers in phase document
2. Update step references in index.md
3. Verify implementation dependencies
4. Check no steps are blocked by reordering

**To insert a new phase:**
1. Create new phase file with appropriate number
2. Renumber subsequent phase files
3. Update step numbers inside renumbered phase files
4. Update all references in index.md
5. Verify overall implementation order still makes sense

---

## When to Create Implementation Plans

**Create for:**
- Multi-day features requiring coordination
- Features with multiple dependent phases
- Features requiring careful sequencing
- Features that need progress tracking across sessions

**Don't create for:**
- Single-file changes
- Simple bug fixes
- Routine maintenance tasks
- One-step operations

---

## Key Principles

- **Fail fast**: Validate prerequisites before starting
- **Incremental**: Each step should be independently testable
- **Reversible**: Each step ends with git commit for easy reset
- **Explicit**: No assumptions, document everything
- **Trackable**: Clear progress indicators at all times

---

## Progressive Improvement

If the developer corrects a behavior that this skill should have prevented, suggest a specific amendment to this skill to prevent the same correction in the future.
