# Security, Auth & Authorization

## Threat Model

Assume hostile clients, especially team clients attempting to cheat through:
- forged location updates
- invalid action payloads
- replayed requests
- unauthorized cross-team access

## Backend Enforcement Rules

- Every protected endpoint validates principal type and game access.
- Team principals may only act for their own team when endpoint is team-scoped.
- Owner/admin/game-master scopes are resolved per game membership.
- Super Admin APIs require `ROLE_SUPER_ADMIN` and must reject non-super-admin principals.
- Ambiguous auth context must fail closed.

## Super Admin Panel Security

- The standalone Super Admin app (`admin/`) is a privileged control plane.
- Login is accepted only when `/api/auth/user` returns `ROLE_SUPER_ADMIN`.
- Protected views must re-validate token/session against backend-protected endpoints.
- On `401` or `403`, the client must clear local session and force re-authentication.
- Client role checks are convenience guards only; backend authorization is authoritative.

## WS Enforcement Rules

- `core.subscribe` requires backend access verification before joining channel.
- `core.publish` requires valid backend API key.
- WS does not infer business permissions.

## Input Validation Rules

- Validate IDs, enums, coordinates, and quantity/amount bounds.
- Reject out-of-range geolocation coordinates.
- Sanitize and constrain textual fields.
- Use typed request models in backend modules.

## Data Integrity Rules

- Persist first, then publish realtime event.
- Keep server-calculated score/cash/inventory authoritative.
- Never trust client-submitted score-like fields.

## Secrets & Logging

- Do not expose API keys/tokens in responses or logs.
- Do not log credentials or bearer tokens.
- Log domain identifiers (game/team/resource IDs) for diagnostics.

## Geolocation Safety Rules

- Enforce per-game timing constraints (for relevant games) in backend logic.
- Do not rely on frontend timing enforcement for safety-critical behaviors.
