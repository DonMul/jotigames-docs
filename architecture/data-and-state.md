# Data & State Ownership

## Persistence Layers

Primary persistence is backend-managed SQL storage via SQLAlchemy.

- Schema evolution: Alembic migrations
- Domain repositories: backend modules/repositories
- Runtime projection: frontend in-memory state patched from API bootstrap + WS deltas

## Authoritative State Rules

- Backend DB is authoritative for score/resources/inventory/captures/actions.
- Frontend state is a projection for UX responsiveness.
- WS events are transport deltas, not durable state.

## State Transition Pattern

1. API receives action intent.
2. Backend validates auth + rules.
3. Backend writes DB state.
4. Backend emits relevant WS deltas.
5. Frontend applies patch and updates display.

## Migration/Schema Rules

- Ship schema + repository/service changes together.
- Avoid hidden schema coupling in frontend.
- Keep IDs and types stable across frontend/backend/ws contracts.

## i18n Data Rules

- Backend returns stable translation keys for domain errors/messages.
- Frontend resolves translated user text in `en` and `nl` locale bundles.
