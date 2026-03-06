---
status: draft
reviewers: []
# approved_by: ""
---

# Status Schema Change — Technical Design

## Design Decisions

- Simplified status values across all skills for consistency
- Updated templates to reflect new schema structure
- Aligned kanban script with new status definitions

## Architecture

- Skills affected: create-feature, managing-features, sync-plan, update-status
- Templates updated: meta.yaml.tmpl, design.md.tmpl, log.md.tmpl, prd.md.tmpl
- Scripts updated: kanban.sh

## Implementation Notes

- All existing feature meta.yaml files updated to match new schema
- README and docs updated to reflect changes
