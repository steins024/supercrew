---
status: approved
reviewers: []
# approved_by: ""
---

# SuperCrew MVP - AI Integration Plugin

## Background

Based on the `user/steinsz/supercrew_schema` branch demo implementation, this feature migrates the kanban from `.team/` resource-oriented to `.supercrew/features/` feature-oriented data model. The MVP focuses on the AI integration plugin (Phase 1) that creates and manages `.supercrew/` directories in user repos.

The plugin is developed in the monorepo's `plugins/supercrew/` subdirectory, following the superpowers plugin architecture (skills + hooks + commands). MVP loads via absolute path (`/plugin marketplace add /path/to/supercrew/plugins/supercrew`), with Post-MVP extraction to independent repo for marketplace publication.

## Requirements

### 1.1 Plugin Directory Structure
- `.claude-plugin/marketplace.json` for Claude Code marketplace
- `skills/` directory with SKILL.md files for each skill
- `commands/` directory with slash command definitions
- `hooks/` directory with SessionStart hook
- `templates/` directory with file templates

### 1.2 Schema Definition
- `SupercrewStatus` enum: `todo -> doing -> ready-to-ship -> shipped`
- `FeaturePriority`: `P0 | P1 | P2 | P3`
- 4 files per feature in `.supercrew/features/<feature-id>/`:
  - `meta.yaml`: id, title, status, owner, priority, teams, tags, dates
  - `design.md`: YAML frontmatter (status, reviewers) + markdown body
  - `plan.md`: YAML frontmatter (total_tasks, completed_tasks, progress) + task checklist
  - `log.md`: chronological progress log

### 1.3 Skills Implementation
- `create-feature`: Create feature directory with 4 files via guided prompts
- `update-status`: Update meta.yaml status following valid state transitions
- `sync-plan`: Generate/update plan.md task breakdown and progress
- `log-progress`: Append session log to log.md
- `managing-features`: Coordinate other skills, detect inconsistencies
- `using-supercrew`: Session start context injection, skill invocation rules

### 1.4 Hooks
- SessionStart hook: Detect `.supercrew/features/`, match active feature via branch name, inject context

### 1.5 Commands
- `/supercrew:new-feature`: Trigger create-feature skill
- `/supercrew:feature-status`: Display all features in table format
- `/supercrew:work-on <id>`: Switch active feature for session

## Out of Scope

- **Phase 2 (Backend)**: Read-only Kanban service via GitHub API (moved to separate repo)
- **Phase 3 (Frontend)**: Feature-centric kanban board UI (moved to separate repo)
- **Sprint mechanism**: Time-boxed iterations (Post-MVP Iteration 2)
- **Cross-feature coordination**: Dependency visualization (Post-MVP Iteration 3)
- **Release coordination**: Release readiness checks (Post-MVP Iteration 4)
- **pre-commit hook**: Schema validation on commit (deferred)
