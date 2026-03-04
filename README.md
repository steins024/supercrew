# SuperCrew

AI-driven feature lifecycle management for Claude Code. Tracks features through a structured lifecycle — from planning through design, implementation, and completion — using `.supercrew/features/` directories.

## Installation

### Claude Code (via Plugin Marketplace)

Register the SuperCrew repo as a marketplace, then install:

```
/plugin marketplace add nicepkg/supercrew
/plugin install supercrew@supercrew
```

### Verify Installation

Start a new session and ask Claude to create a feature or show feature status. SuperCrew skills should activate automatically.

## How It Works

SuperCrew injects context at session start via a hook that:
- Scans `.supercrew/features/` for tracked features
- Auto-loads the active feature based on your current git branch (`feature/<id>`)
- Provides a summary table of all features and their statuses

Skills are invoked automatically when relevant — you don't need to call them manually.

## The Feature Lifecycle

```
planning → designing → ready → active → done
                  ↕               ↕
               planning        blocked
```

Each feature lives in `.supercrew/features/<id>/` with 4 files:
- `meta.yaml` — id, title, status, priority, owner
- `design.md` — background, requirements, design decisions
- `plan.md` — task checklist with progress tracking
- `log.md` — chronological session log

## Skills

| Skill | Triggers on |
|---|---|
| `using-supercrew` | Injected at session start — establishes behavior rules |
| `create-feature` | Creating a new feature |
| `update-status` | Status transitions (e.g. "mark as ready") |
| `sync-plan` | Generating or updating task breakdowns |
| `log-progress` | Recording what was done |
| `managing-features` | Auto-orchestrates lifecycle when `.supercrew/features/` exists |
| `kanban` | Displaying a kanban board of all features |

## Commands

- `/supercrew:new-feature` — Create a new feature with guided prompts
- `/supercrew:feature-status` — Show all features in a table
- `/supercrew:work-on <id>` — Switch the active feature for this session

## Updating

```
/plugin update supercrew
```

## License

MIT License — see [LICENSE](LICENSE) for details.
