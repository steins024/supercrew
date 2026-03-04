---
description: "Show the current status of all features tracked in .supercrew/features/. Displays a summary table with id, title, status, progress, priority, and owner."
disable-model-invocation: true
---

List all features in `.supercrew/features/` and display them in a table format:

| ID | Title | Status | Priority | Progress | Owner |
|---|---|---|---|---|---|

For each feature directory in `.supercrew/features/`:
1. Check if a remote branch `origin/feature/<id>` exists (via `git show origin/feature/<id>:.supercrew/features/<id>/meta.yaml`).
2. If the remote branch exists, read `meta.yaml` and `plan.md` from it (these are the latest versions). Otherwise, fall back to the local files.
3. From `meta.yaml`: extract id, title, status, priority, owner.
4. From `plan.md` frontmatter: extract progress (completed_tasks/total_tasks).
5. Display the row.

If no `.supercrew/features/` directory exists, tell the user to create their first feature with `/supercrew:new-feature`.
