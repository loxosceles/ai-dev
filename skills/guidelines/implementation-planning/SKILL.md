---
name: implementation-planning
description: Directory convention and numbering system for multi-session implementation plans. Follow when creating phased feature plans that span multiple sessions.
type: guideline
---

# Implementation Planning

**This is a strict guideline.** Follow these rules exactly.

Convention for structuring implementation plans that persist across sessions.

## Directory Structure

```
docs/planning/implementation/
  {feature-name}/
    index.md              # Navigation hub with progress tracking
    phase-0-*.md          # Prerequisites/setup
    phase-1-*.md          # First implementation phase
    phase-2-*.md          # Second implementation phase
    ...
```

## Numbering

**Phases** = separate documents, executed in order. Phase 0 is always setup.

**Steps** = units of work within a phase, numbered `{phase}.{step}`:
- Phase 1 steps: 1.1, 1.2, 1.3
- Phase 2 steps: 2.1, 2.2, 2.3

Each step should be completable in 15-60 minutes, independently testable, and leave the codebase committable.

## Index File

`index.md` is the single source of truth for progress:

```markdown
# {Feature Name} - Implementation Guide

**Last Updated**: YYYY-MM-DD
**Current Step**: Phase X, Step X.X - {Step Name}

## Quick Navigation

| Phase | Document |
|-------|----------|
| Phase 0 | [Setup](./phase-0-setup.md) |
| Phase 1 | [API Endpoints](./phase-1-api-endpoints.md) |

## Progress Tracking

### Phase 0: Setup
- [x] Step 0.1: Configure dependencies
- [ ] Step 0.2: Create base types

### Phase 1: API Endpoints
- [ ] Step 1.1: Define interfaces
- [ ] Step 1.2: Create handler
```

## Phase Documents

Keep under 500 lines. If approaching that, the phase is doing too much — split into focused phases.

```markdown
## Phase {N}: {Name}

**Goal**: Single sentence.

### Step {N}.1: {Step Name}

**Goal**: What this accomplishes.

**Tasks**:
1. Specific task
2. Specific task

**Testing**: How to verify.

**Commit**: `git commit -m "Step {N}.1: description"`
```

## When to Use

Create implementation plans for multi-day features with dependent phases that need progress tracking across sessions. Don't use for single-file changes, bug fixes, or one-step tasks.
