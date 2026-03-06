````skill
---
name: create-task
description: "Use when the user wants to create a new feature or task. Creates a backlog branch and the .supercrew/features/<id>/ directory with meta.yaml and prd.md with real content."
---

# Create Task

## Overview

Create a new feature in the `.supercrew/features/` directory. This skill guides the user through defining a feature, creates a user-namespaced backlog branch, and generates `meta.yaml` and `prd.md` with real content.

## Critical Rule: Gather ALL Input Before ANY Action

**Ask all questions first, then execute all actions.** Never interleave questions with file creation or git operations.

## Process

### Step 1: Gather ALL Feature Information

Ask the user for the following:

1. **Feature title** — a short, descriptive name (e.g., "User Authentication", "Dashboard Redesign")
2. **Feature ID** — suggest a kebab-case slug derived from the title (e.g., `user-auth`, `dashboard-redesign`). Let the user confirm or override.
3. **Priority** — P0 (critical) | P1 (high) | P2 (medium, default) | P3 (low)
4. **Owner** — who is responsible for this feature (default: current git user name)
5. **Background** — why is this feature needed? What problem does it solve?
6. **Requirements** — what must this feature do? List functional requirements.
7. **Out of Scope** — what is explicitly NOT included? (optional)

Do NOT proceed to Step 2 until all questions are answered.

### Step 2: Derive Username

Get the username for branch naming:

1. Run `git config user.name` to get the full name (e.g., "Steins Z")
2. Convert to lowercase and replace spaces with hyphens (e.g., `steins-z`)

### Step 3: Create Backlog Branch

1. Fetch latest: `git fetch origin main`
2. Create branch from `origin/main`: `git checkout -b user/<username>/backlog-<feature-id> origin/main`

Example: `user/steins-z/backlog-architecture-refinement`

### Step 4: Create Feature Directory and Files

Create the directory `.supercrew/features/<feature-id>/` with 2 files.

#### File 1: `meta.yaml`

```yaml
id: <feature-id>
title: "<title>"
status: todo
owner: "<owner>"
priority: <P0|P1|P2|P3>
teams: []
tags: []
created: "<YYYY-MM-DD>"
updated: "<YYYY-MM-DD>"
```

#### File 2: `prd.md`

```markdown
---
status: draft
reviewers: []
---

# <title>

## Background

<Background from user input — real content, not placeholders>

## Requirements

<Requirements from user input — real content, not placeholders>

## Out of Scope

<Out of scope from user input, or "To be defined" if not provided>
```

**Important:** The `prd.md` must contain real content from user input, not template placeholders.

### Step 5: Confirm and Summarize

After creating all files, present a summary:

```
✅ Feature created: <feature-id>
🌿 Branch: user/<username>/backlog-<feature-id>
📁 Location: .supercrew/features/<feature-id>/
📄 Files: meta.yaml, prd.md
🏷️ Status: todo | Priority: <priority> | Owner: <owner>
```

**Note:** The branch is local. Push when ready with `git push -u origin user/<username>/backlog-<feature-id>`

### Step 6: Next Steps

Present next steps:

```
Next steps:
- Review and refine prd.md if needed
- Push branch when ready: git push -u origin <branch-name>
- Use /supercrew:start <feature-id> to begin work (creates dev-* files and switches to work branch)
```

## Validation Rules

- Feature ID must be kebab-case (lowercase, hyphens only, no spaces)
- Feature ID must be unique (check `.supercrew/features/` for existing directories)
- Priority must be one of: P0, P1, P2, P3
- Status is always `todo` for new features
- `created` and `updated` dates use YYYY-MM-DD format (today's date)
- Username must be lowercase with hyphens (no spaces)

## Important

- Do NOT start implementation after creating the feature
- Do NOT auto-push — user controls when to push
- If `.supercrew/features/` directory doesn't exist yet, create it
- If `.supercrew/` directory doesn't exist yet, create it
- The `prd.md` must have real content, not empty template placeholders

````
