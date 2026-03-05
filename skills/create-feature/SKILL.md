````skill
---
name: create-feature
description: "Use when the user wants to create a new feature, start a new project initiative, or when /new-feature command is invoked. Creates the .supercrew/features/<id>/ directory with meta.yaml, prd.md, plan.md, and log.md."
---

# Create Feature

## Overview

Create a new feature in the `.supercrew/features/` directory. This skill guides the user through defining a feature, then generates the 4 required files.

## Critical Rule: Gather ALL Input Before ANY Action

**Ask all questions first, then execute all actions.** Never interleave questions with file creation or git operations. This prevents issues like creating files on the wrong branch or needing to stash/recreate work.

## Process

### Step 1: Gather ALL Feature Information

Ask the user for the following (one question at a time, or batch where appropriate):

1. **Feature title** — a short, descriptive name (e.g., "User Authentication", "Dashboard Redesign")
2. **Feature ID** — suggest a kebab-case slug derived from the title (e.g., `user-auth`, `dashboard-redesign`). Let the user confirm or override.
3. **Priority** — P0 (critical) | P1 (high) | P2 (medium, default) | P3 (low)
4. **Owner** — who is responsible for this feature (default: current git user name)
5. **Brief description** — one sentence describing what this feature does and why
6. **Create backlog PR?** — ask if they want to create a PR to make this feature visible to the team

Do NOT proceed to Step 2 until all questions are answered.

### Step 2: Execute All Actions

Now that all input is gathered, execute the appropriate workflow:

#### If creating a backlog PR (user said yes in Step 1):

1. Fetch latest and create a new branch named `backlog/<feature-id>` from `origin/main`
2. Create the feature directory and files **on the new branch** (see file specs below)
3. Stage, commit with message: `feat: add feature <feature-id> to backlog`
4. Push the branch and create a PR with:
   - **Title:** `feat: add <feature-id> to backlog`
   - **Body:** The feature title, description, priority, and owner
5. Switch back to the original branch

#### If NOT creating a backlog PR:

1. Create the feature directory and files on the current branch (see file specs below)
2. Remind the user that the feature won't be visible to the rest of the team until it's merged into remote main via a PR

### Feature Directory and Files

Create the directory `.supercrew/features/<feature-id>/` with 4 files.

**Use the templates in the plugin's `templates/` directory as reference for file structure.** Read the templates first, then generate files with the gathered information filled in.

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

<Brief description from user input>

## Requirements

<!-- To be refined during brainstorming -->

## Out of Scope

<!-- To be defined -->
```

#### File 3: `plan.md`

```markdown
---
total_tasks: 0
completed_tasks: 0
progress: 0
---

# <title> — Implementation Plan

## Tasks

- [ ] Task 1: (to be defined after design approval)
```

#### File 4: `log.md`

```markdown
# <title> — Progress Log

## <YYYY-MM-DD> — Feature Created

- Feature initialized with status: `todo`
- Owner: <owner>
- Priority: <priority>
```

### Step 3: Confirm and Summarize

After creating all files, present a summary:

```
✅ Feature created: <feature-id>
📁 Location: .supercrew/features/<feature-id>/
📄 Files: meta.yaml, prd.md, plan.md, log.md
🏷️ Status: todo | Priority: <priority> | Owner: <owner>
```

If a PR was created, include the PR URL.

### Step 4: Next Steps

Present next steps:

```
Next steps:
- Refine the requirements in prd.md
- Use /supercrew:work-on <feature-id> to start working (creates feature branch and design.md)
- The design.md for technical decisions will be created when entering 'doing' status
```

## Validation Rules

- Feature ID must be kebab-case (lowercase, hyphens only, no spaces)
- Feature ID must be unique (check `.supercrew/features/` for existing directories)
- Priority must be one of: P0, P1, P2, P3
- Status is always `todo` for new features
- `created` and `updated` dates use YYYY-MM-DD format (today's date)

## Important

- Do NOT start implementation after creating the feature. The next step is refining requirements.
- If `.supercrew/features/` directory doesn't exist yet, create it.
- If `.supercrew/` directory doesn't exist yet, create it.

````
