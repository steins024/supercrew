---
description: "Show the current status of all features in a table or kanban view."
disable-model-invocation: true
---

Show the status of all features tracked in `.supercrew/features/`.

1. List all feature directories in `.supercrew/features/`
2. For each feature, read `meta.yaml` to get: id, title, status, priority, owner
3. If `dev-plan.md` exists, read progress from frontmatter
4. Display as a table:

| ID | Title | Status | Priority | Progress | Owner |
|---|---|---|---|---|---|

Group by status if there are many features:
- **Todo**: Features not yet started
- **Doing**: Features in progress
- **Ready to Ship**: Features awaiting release
- **Shipped**: Completed features

Offer to start work on a specific feature: "Use `/supercrew:start <id>` to begin working on a feature."
