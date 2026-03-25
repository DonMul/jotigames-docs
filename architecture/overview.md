# Architecture Overview

JotiGames is a phygital game platform with three synchronized systems:

- `frontend/`: React UI for admin, team, and live overviews.
- `backend/`: FastAPI API and central game/business logic owner.
- `ws/`: transport-only websocket pub/sub service.

## Ownership Boundaries

## Frontend (`frontend/`)

Owns:
- page rendering and UX interaction
- API integration via `src/lib/api.js`
- websocket subscriptions and UI patching
- locale rendering (`src/i18n/locales/*.json`)

Does not own:
- game rule decisions
- authorization
- authoritative score/resource/capture outcomes

## Backend (`backend/`)

Owns:
- all game state transitions and validations
- authorization checks for every protected endpoint
- persistence and migrations
- outbound realtime event semantics and payloads

Does not own:
- websocket transport fanout mechanics

## WS (`ws/`)

Owns:
- channel subscriptions
- publish fanout to channels
- publish API key gate (`BACKEND_TO_WS_API_KEY`)

Does not own:
- game logic or rule evaluation
- score validation
- authorization decisions beyond subscribe gating and publish key checks

## Channel Model

- `channel:{game_id}`: game-wide events
- `channel:{game_id}:{team_id}`: team-private events
- `channel:{game_id}:admin`: admin events

## Runtime Data Flow

1. Frontend performs one-time bootstrap via backend endpoint.
2. Frontend subscribes to one or more WS channels.
3. User action goes to backend API.
4. Backend commits state.
5. Backend publishes minimal WS delta events.
6. Frontend patches local runtime state from WS deltas.

## Design Principles

- Keep contracts stable and explicit.
- Keep realtime payloads as small as possible.
- Keep role-separated routing and actions (`/admin/*` vs `/team/*`).
- Keep user-facing text localized.
- Fail closed on auth ambiguity.
