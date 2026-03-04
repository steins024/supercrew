---
description: "Switch the active feature for this session. Usage: /work-on <feature-id>. All subsequent supercrew skill operations will target this feature."
disable-model-invocation: true
---

The user wants to switch the active feature for this session.

1. Check if the user provided a feature ID argument. If not, list available features from `.supercrew/features/` and ask them to choose.
2. Verify the feature directory `.supercrew/features/<feature-id>/` exists.
3. **Create or switch to the feature branch:**
   - Check if a local branch `feature/<feature-id>` already exists.
   - If it exists, switch to it with `git checkout feature/<feature-id>`.
   - If it doesn't exist, create and switch to it with `git checkout -b feature/<feature-id>`.
4. **Auto-advance status if `planning`:**
   - Read the feature's `meta.yaml`. If `status` is `planning`, update it to `designing`.
   - Stage and commit the change: `git add .supercrew/features/<feature-id>/meta.yaml && git commit -m "chore: advance <feature-id> status to designing"`.
5. **Push the branch to remote:**
   - Run `git push -u origin feature/<feature-id>` to ensure the remote branch exists and is tracked.
6. Read the feature's `meta.yaml`, `plan.md` progress, and last `log.md` entry.
7. Confirm to the user:

```
🔄 Switched active feature to: <feature-id>
🌿 Branch: feature/<feature-id>
📋 Title: <title>
🏷️ Status: <status> | Priority: <priority> | Progress: <progress>%

All supercrew skills will now operate on this feature.
```

8. For the rest of this session, treat this feature as the active feature for `update-status`, `sync-plan`, `log-progress`, and `managing-features` skills.
