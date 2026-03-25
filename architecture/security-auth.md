# Security, Auth & Authorization

## Threat Model

Assume hostile clients, especially team clients attempting to cheat through:
- forged location updates
- invalid action payloads (score manipulation, answer spoofing)
- replayed requests
- unauthorized cross-team access
- reading API response data to reveal answers

## Backend Enforcement Rules

- Every protected endpoint validates principal type and game access.
- Team principals may only act for their own team when endpoint is team-scoped.
- Owner/admin/game-master scopes are resolved per game membership.
- Super Admin APIs require `ROLE_SUPER_ADMIN` and must reject non-super-admin principals.
- Ambiguous auth context must fail closed.

## Anti-Cheat: Server-Side Scoring

- **All points/scores are determined server-side.** Client request models must
  never include `points`, `points_delta`, `correct`, or similar score-affecting fields.
- Services look up the relevant entity (checkpoint, beacon, node, zone, dropoff,
  hotspot, POI) and read the admin-configured point value from the database.
- For Birds of Prey egg destruction, the awarded points are a server-fixed constant (1).
- For GeoHunter, the server validates answers against stored `correct_answer`
  (open questions) or `is_correct` flags on choices (multiple-choice). The team
  bootstrap response **never** includes `correct_answer` or `is_correct`.
- For Code Conspiracy, the server validates submitted codes against the
  `code_conspiracy_team_code` table and awards `correct_points` / applies
  `penalty_value` from game configuration.

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

## HTTP Security Hardening

### CORS
- Backend `CORSMiddleware` restricts origins via `CORS_ALLOWED_ORIGINS` env var
  (comma-separated list). Empty default = same-origin only.
- Allowed methods: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`.
- Allowed headers: `Authorization`, `Content-Type`, `Accept-Language`.

### Security Response Headers (backend middleware)
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `X-XSS-Protection: 1; mode=block`
- `Cache-Control: no-store`
- `Permissions-Policy: geolocation=(self), camera=(), microphone=()`

### Nginx Security Headers
- `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`
- Same set as backend (`X-Content-Type-Options`, `X-Frame-Options`, etc.)
- Rate limiting: `auth_api` zone (10 req/s) on `/api/auth/`, `general_api` zone
  (30 req/s) on `/api/`.
- Body size limit: `client_max_body_size 2m`.

### Token Lifecycle
- Default token TTL: **24 hours** (`TOKEN_TTL_MINUTES=1440`).
- Configurable via `TOKEN_TTL_MINUTES` environment variable.

## Secrets & Logging

- Do not expose API keys/tokens in responses or logs.
- Do not log credentials or bearer tokens.
- Log domain identifiers (game/team/resource IDs) for diagnostics.

## Geolocation Safety Rules

- Enforce per-game timing constraints (for relevant games) in backend logic.
- Do not rely on frontend timing enforcement for safety-critical behaviors.
