# Coding & Logical Decisions

This document captures cross-system implementation decisions that should remain stable unless intentionally changed.

## Core Decisions

## 1) Backend is the only game-logic owner

Why:
- anti-cheat integrity
- consistent scoring and validations

Implication:
- frontend submits intent
- backend computes outcomes

## 2) WS is transport-only

Why:
- single source of truth for game logic
- reduced duplication

Implication:
- no rule evaluation inside WS server

## 3) Role-separated routing and UX

Why:
- prevent privilege confusion
- reduce accidental admin exposure

Implication:
- admin-only functions under `/admin/*`
- team self-service under `/team/*`

## 4) Bootstrap once + patch by WS

Why:
- responsive live UX without polling storms
- clearer state synchronization model

Implication:
- live pages should not rely on repeated fetch loops for runtime deltas

## 5) Translation keys as API-domain responses

Why:
- stable localized error handling
- consistent frontend messaging

Implication:
- backend returns translation-key style domain errors
- frontend resolves localized display text

## 6) Minimal realtime payloads

Why:
- lower WS bandwidth
- less client merge complexity

Implication:
- emit patch-oriented events (changed entities/fields only)

## 7) Worker wiring discipline

Why:
- avoid orphan long-running jobs
- ensure deploy consistency

Implication:
- every new worker must be added to both cron template and restart script

## 8) Backend functions must be explicitly documented

Why:
- backend owns authorization outcomes and game-state mutations
- onboarding/debug speed depends on readable intent at function level
- route/service behavior must be explainable without reverse-engineering implementation details

Implication:
- backend functions should include docstrings that explain purpose, side effects, and failure behavior
- auth/security functions should document trust boundaries and validation rules
- route handlers should document authorization expectations and response semantics

## 9) Python 3.14 coroutine-introspection compatibility

Why:
- Python 3.14 deprecates `asyncio.iscoroutinefunction`, while current FastAPI internals still reference it.
- This emitted runtime deprecation warnings across route-building tests and polluted CI signal.

Implication:
- Backend startup applies a compatibility bridge in `app/main.py` by routing FastAPI's coroutine check to `inspect.iscoroutinefunction`.
- This is a runtime behavior fix, not warning suppression, and keeps tests warning-free on Python 3.14+.

## 10) Frontend locale selection resolves per-locale files first

Why:
- users can explicitly select many supported locales from the language selector
- forcing non-NL locales to English breaks expected UX and translation coverage

Implication:
- frontend i18n must resolve `frontend/src/i18n/locales/<locale>.json` first, then language-only fallback, then `nl`, then `en`
- locale resolution should normalize both hyphen and underscore formats (for example `en-GB` → `en_GB` → `en`)
- when using `import.meta.glob`, loader lookups must use full glob keys (for example `../i18n/locales/en.json`), not shorthand locale codes

## Data & Persistence Decisions

- SQLAlchemy/Alembic are standard for backend schema changes.
- No ad-hoc insecure dynamic SQL patterns.
- Schema and migration changes must ship together.
- Repositories that reflect legacy tables at runtime must resolve key columns via candidate aliases (for example `id`/`game_id` and `id`/`team_id`) instead of hard-coding one column name.
- Courier Rush spawn-area config persistence must resolve both `...geojson` and legacy `...geo_json` column aliases on the reflected `game` table when reading and writing `pickup_spawn_area_geojson`.
- Pandemic Response spawn-area config persistence must resolve both `...geojson` and legacy `...geo_json` column aliases on the reflected `game` table when reading and writing `spawn_area_geojson`.
- Long-running workers that need per-entity tick timestamps (for example Pandemic hotspot escalation and Territory hold ticks) must persist worker progress in game `settings` under `_backend_workers` to stay compatible with reflected schemas that may miss dedicated timing columns.
- GeoHunter POI `expected_answers` persistence must be schema-safe for reflected tables where `geo_point.expected_answers` is `LONGTEXT`; store as JSON string and decode on read so API contracts remain `list[str]`.
- GeoHunter POI scoring uses a required `geo_point.points` column. Environments must be migrated (Alembic revision `20260330_0007`) instead of silently dropping unknown write fields at repository level.
- GeoHunter retry lockouts must be derived from `geo_submission.submitted_at` per `(team_id, point_id)` (lock checked before answer validation), and location/bootstrap payloads must expose server-derived lock countdown maps so frontend timers stay authoritative.
- GeoHunter answer submission must be radius-gated server-side (`team` must be in POI range), and map visibility mode (`all_visible` or `in_range_only`) is stored schema-first in `game.geo_hunter_visibility_mode` (Alembic revision `20260330_0008`) with legacy settings fallback, then propagated through team bootstrap + nearby-update events.
- Game deletion must explicitly clear GeoHunter `geo_submission` rows via `geo_point` parent linkage before deleting game rows, because reflected legacy schemas may not expose complete FK metadata for automatic cascade traversal.

## Frontend Decisions

- Leaflet is the default geolocation map framework.
- Team markers on maps should use team logo markers where team identity is represented.
- Geolocation pickers should center on browser location if no marker is set.
- Team dashboard maps default to zoom level `18`.
- Admin maps (including configure/points maps) default to zoom level `15`.
- Team and admin maps should center on the current browser geolocation and keep following position updates when location access is available.
- Frontend realtime transport (`window.JotiWs`) must be available before React route chunks render; keep `/assets/scripts/ws_transport.js` in `frontend/public` and retain bootstrap fallback loading in `frontend/src/main.jsx`.
- Public top navigation includes a dark-mode toggle and persists the selected theme in `localStorage` key `jotigames-theme`.
- Public pages must provide full-surface dark mode coverage (section backgrounds, cards, text, borders, inputs, and dropdowns), not only isolated components.
- Shared public components (for example carousels) and all game-info variants (detailed, bespoke, and fallback tabbed layouts) must include equivalent dark-mode states.
- Authenticated user surfaces (`/account/*`, `/admin/games*`, and related create/edit forms) must ship with equivalent dark-mode card, form, and typography states.
- Shared legacy admin utility classes in `frontend/src/styles.css` (for example `page-shell`, `admin-block`, `overview-panel`, `admin-table`, `form-row`, and map/chat helper classes) are part of the dark-mode contract and must include `dark:` variants so game-admin module pages inherit theme parity without per-page duplication.
- Public game-type carousel cards are ordered alphabetically by translated game name.
- Public carousel edge controls should avoid clipping the first card: left control uses full outside offset (`-translate-x-full`) while right control keeps edge overlap.
- Login submit copy must preserve the E2E selector contract (`Sign in` in English, `Inloggen` in Dutch).

## Documentation Change Rule

When behavior/flow/contracts change:
1. update docs in `docs/`
2. update event catalog when realtime contracts change
3. keep agent file concise and pointer-based
