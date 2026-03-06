---
description: "Start working on a feature. Transitions from todo to doing, creates work branch and dev-* files."
disable-model-invocation: true
---

The user wants to start working on a feature.

1. If an argument was provided, use it as the feature ID.
2. If no argument, ask which feature to start.
3. Invoke the supercrew:do-task skill to transition the feature from `todo` to `doing`.

ARGUMENTS: $ARGUMENTS
