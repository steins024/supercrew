````skill
---
name: update-status
description: "Use when a feature's status needs to change — when starting work, completing implementation, shipping, or going back to requirements. Updates meta.yaml status field following the valid state transition graph."
---

# Update Feature Status

## Overview

Update the `status` field in a feature's `meta.yaml`. Status changes must follow the valid transition graph.

## Valid Status Transitions

```
todo → doing               (work started, design.md and plan.md created)
doing → ready-to-ship      (implementation complete)
ready-to-ship → shipped    (released)
doing → todo               (back to requirements)
```

**Invalid transitions** (e.g., `todo → shipped`, `shipped → doing`) must be rejected with an explanation of which intermediate steps are needed.

## Process

### Step 1: Identify the Feature

- Use the active feature from the current session context (set by SessionStart hook or `/work-on`)
- If no active feature, ask the user which feature to update

### Step 2: Determine New Status

Infer the appropriate status transition based on context:

| Signal | Suggested Transition |
|--------|---------------------|
| User says "start working on this" or begins work | `todo → doing` |
| Implementation complete, ready for release | `doing → ready-to-ship` |
| User says "ship it" or "released" | `ready-to-ship → shipped` |
| Requirements need rework | `doing → todo` |
| All tasks in `plan.md` checked off, tests pass | `doing → ready-to-ship` |

If the transition is ambiguous, ask the user to confirm.

### Step 3: Create design.md and plan.md (for todo → doing transition)

When transitioning from `todo` to `doing`:
1. Read the `templates/design.md.tmpl` template
2. Create `design.md` in the feature directory with the template content
3. Fill in the `{{title}}` placeholder with the feature title
4. Read the `templates/plan.md.tmpl` template
5. Create `plan.md` in the feature directory with the template content
6. Fill in the `{{title}}` placeholder with the feature title

### Step 4: Update `meta.yaml`

1. Read the current `meta.yaml`
2. Validate the transition is legal (see graph above)
3. Update `status` to the new value
4. Update `updated` date to today (YYYY-MM-DD)
5. Write the file

### Step 5: Log the Change

Append to `log.md`:

```markdown
## <YYYY-MM-DD> — Status: <old_status> → <new_status>

- Reason: <brief reason for the transition>
```

### Step 6: Commit and Push to Remote

1. Commit the updated files (`meta.yaml`, `log.md`, and `design.md`/`plan.md` if created) to the feature branch (`feature/<id>`)
2. Push to `origin/feature/<id>` so the kanban (SessionStart hook) reflects the change

If already on the feature branch, commit and push directly. If on another branch (e.g., `main`), checkout the feature branch, commit, push, then return to the original branch.

## Validation

- Reject any transition not in the valid transitions graph
- Ensure `meta.yaml` required fields remain intact after edit
- `updated` date must be set to today

````
