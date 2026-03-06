---
total_tasks: 14
completed_tasks: 14
progress: 100
---

# Architecture Refinement for Status Schema Alignment — Implementation Plan

## Phase 1: Skills

### create-task skill
- [x] Create `skills/create-task/SKILL.md` with full process documentation
- [x] Define username extraction logic (git config → lowercase hyphenated)
- [x] Define branch creation from `origin/main`
- [x] Define `meta.yaml` and `prd.md` generation with real content

### do-task skill
- [x] Create `skills/do-task/SKILL.md` with full lifecycle management
- [x] Implement `todo → doing` transition (branch creation, dev-* file creation)
- [x] Implement `doing → ready-to-ship` transition
- [x] Implement `ready-to-ship → shipped` transition
- [x] Implement `doing → todo` back-transition

### sync-supercrew skill
- [x] Create `skills/sync-supercrew/SKILL.md` with sync process
- [x] Define dev-design.md sync logic
- [x] Define dev-plan.md sync logic (task updates, progress calculation)
- [x] Define dev-log.md sync logic (session entries)

## Phase 2: Commands

- [x] Create `commands/create.md` → invokes create-task
- [x] Create `commands/start.md` → invokes do-task (todo → doing)
- [x] Create `commands/sync.md` → invokes sync-supercrew
- [x] Create `commands/status.md` → display kanban/table

## Phase 3: Hooks

- [x] Update `hooks/session-start` script
  - [x] Support new branch patterns (`user/<username>/<feature>`, `user/<username>/backlog-<feature>`)
  - [x] Maintain backward compatibility with old patterns (`feature/<id>`)
  - [x] Embed skill routing guidance (replaces using-supercrew skill)

## Phase 4: Templates

- [x] Update `templates/meta.yaml.tmpl`
- [x] Update `templates/prd.md.tmpl` (real content placeholders)
- [x] Create `templates/dev-design.md.tmpl`
- [x] Create `templates/dev-plan.md.tmpl`
- [x] Create `templates/dev-log.md.tmpl`
- [x] Remove old templates (design.md.tmpl, plan.md.tmpl, log.md.tmpl)

## Phase 5: Cleanup

- [x] Remove deprecated skills (create-feature, update-status, sync-plan, log-progress, managing-features, kanban, using-supercrew)
- [x] Remove deprecated commands (new-feature, work-on, feature-status)
- [x] Update plugin version
