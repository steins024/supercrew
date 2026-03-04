# SuperCrew

AI-driven feature lifecycle management for Claude Code. Tracks features through a structured lifecycle — from planning through design, implementation, and completion — using `.supercrew/features/` directories.

## Installation

Register the SuperCrew marketplace and install the plugin:

```
/plugin marketplace add steins-z/supercrew
/plugin install supercrew@supercrew
```

### Verify Installation

Start a new session. Claude will proactively announce which supercrew skill it is invoking before taking any action related to features. No manual skill invocation needed.

## How It Works

SuperCrew injects context at session start via a `SessionStart` hook that:
- Reads the `using-supercrew` skill and passes it directly as session context
- Scans `.supercrew/features/` for tracked features
- Auto-loads the active feature based on your current git branch (`feature/<id>`)
- Provides a summary table of all features and their statuses

### Proactive Skill Triggering

Skills activate using the **1% rule**: if there is even a 1% chance a skill applies, Claude invokes it before responding — including before clarifying questions. When a skill fires, Claude announces it:

> *"Using supercrew:create-feature to set up the new feature directory..."*

This means you never need to call skills manually. Just describe what you want.

## The Feature Lifecycle

```
planning → designing → ready → active → done
                  ↕               ↕
               planning        blocked
```

Each feature lives in `.supercrew/features/<id>/` with 4 files:

| File | Contents |
|---|---|
| `meta.yaml` | id, title, status, priority, owner |
| `design.md` | background, requirements, design decisions |
| `plan.md` | task checklist with progress tracking |
| `log.md` | chronological session log |

## Skills

| Skill | When it activates |
|---|---|
| `using-supercrew` | Injected at session start — establishes proactive behavior rules |
| `create-feature` | User wants to create or start a new feature |
| `update-status` | Status transitions (e.g. "mark as ready", "block this") |
| `sync-plan` | Generating or updating task breakdowns after design changes |
| `log-progress` | End of session, checkpoint, or completed work |
| `managing-features` | General lifecycle orchestration when `.supercrew/features/` exists |
| `kanban` | Displaying a kanban board of all features |

## Commands

| Command | Description |
|---|---|
| `/supercrew:new-feature` | Create a new feature with guided prompts |
| `/supercrew:feature-status` | Show all features in a status table |
| `/supercrew:work-on <id>` | Switch the active feature for this session |

## Updating

```
/plugin update supercrew
```

## License

MIT License — see [LICENSE](LICENSE) for details.
