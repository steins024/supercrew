# SuperCrew

AI-driven feature lifecycle management for Claude Code. Tracks features through a structured lifecycle — from requirements through design, implementation, and shipping — using `.supercrew/features/` directories.

## Installation

Register the SuperCrew marketplace and install the plugin:

```
/plugin marketplace add steins-z/supercrew
/plugin install supercrew@supercrew
```

### Development/Testing Installation

To install from a local directory (for development or testing):

```
/plugin install /path/to/supercrew
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
todo → doing → ready-to-ship → shipped
  ↑      │
  └──────┘
```

Each feature lives in `.supercrew/features/<id>/` with files:

| File | Contents | When Created |
|---|---|---|
| `meta.yaml` | id, title, status, priority, owner | Feature creation (`todo`) |
| `prd.md` | background, requirements, out of scope | Feature creation (`todo`) |
| `design.md` | design decisions, architecture, implementation notes | Work starts (`doing`) |
| `plan.md` | task checklist with progress tracking | Feature creation (`todo`) |
| `log.md` | chronological session log | Feature creation (`todo`) |

## Skills

| Skill | When it activates |
|---|---|
| `using-supercrew` | Injected at session start — establishes proactive behavior rules |
| `create-feature` | User wants to create or start a new feature |
| `update-status` | Status transitions (e.g. "start working", "ship it") |
| `sync-plan` | Generating or updating task breakdowns from design |
| `log-progress` | End of session, checkpoint, or completed work |
| `managing-features` | General lifecycle orchestration when `.supercrew/features/` exists |
| `kanban` | Displaying a kanban board of all features |

## Commands

| Command | Description |
|---|---|
| `/supercrew:new-feature` | Create a new feature with guided prompts |
| `/supercrew:feature-status` | Show all features in a status table |
| `/supercrew:work-on <id>` | Switch the active feature for this session |

## Example Usage

Just talk naturally — SuperCrew skills activate automatically based on context.

### Creating a Feature
```
"I want to add dark mode support"
"Let's create a new feature for user authentication"
"Start tracking a caching layer feature"
```

### Working on a Feature
```
"/supercrew:work-on dark-mode"
"Switch to the auth feature"
"Show me the kanban board"
```

### During Implementation
```
"I finished the theme toggle component"
"Mark task 3 as done"
"What's left to do on this feature?"
```

### Status Updates
```
"This feature is ready to ship"
"Ship it!"
"We need to go back to requirements"
```

### End of Session
```
"I'm done for today"
"Log my progress"
"Wrap up this session"
```

## Updating

```
/plugin update supercrew
```

## License

MIT License — see [LICENSE](LICENSE) for details.
