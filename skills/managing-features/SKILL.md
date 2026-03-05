````skill
---
name: managing-features
description: "Use when working in a project with .supercrew/features/ directory. Guides feature lifecycle management — coordinates status updates, plan syncing, and progress logging automatically throughout the session."
---

# Managing Features

## Overview

This skill provides lifecycle management guidance when working on a project that uses `.supercrew/features/`. It coordinates when to invoke other supercrew skills (`update-status`, `sync-plan`, `log-progress`) based on what's happening in the session.

## Active Feature Context

At session start, the SessionStart hook determines which feature is active. This skill operates on that active feature. Key context to be aware of:

- **Active feature**: The feature this session is focused on (matched by branch name or user selection)
- **Feature status**: Current status from `meta.yaml`
- **Plan progress**: `completed_tasks / total_tasks` from `plan.md`
- **Last log entry**: Most recent entry from `log.md`

## When to Trigger Other Skills

### Trigger `update-status`

- When user says "start working" or begins implementation and feature is in `todo` → suggest `todo → doing`
- When all `plan.md` tasks are complete → suggest `doing → ready-to-ship`
- When user says "ship it" or "released" → suggest `ready-to-ship → shipped`
- When requirements need rework → suggest `doing → todo`

### Trigger `sync-plan`

- When feature enters `doing` status and `plan.md` has no real tasks → generate task breakdown from `design.md`
- After completing tasks in code → update `plan.md` progress counters
- When user asks for progress update → sync and report

### Trigger `log-progress`

- When the session is ending (user says goodbye, wrapping up, etc.)
- When a significant milestone is reached (multiple tasks completed)
- When the user has been working for a while without a log entry

## Proactive Checks

During the session, periodically check for inconsistencies:

1. **Status vs. reality**: Feature status says `todo` but user is writing code → suggest status update to `doing`
2. **Progress drift**: `plan.md` `completed_tasks` doesn't match actual checked items → suggest sync
3. **Missing log**: Long session with no `log.md` entry → remind user to log before ending
4. **Missing design**: Feature is `doing` but has no `design.md` → warn and offer to create it

## Communication Style

- Be concise — one-line suggestions, not paragraphs
- Frame as suggestions, not commands: "Would you like me to update the feature status to `doing`?"
- Group related updates: "I'll update the status and sync the plan progress."
- Don't interrupt flow — wait for natural pauses (task completion, topic change) to suggest updates

````
