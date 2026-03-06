---
status: draft
reviewers: []
# approved_by: ""
---

# Architecture Refinement for Status Schema Alignment — Technical Design

## Overview

Simplify supercrew from 7 skills to 3, with clearer responsibility boundaries and user-namespaced branch conventions.

## Design Decisions

### 1. Skill Consolidation (7 → 3)

| Previous Skills | New Skill | Rationale |
|-----------------|-----------|-----------|
| create-feature | **create-task** | Renamed, streamlined |
| update-status, managing-features | **do-task** | Merged lifecycle management |
| sync-plan, log-progress, kanban | **sync-supercrew** | Merged sync operations |
| using-supercrew | (embedded in session-start hook) | Guidance moved to hook context |

### 2. Branch Naming Convention

User-namespaced branches for better team collaboration:

| Stage | Branch Pattern | Example |
|-------|----------------|---------|
| Backlog | `user/<username>/backlog-<feature-name>` | `user/steins-z/backlog-architecture-refinement` |
| Active work | `user/<username>/<feature-name>` | `user/steins-z/architecture-refinement` |

**Username format:** Lowercase, hyphenated full name from `git config user.name`
- "Steins Z" → `steins-z`
- "John Doe" → `john-doe`

### 3. File Structure Changes

**At feature creation (create-task):**
- `meta.yaml` — Feature metadata
- `prd.md` — Product requirements with real content

**When work starts (do-task, todo → doing):**
- `dev-design.md` — Technical design decisions
- `dev-plan.md` — Task breakdown with checkboxes
- `dev-log.md` — Progress log

**Rationale for `dev-` prefix:**
- Clear separation between requirements (prd.md) and implementation artifacts
- Signals these are work-in-progress documents
- Deferred creation — only when work actually starts

### 4. No Auto-Push

Neither `create-task` nor `sync-supercrew` auto-pushes to remote. User controls when to push.

## Architecture

### Skills

#### create-task

**Purpose:** Create a new feature in backlog

**Trigger:** User wants to create a new feature

**Process:**
1. Gather feature info (title, ID, priority, owner, description)
2. Fetch latest `origin/main`
3. Create branch `user/<username>/backlog-<feature-name>` from `origin/main`
4. Create `.supercrew/features/<feature-id>/` directory
5. Create `meta.yaml` with status: `todo`
6. Create `prd.md` with real content from user input
7. Confirm to user (no auto-push)

**Files created:**
```
.supercrew/features/<feature-id>/
├── meta.yaml
└── prd.md
```

#### do-task

**Purpose:** Manage full feature lifecycle

**Triggers:**
- User wants to start working on a feature
- User says "done", "ready to ship", "ship it"
- Context implies status change

**Status Transitions:**
```
todo → doing               (creates dev-* files, switches branch)
doing → ready-to-ship      (implementation complete)
ready-to-ship → shipped    (released)
doing → todo               (back to requirements)
```

**Process for todo → doing:**
1. Identify feature (from context or user input)
2. Determine branch base:
   - If on `user/<username>/backlog-<feature-name>` → branch from current
   - Otherwise → branch from `origin/main`
3. Create/switch to `user/<username>/<feature-name>` branch
4. Create `dev-design.md` from template
5. Create `dev-plan.md` with task breakdown (generated from prd.md)
6. Create `dev-log.md` with initial entry
7. Update `meta.yaml` status to `doing`
8. Confirm to user

**Process for other transitions:**
1. Validate transition is allowed
2. Update `meta.yaml` status and updated date
3. Add entry to `dev-log.md`
4. Confirm to user

#### sync-supercrew

**Purpose:** Sync all .supercrew updates during work

**Triggers:**
- Design iteration (dev-design.md updates)
- Task completion or breakdown changes (dev-plan.md updates)
- Session checkpoint or end (dev-log.md updates)
- User explicitly asks to sync

**Process:**
1. Identify active feature
2. Sync `dev-design.md` — capture design decisions/changes
3. Sync `dev-plan.md` — update task checkboxes, add new tasks, recalculate progress
4. Sync `dev-log.md` — append session entry with completed/issues/next steps
5. Update `meta.yaml` updated date
6. Confirm to user (no auto-push)

**No commit/push** — user controls git operations

### Commands

| Command | Purpose | Invokes |
|---------|---------|---------|
| `/supercrew:create` | Create a new feature in backlog | `create-task` |
| `/supercrew:start <id>` | Start working on a feature | `do-task` (todo → doing) |
| `/supercrew:sync` | Sync all .supercrew updates | `sync-supercrew` |
| `/supercrew:status` | Show all features (kanban/table) | Display only |

**Status transitions** (`done`, `ship`) are handled conversationally via `do-task` skill rather than explicit commands.

### Hooks

#### session-start

**Purpose:** Inject feature context at session start

**Process:**
1. Check if `.supercrew/features/` exists
2. Build summary table of all features
3. Detect active feature by matching branch pattern:
   - `user/<username>/<feature-name>` → active feature
   - `user/<username>/backlog-<feature-name>` → backlog feature
4. Inject context with:
   - Skill routing guidance (which skill for which action)
   - Feature table (ID, title, status, priority, progress, owner)
   - Active feature details (if matched)

### Templates

#### meta.yaml

```yaml
id: {{id}}
title: "{{title}}"
status: todo
owner: "{{owner}}"
priority: {{priority}}
teams: []
tags: []
created: "{{date}}"
updated: "{{date}}"
```

#### prd.md

```markdown
---
status: draft
reviewers: []
---

# {{title}}

## Background

{{description}}

## Requirements

{{requirements}}

## Out of Scope

{{out_of_scope}}
```

#### dev-design.md

```markdown
---
status: draft
reviewers: []
---

# {{title}} — Technical Design

## Design Decisions

<!-- Key architectural and implementation decisions -->

## Architecture

<!-- High-level architecture, data flow, component interactions -->

## Implementation Notes

<!-- Specific implementation details, edge cases, gotchas -->
```

#### dev-plan.md

```markdown
---
total_tasks: 0
completed_tasks: 0
progress: 0
---

# {{title}} — Implementation Plan

## Tasks

<!-- Task breakdown generated from prd.md -->

- [ ] Task 1
- [ ] Task 2
```

#### dev-log.md

```markdown
# {{title}} — Development Log

## {{date}} — Work Started

- Status changed: `todo` → `doing`
- Created dev-design.md, dev-plan.md, dev-log.md
- Ready to begin implementation
```

## Migration Notes

### From v0 to v1

1. **Existing features** — No migration needed for `meta.yaml` and `prd.md`
2. **Existing dev files** — Rename if desired:
   - `design.md` → `dev-design.md`
   - `plan.md` → `dev-plan.md`
   - `log.md` → `dev-log.md`
3. **Branch rename** — Optional, existing branches continue to work
4. **Skills** — Old skill names deprecated, new skills take over

### Backward Compatibility

- Session-start hook should recognize both old (`feature/<id>`) and new (`user/<username>/<id>`) branch patterns during transition period
- Old commands (`/supercrew:new-feature`, `/supercrew:work-on`) can alias to new commands

## Summary

| Component | v0 | v1 |
|-----------|----|----|
| Skills | 7 | 3 |
| Commands | 3 | 4 |
| Files at creation | 3 | 2 |
| Dev file prefix | (none) | `dev-` |
| Branch pattern | `feature/<id>` | `user/<username>/<id>` |
| Auto-push | Yes | No |
