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
/plugin marketplace add /absolute/path/to/supercrew
/plugin install supercrew@supercrew
```

### Verify Installation

Start a new session. Claude will proactively announce which supercrew skill it is invoking before taking any action related to features. No manual skill invocation needed.

## How It Works

SuperCrew injects context at session start via a `SessionStart` hook that:
- Embeds skill routing guidance directly into session context
- Scans `.supercrew/features/` for tracked features
- Auto-loads the active feature based on your current git branch
- Provides a summary table of all features and their statuses

### Branch Naming Convention

SuperCrew uses user-namespaced branches for better team collaboration:

| Stage | Branch Pattern | Example |
|-------|----------------|---------|
| Backlog | `user/<username>/backlog-<feature-id>` | `user/steins-z/backlog-dark-mode` |
| Active work | `user/<username>/<feature-id>` | `user/steins-z/dark-mode` |

Username is derived from `git config user.name`, converted to lowercase with hyphens (e.g., "Steins Z" → `steins-z`).

### Proactive Skill Triggering

Skills activate using the **1% rule**: if there is even a 1% chance a skill applies, Claude invokes it before responding — including before clarifying questions. When a skill fires, Claude announces it:

> *"Using supercrew:create-task to set up the new feature directory..."*

This means you never need to call skills manually. Just describe what you want.

## The Feature Lifecycle

```
todo → doing → ready-to-ship → shipped
  ↑      │
  └──────┘
```

Each feature lives in `.supercrew/features/<id>/` with files:

| File | Contents | When Created |
|------|----------|--------------|
| `meta.yaml` | id, title, status, priority, owner | Feature creation (`todo`) |
| `prd.md` | background, requirements, out of scope | Feature creation (`todo`) |
| `dev-design.md` | design decisions, architecture, implementation notes | Work starts (`doing`) |
| `dev-plan.md` | task checklist with progress tracking | Work starts (`doing`) |
| `dev-log.md` | chronological development log | Work starts (`doing`) |

## Skills

| Skill | When it activates |
|-------|-------------------|
| `create-task` | User wants to create a new feature — creates backlog branch, `meta.yaml`, and `prd.md` with real content |
| `do-task` | Status transitions — starting work, marking done, shipping. Creates `dev-*` files on `todo → doing` |
| `sync-supercrew` | Syncing design iterations, task updates, and progress logging during development |

## Commands

| Command | Description |
|---------|-------------|
| `/supercrew:create` | Create a new feature in the backlog |
| `/supercrew:start <id>` | Start working on a feature (creates work branch and dev-* files) |
| `/supercrew:sync` | Sync all .supercrew updates (design, plan, log) |
| `/supercrew:status` | Show all features in a status table |

## Example Usage

Just talk naturally — SuperCrew skills activate automatically based on context.

### Creating a Feature
```
"I want to add dark mode support"
"Let's create a new feature for user authentication"
"Start tracking a caching layer feature"
```

### Starting Work on a Feature
```
"/supercrew:start dark-mode"
"Start working on the auth feature"
"Begin implementation of dark-mode"
```

### During Implementation
```
"I finished the theme toggle component"
"Update the plan - task 3 is done"
"Sync my progress"
"/supercrew:sync"
```

### Status Updates
```
"This feature is ready to ship"
"Ship it!"
"We need to go back to requirements"
```

### Checking Status
```
"/supercrew:status"
"Show me all features"
"What's the status of our features?"
```

## File Structure

```
.supercrew/
└── features/
    └── <feature-id>/
        ├── meta.yaml        # Feature metadata (always present)
        ├── prd.md           # Product requirements (always present)
        ├── dev-design.md    # Technical design (created when work starts)
        ├── dev-plan.md      # Task breakdown (created when work starts)
        └── dev-log.md       # Development log (created when work starts)
```

## Updating

```
/plugin update supercrew
```

## License

MIT License — see [LICENSE](LICENSE) for details.
