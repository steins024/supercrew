---
status: approved
reviewers: []
# approved_by: ""
---

# SuperCrew MVP - AI Integration Plugin — Technical Design

## Design Decisions

- Plugin follows superpowers architecture (skills + hooks + commands)
- Feature-oriented data model in `.supercrew/features/<feature-id>/`
- Session context injection via SessionStart hook
- 1% rule for proactive skill invocation

## Architecture

```
plugins/supercrew/
├── .claude-plugin/marketplace.json
├── skills/
│   ├── create-feature/SKILL.md
│   ├── update-status/SKILL.md
│   ├── sync-plan/SKILL.md
│   ├── log-progress/SKILL.md
│   ├── managing-features/SKILL.md
│   ├── using-supercrew/SKILL.md
│   └── kanban/SKILL.md
├── commands/
│   ├── new-feature.md
│   ├── feature-status.md
│   └── work-on.md
├── hooks/
│   ├── hooks.json
│   ├── session-start
│   └── run-hook.cmd
└── templates/
    ├── meta.yaml.tmpl
    ├── design.md.tmpl
    ├── plan.md.tmpl
    └── log.md.tmpl
```

## Implementation Notes

### Proactive Skill Invocation
The `using-supercrew` skill establishes the **1% rule**: if there's even a 1% chance a skill applies, Claude must invoke it before any response or action. This is enforced via session context injection.

### Active Feature Matching
1. Git branch name `feature/<id>` -> auto-match to `.supercrew/features/<id>/`
2. If no match, prompt user to select or use `/supercrew:work-on`
3. All skill operations target the active feature by default

### Status Transitions
Valid transitions form a linear flow:
- todo -> doing
- doing -> ready-to-ship
- ready-to-ship -> shipped
