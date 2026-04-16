# Playbook Index

This repository serves as the canonical playbook for Hermes sessions and tooling conventions.

## Contents
- 06-tools/SESSION_DISCIPLINE.md (Session discipline & quick setup)
- docs/ (detailed guides and schemas)

## Quick Start
1. Source the environment: ensure `HERMES_HOME` and any API keys are set.
2. Follow the session discipline checklist in 06-tools/SESSION_DISCIPLINE.md.
3. Reference skills from `~/.hermes/skills/` when onboarding to a new system.

## Aliases & Shortcuts (add to shell profile)
- `hresume='hermes -c'` — resume current project
- `htitle='hermes title'` — name a session (use via `hermes title <name>`)
- `hsession-list='hermes sessions list'`
- `hsession-prune='hermes sessions prune'`

## Migration Notes
- Skills are portable: copy `~/.hermes/skills/` when moving to a new system.
- Honcho memory entries are durable and will be injected on new sessions after migration.
- Always prefer `hermes backup --quick` before major changes.