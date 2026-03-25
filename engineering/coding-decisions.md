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

## Data & Persistence Decisions

- SQLAlchemy/Alembic are standard for backend schema changes.
- No ad-hoc insecure dynamic SQL patterns.
- Schema and migration changes must ship together.

## Frontend Decisions

- Leaflet is the default geolocation map framework.
- Team markers on maps should use team logo markers where team identity is represented.
- Geolocation pickers should center on browser location if no marker is set.

## Documentation Change Rule

When behavior/flow/contracts change:
1. update docs in `docs/`
2. update event catalog when realtime contracts change
3. keep agent file concise and pointer-based
