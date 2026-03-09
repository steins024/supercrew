````skill
---
name: do
description: "Use when managing feature lifecycle — starting work (todo → doing), completing implementation (doing → ready-to-ship), shipping (ready-to-ship → shipped), or going back to requirements (doing → todo). Handles branch creation and dev-* file management."
---

# Do Task

## Overview

Manage the full lifecycle of a feature. This skill handles status transitions, branch management, and creation of development files (`dev-design.md`, `dev-plan.md`, `dev-log.md`).

## Valid Status Transitions

```
todo → doing               (creates dev-* files, switches to work branch)
doing → ready-to-ship      (implementation complete)
ready-to-ship → shipped    (released)
doing → todo               (back to requirements)
```

**Invalid transitions** (e.g., `todo → shipped`, `shipped → doing`) must be rejected with an explanation of which intermediate steps are needed.

## Process

### Step 1: Identify the Feature

- Use the active feature from the current session context
- If no active feature, ask the user which feature to work on
- Verify the feature directory exists in `.supercrew/tasks/<feature-id>/`

### Step 2: Determine New Status

Infer the appropriate status transition based on context:

| Signal | Transition |
|--------|------------|
| User says "start working", "begin", or `/supercrew:start <id>` | `todo → doing` |
| User says "done", "implementation complete", "ready for review" | `doing → ready-to-ship` |
| User says "ship it", "released", "deployed" | `ready-to-ship → shipped` |
| User says "back to requirements", "needs rework" | `doing → todo` |

If ambiguous, ask the user to confirm.

### Step 3: Execute Transition

#### For `todo → doing`:


1. **(Optional) Create or switch to a feature branch**:
   - If you are currently on the 'main' or 'dev' branch, it is recommended to create a new branch for your feature work:
   - Example: `git checkout -b my-feature-branch`
   - If you are already on a feature branch, you can continue working there.
   - If branch exists, switch to it; otherwise create it

4. **Create dev-design.md**:
   ```markdown
   ---
   status: draft
   reviewers: []
   ---

   # <title> — Technical Design

   ## Design Decisions

   <!-- Key architectural and implementation decisions -->

   ## Architecture

   <!-- High-level architecture, data flow, component interactions -->

   ## Implementation Notes

   <!-- Specific implementation details, edge cases, gotchas -->
   ```

5. **Create dev-plan.md**:
   ```markdown
   ---
   total_tasks: 0
   completed_tasks: 0
   progress: 0
   ---

   # <title> — Implementation Plan

   ## Tasks

   <!-- Task breakdown will be generated from design -->

   - [ ] Task 1: (to be defined)
   ```

6. **Create dev-log.md**:
   ```markdown
   # <title> — Development Log

   ## <YYYY-MM-DD> — Work Started

   - Status changed: `todo` → `doing`
   - Created dev-design.md, dev-plan.md, dev-log.md
   - Branch: user/<username>/<feature-id>
   - Ready to begin implementation
   ```

7. **Update meta.yaml**:
   - Set `status: doing`
   - Update `updated` date to today

#### For `doing → ready-to-ship`:

1. **Update meta.yaml**:
   - Set `status: ready-to-ship`
   - Update `updated` date

2. **Append to dev-log.md**:
   ```markdown
   ## <YYYY-MM-DD> — Ready to Ship

   - Status changed: `doing` → `ready-to-ship`
   - Implementation complete
   ```

#### For `ready-to-ship → shipped`:

1. **Update meta.yaml**:
   - Set `status: shipped`
   - Update `updated` date

2. **Append to dev-log.md**:
   ```markdown
   ## <YYYY-MM-DD> — Shipped

   - Status changed: `ready-to-ship` → `shipped`
   - Feature released
   ```

#### For `doing → todo`:

1. **Update meta.yaml**:
   - Set `status: todo`
   - Update `updated` date

2. **Append to dev-log.md**:
   ```markdown
   ## <YYYY-MM-DD> — Back to Requirements

   - Status changed: `doing` → `todo`
   - Reason: <ask user for reason>
   ```

### Step 4: Confirm to User

Present the result:

```
🔄 Status updated: <old_status> → <new_status>
🌿 Branch: user/<username>/<feature-id>
📋 Feature: <title>
🏷️ Status: <new_status> | Priority: <priority> | Progress: <progress>%
```

For `todo → doing`, also mention:
```
📄 Created: dev-design.md, dev-plan.md, dev-log.md

Next: Fill in dev-design.md, then use /supercrew:sync to generate task breakdown.
```

## Validation Rules

- Reject any transition not in the valid transitions graph
- Ensure `meta.yaml` required fields remain intact after edit
- `updated` date must be set to today (YYYY-MM-DD)


## Important

- Do NOT auto-push — user controls git operations
- When creating dev-* files, they go in the feature directory: `.supercrew/tasks/<feature-id>/`
- The dev-* files are only created on `todo → doing` transition
- For other transitions, only update `meta.yaml` and append to `dev-log.md`

````
