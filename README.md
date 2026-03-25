# JotiGames Documentation

This folder is the central documentation source for JotiGames.

## Documentation Map

- [Architecture Overview](architecture/overview.md)
- [Infrastructure & Deployment](architecture/infrastructure.md)
- [Realtime & WS Contract](architecture/realtime.md)
- [Security, Auth & Authorization](architecture/security-auth.md)
- [Data & State Ownership](architecture/data-and-state.md)
- [Coding & Logical Decisions](engineering/coding-decisions.md)
- [Development Workflow](engineering/development-workflow.md)
- [Quality & Observability](engineering/quality-and-observability.md)
- [Page & Route Map](pages/page-map.md)
- [Games Handbook (split by game type + pages)](games/index.md)
- [Super Admin Panel](admin/super-admin-panel.md)
- [Operations: workers, cron, runbooks](operations/workers-and-runbooks.md)
- [WS Events Reference (canonical contributor view)](ws/events-reference.md)

## Scope Rules

1. Every code change must include docs maintenance: update relevant docs here in the same change set, or explicitly declare "no-doc-impact".
2. WS command/event changes must update [ws/events-reference.md](ws/events-reference.md).
3. New game types must update [games/index.md](games/index.md) and [pages/page-map.md](pages/page-map.md).
4. Existing module-specific READMEs may remain for local setup, but cross-system logic belongs in this folder.

## Related Local READMEs

- `backend/README.md` for backend local setup
- `frontend/README.md` for frontend local setup
- `ws/README.md` for websocket server local setup
- `system/README.md` for production/system scripts
