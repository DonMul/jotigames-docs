# Quality & Observability

## Quality Expectations

- Contract stability across frontend/backend/ws.
- Role-safe behavior for team/admin boundaries.
- No regression in live-overview realtime behavior.
- i18n completeness for new user-facing strings.

## Validation Strategy

- Validate touched files first (fast diagnostics/lint/tests where available).
- Validate affected integration surface (route + API + WS event consumer).
- Avoid unrelated fixes unless requested.

## Realtime Verification Checklist

- Event emitted with expected name.
- Channel target is correct.
- Payload contains only required delta fields.
- Frontend consumer applies patch idempotently.

## Logging Guidance

- Log domain identifiers (`game_id`, `team_id`, object ids).
- Never log secrets/tokens/api keys.
- Include worker loop heartbeat logs for cron-managed jobs.

## Incident Readiness

- Keep workers restartable without reboot.
- Keep runbooks current in `docs/operations/`.
- Keep WS event references current in `docs/ws/events-reference.md`.
