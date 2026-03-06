````skill
---
name: sync
description: "Use to sync all .supercrew updates during development — design iterations (dev-design.md), task updates (dev-plan.md), and progress logging (dev-log.md). Does not auto-commit or push."
---

# Sync Supercrew

## Overview

Sync all `.supercrew` updates for the active feature. This skill handles:
- Design iteration updates (`dev-design.md`)
- Task breakdown and progress updates (`dev-plan.md`)
- Session logging (`dev-log.md`)

## When to Use

- After making design decisions → sync `dev-design.md`
- After completing tasks or adding new ones → sync `dev-plan.md`
- At session checkpoints or end → sync `dev-log.md`
- User explicitly asks to sync with `/supercrew:sync`

## Process

### Step 1: Identify Active Feature

- Use the active feature from the current session context
- If no active feature, ask the user which feature to sync
- Verify the feature is in `doing` status (dev-* files should exist)

### Step 2: Gather Sync Context

Ask the user (or infer from conversation):

1. **Design changes** — Any new design decisions, architecture changes, or implementation notes?
2. **Task updates** — Any tasks completed? New tasks to add? Task breakdown needed?
3. **Session notes** — What was accomplished? Any issues? What's next?

### Step 3: Sync dev-design.md

If there are design updates:

1. Read current `dev-design.md`
2. Update the relevant sections:
   - **Design Decisions**: Add new decisions with rationale
   - **Architecture**: Update diagrams or component descriptions
   - **Implementation Notes**: Add edge cases, gotchas, learnings
3. Write the updated file

### Step 4: Sync dev-plan.md

#### Mode A: Generate Task Breakdown

If `dev-plan.md` has no real tasks yet (just placeholder):

1. Read `dev-design.md` and `prd.md` for context
2. Break down into implementation tasks:
   - Each task should be completable in one coding session
   - Each task should be independently verifiable
   - Order by dependency (earlier tasks don't depend on later ones)
3. Write tasks as checkbox items:

```markdown
---
total_tasks: <count>
completed_tasks: 0
progress: 0
---

# <title> — Implementation Plan

## Tasks

- [ ] Task 1: <description>
- [ ] Task 2: <description>
...
```

#### Mode B: Update Progress

If tasks exist:

1. Read `dev-plan.md`
2. Update task checkboxes based on what was completed:
   - Mark completed tasks: `- [ ]` → `- [x]`
   - Add any new tasks discovered during implementation
3. Recalculate frontmatter:
   - `total_tasks`: count of all `- [ ]` and `- [x]` items
   - `completed_tasks`: count of `- [x]` items
   - `progress`: `round(completed_tasks / total_tasks * 100)`
4. Write the updated file

### Step 5: Sync dev-log.md

Append a session entry:

```markdown
## <YYYY-MM-DD> — <Brief summary>

### Completed
- <What was accomplished>
- <Tasks completed from plan>

### Issues
- <Any blockers or problems encountered>
- (or "None" if no issues)

### Next Steps
- <What to do next session>
```

### Step 6: Update meta.yaml

1. Update `updated` date to today (YYYY-MM-DD)
2. If progress changed significantly, update any progress field if present

### Step 7: Confirm to User

Present the sync summary:

```
✅ Sync complete for: <feature-id>

📐 dev-design.md: <updated/unchanged>
📋 dev-plan.md: <X/Y tasks complete> (<progress>%)
📝 dev-log.md: Session entry added

Files updated locally. Commit and push when ready.
```

If progress reached 100%, suggest:
```
🎉 All tasks complete! Consider marking as ready-to-ship:
   "mark as ready to ship" or use do-task skill
```

## Validation Rules

- Feature must be in `doing` status (dev-* files must exist)
- `total_tasks` must equal actual count of task checkbox items
- `completed_tasks` must equal actual count of checked items
- `progress` must be `round(completed_tasks / total_tasks * 100)`
- Session log entries must have date and summary

## Important

- Do NOT auto-commit or push — user controls git operations
- All dev-* files are in `.supercrew/tasks/<feature-id>/`
- If dev-* files don't exist, suggest using `do-task` to start work first
- Keep session log entries concise but informative
- When generating task breakdown, make tasks granular and actionable

````
