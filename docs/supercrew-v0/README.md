# SuperCrew v0 Documentation

## Table of Contents

- [Overview](#overview)
- [Skill Workflows & Scenarios](#skill-workflows--scenarios)
- [SessionStart Hook](#sessionstart-hook)
- [Commands Reference](#commands-reference)
- [Skills Reference](#skills-reference)
- [Templates Reference](#templates-reference)

---

## Overview

### What It Is

**SuperCrew** is a **feature lifecycle management plugin** for Claude Code. It provides structured tracking of features from ideation through completion, with automatic context injection and skill-based workflows.

### What It Does for Users

#### Core Value Proposition

- **Structured feature tracking** in `.supercrew/features/<feature-id>/` directories
- **Automatic context injection** at session start (knows what features exist, which is active)
- **Enforced lifecycle workflow** with valid status transitions
- **Progress tracking** via plan.md with task checklists
- **Session logging** for continuity across conversations

#### Per-Feature File Structure

Each feature gets these files:

| File | Purpose | When Created |
|------|---------|--------------|
| `meta.yaml` | ID, title, status, owner, priority, dates | Feature creation (`todo`) |
| `prd.md` | Background, requirements, out of scope | Feature creation (`todo`) |
| `design.md` | Design decisions, architecture, implementation notes | Work starts (`doing`) |
| `plan.md` | Task breakdown with checkboxes, progress tracking | Feature creation (`todo`) |
| `log.md` | Chronological session logs | Feature creation (`todo`) |

### Status Lifecycle

```
todo → doing → ready-to-ship → shipped
  ↑      │
  └──────┘
```

- **todo**: Feature defined with requirements (prd.md)
- **doing**: Work in progress, design.md created, tasks being completed
- **ready-to-ship**: Implementation complete, ready for release
- **shipped**: Released

### Key Design Decisions

1. **Branch-per-feature**: Each feature gets `feature/<id>` branch for isolation
2. **Remote-first reads**: SessionStart and feature-status prefer `origin/feature/<id>` for latest state
3. **Commit-and-push**: Status/plan/log changes are committed and pushed to keep remote in sync
4. **Backlog PR option**: New features can optionally create a `backlog/<id>` PR to make them visible to the team

---

## Skill Workflows & Scenarios

### Skill Trigger Summary

| User Action | Skill Invoked |
|-------------|---------------|
| "Create a new feature" | `create-feature` |
| "Start working on this" | `update-status` (todo → doing) |
| "Implementation done" | `update-status` (doing → ready-to-ship) |
| "Ship it" | `update-status` (ready-to-ship → shipped) |
| "Generate tasks from design" | `sync-plan` |
| "Update task progress" | `sync-plan` |
| "End of session / checkpoint" | `log-progress` |
| "Show all features" | `feature-status` / `kanban` |
| "Switch to feature X" | `work-on` command |
| General feature work | `managing-features` (orchestration) |

### Scenario 1: Starting a New Feature

```
User: "I want to add user authentication"
       ↓
   create-feature
       ↓
   Questions: title, ID, priority, owner, description, PR?
       ↓
   Creates: .supercrew/features/user-auth/
            - meta.yaml (status: todo)
            - prd.md (status: draft)
            - plan.md (0 tasks)
            - log.md (creation entry)
```

### Scenario 2: Working on an Existing Feature

```
User: "/supercrew:work-on user-auth"
       ↓
   work-on command
       ↓
   - Checkout/create branch feature/user-auth
   - Auto-advance todo → doing
   - Create design.md from template
   - Push branch to remote
   - Load feature context
```

### Scenario 3: Implementation In Progress

```
User: "I finished the login form"
       ↓
   sync-plan (check off tasks, update progress %)
       ↓
   (If progress hits 100%)
   update-status (doing → ready-to-ship)
```

### Scenario 4: Ready to Ship

```
User: "Ship it" or "We're releasing this"
       ↓
   update-status (ready-to-ship → shipped)
```

### Scenario 5: End of Session

```
User: "I'm done for today"
       ↓
   log-progress
       ↓
   Appends to log.md:
   - What was done
   - Tasks completed
   - Issues encountered
   - Next steps
```

### Scenario 6: Checking Overall Status

```
User: "/supercrew:feature-status" or "show me the kanban"
       ↓
   feature-status or kanban skill
       ↓
   Displays table of all features with status/progress
```

---

## SessionStart Hook

### How the Hook Works

The **SessionStart hook** (`hooks/session-start`) runs automatically at the beginning of each Claude Code session and:

1. Checks if `.supercrew/features/` exists
2. Builds a summary table of all features (reads from remote branches if available)
3. Detects active feature by matching `feature/<id>` branch pattern
4. Injects the `using-supercrew` skill + feature context into Claude's context

### Context Injection

This ensures Claude always knows:

- What features exist
- Which feature is currently active
- The full skill workflow instructions

### Hook Configuration

Located in `hooks/hooks.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "'${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd' session-start",
            "async": false
          }
        ]
      }
    ]
  }
}
```

### Feature Detection Logic

1. **Remote-first reads**: The hook prefers `origin/feature/<id>` branches for the latest feature state
2. **Branch matching**: If current branch is `feature/<id>`, that feature becomes active
3. **Fallback**: If no branch match, prompts user to select a feature with `/supercrew:work-on`

### Output Structure

The hook outputs JSON with:

- `additional_context`: Full context string injected into Claude
- `hookSpecificOutput.hookEventName`: "SessionStart"
- `hookSpecificOutput.additionalContext`: Same as additional_context

This context includes:
- The `using-supercrew` skill content
- A table of all features with ID, title, status, priority, progress, owner
- Active feature details (if matched) including meta.yaml, recent log entries, and plan progress

---

## Commands Reference

SuperCrew provides three user-invocable commands:

### /supercrew:new-feature

**Description**: Create a new feature for tracking in `.supercrew/features/`

**Usage**: `/supercrew:new-feature`

**What it does**:
1. Invokes the `create-feature` skill
2. Guides through defining title, priority, owner, and description
3. Generates `meta.yaml`, `prd.md`, `plan.md`, and `log.md`
4. Optionally creates a backlog PR to make the feature visible to the team

### /supercrew:feature-status

**Description**: Show the current status of all features

**Usage**: `/supercrew:feature-status`

**What it does**:
1. Lists all features in `.supercrew/features/`
2. Displays a summary table:

| ID | Title | Status | Priority | Progress | Owner |
|---|---|---|---|---|---|

3. For each feature:
   - Checks remote branch `origin/feature/<id>` for latest data
   - Falls back to local files if no remote branch
   - Extracts progress from `plan.md` frontmatter

### /supercrew:work-on \<id\>

**Description**: Switch the active feature for this session

**Usage**: `/supercrew:work-on <feature-id>`

**What it does**:
1. Verifies the feature directory exists
2. Creates or switches to `feature/<feature-id>` branch
3. Auto-advances status from `todo` to `doing` if applicable (and creates `design.md`)
4. Pushes the branch to remote
5. Loads feature context (meta.yaml, plan.md progress, last log entry)
6. Sets this feature as active for all subsequent skill operations

**Output example**:
```
🔄 Switched active feature to: user-auth
🌿 Branch: feature/user-auth
📋 Title: User Authentication
🏷️ Status: doing | Priority: P1 | Progress: 0%

All supercrew skills will now operate on this feature.
```

---

## Skills Reference

SuperCrew provides 7 skills that Claude uses to manage feature lifecycle:

### using-supercrew

**Purpose**: Establishes how supercrew skills work at session start

**When used**: Automatically loaded via SessionStart hook

**Key behavior**:
- Defines the rule: invoke relevant skills BEFORE any response or action
- Provides the skill trigger mapping table
- Lists "red flag" thoughts that indicate skill-skipping rationalization

### create-feature

**Purpose**: Create a new feature with all required files

**When triggered**: User wants to create/start a new feature

**Process**:
1. Gather ALL input first (title, ID, priority, owner, description, PR preference)
2. Execute actions (create branch if PR, create files, commit)
3. Confirm and summarize

**Creates**:
- `.supercrew/features/<id>/meta.yaml`
- `.supercrew/features/<id>/prd.md`
- `.supercrew/features/<id>/plan.md`
- `.supercrew/features/<id>/log.md`

### update-status

**Purpose**: Change a feature's status following valid transitions

**When triggered**: User says "start working", "done", "ship it" or context implies status change

**Valid transitions**:
```
todo → doing               (work started, design.md created)
doing → ready-to-ship      (implementation complete)
ready-to-ship → shipped    (released)
doing → todo               (back to requirements)
```

**Process**:
1. Identify the feature (active or ask user)
2. Determine new status from context
3. Create design.md if transitioning to `doing`
4. Update `meta.yaml` status and updated date
5. Log the change in `log.md`
6. Commit and push to remote

### sync-plan

**Purpose**: Generate or update plan.md with task breakdown

**When triggered**: Entering `doing` status and need tasks, or tasks completed

**Two modes**:

1. **Generate mode**: When entering `doing`, create task breakdown from `design.md`
2. **Update mode**: During implementation, sync `completed_tasks` and `progress` with actual checklist state

**Process**:
1. Read design or plan as appropriate
2. Generate tasks or count checked items
3. Update frontmatter (`total_tasks`, `completed_tasks`, `progress`)
4. Commit and push to remote

### log-progress

**Purpose**: Append structured session entry to log.md

**When triggered**: End of session, milestone reached, or long session without log

**Process**:
1. Gather session context (what was done, tasks completed, issues, next steps)
2. Append entry to `log.md`
3. Update `meta.yaml` updated date
4. Commit and push to remote

**Entry format**:
```markdown
## YYYY-MM-DD — Brief summary

### Completed
- What was accomplished
- Tasks completed from plan

### Issues
- Any blockers or problems

### Next Steps
- What to do next session
```

### managing-features

**Purpose**: Orchestrate other skills during a coding session

**When triggered**: Working in a project with `.supercrew/features/`

**Key behaviors**:
- Suggests `update-status` when context implies status change
- Triggers `sync-plan` after entering `doing` or task completion
- Reminds about `log-progress` at session end or milestones
- Performs proactive consistency checks (status vs reality, progress drift)

### kanban

**Purpose**: Display features as a grouped kanban board

**When triggered**: User asks for kanban board, dashboard, or project overview

**Process**:
1. Run `scripts/kanban.sh`
2. Display ANSI-colored output directly (no re-rendering)
3. Offer `/supercrew:work-on <id>` for specific features

---

## Templates Reference

Templates are located in the `templates/` directory and define the initial structure of feature files.

### meta.yaml.tmpl

Metadata file for feature tracking:

```yaml
id: {{id}}
title: "{{title}}"
status: todo
owner: "{{owner}}"
priority: P2
teams: []
tags: []
created: "{{date}}"
updated: "{{date}}"
# target_release: ""
# blocked_by: []
```

**Fields**:
| Field | Description |
|-------|-------------|
| `id` | Kebab-case feature identifier |
| `title` | Human-readable feature name |
| `status` | Current lifecycle status (todo, doing, ready-to-ship, shipped) |
| `owner` | Person responsible for the feature |
| `priority` | P0 (critical) to P3 (low) |
| `teams` | Teams involved (optional) |
| `tags` | Categorization tags (optional) |
| `created` | Creation date (YYYY-MM-DD) |
| `updated` | Last update date (YYYY-MM-DD) |

### prd.md.tmpl

Product Requirements Document for feature specification (created at feature creation):

```markdown
---
status: draft
reviewers: []
# approved_by: ""
---

# {{title}}

## Background

<!-- Why is this feature needed? What problem does it solve? -->

## Requirements

<!-- What must this feature do? List functional requirements. -->

## Out of Scope

<!-- What is explicitly NOT included in this feature? -->
```

**Frontmatter fields**:
| Field | Description |
|-------|-------------|
| `status` | `draft` or `approved` |
| `reviewers` | List of reviewers |
| `approved_by` | Who approved the PRD (optional) |

### design.md.tmpl

Technical design document (created when entering `doing` status):

```markdown
---
status: draft
reviewers: []
# approved_by: ""
---

# {{title}} — Technical Design

## Design Decisions

<!-- Key architectural and implementation decisions. Why did you choose this approach? -->

## Architecture

<!-- High-level architecture, data flow, component interactions. -->

## Implementation Notes

<!-- Specific implementation details, edge cases, gotchas. -->
```

**Frontmatter fields**:
| Field | Description |
|-------|-------------|
| `status` | `draft` or `approved` |
| `reviewers` | List of reviewers |
| `approved_by` | Who approved the design (optional) |

### plan.md.tmpl

Implementation plan with task tracking:

```markdown
---
total_tasks: 0
completed_tasks: 0
progress: 0
---

# {{title}} — Implementation Plan

<!-- Task breakdown will be generated after design approval. -->
<!-- Each task should be a checkbox item with clear acceptance criteria. -->

## Tasks

- [ ] Task 1: (to be defined)
```

**Frontmatter fields**:
| Field | Description |
|-------|-------------|
| `total_tasks` | Total number of task items |
| `completed_tasks` | Number of checked items |
| `progress` | Percentage complete (0-100) |

### log.md.tmpl

Progress log for session tracking:

```markdown
# {{title}} — Progress Log

## {{date}} — Feature Created

- Feature initialized with status: `todo`
- Owner: {{owner}}
- Priority: P2
```

**Entry format**:
Each session adds an entry with:
- Date and brief summary in heading
- Completed items
- Issues encountered
- Next steps

### Template Variables

| Variable | Description |
|----------|-------------|
| `{{id}}` | Feature ID (kebab-case) |
| `{{title}}` | Feature title |
| `{{owner}}` | Feature owner name |
| `{{date}}` | Current date (YYYY-MM-DD) |
