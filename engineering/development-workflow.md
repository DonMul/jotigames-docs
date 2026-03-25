# Development Workflow

## Typical Change Sequence

1. Identify layer ownership (frontend/backend/ws/ops).
2. Implement backend domain logic and endpoint updates.
3. Update frontend API integration and UI.
4. Add/update WS events only if shared realtime state changes.
5. Wire workers/cron when long-running automation is introduced.
6. Add/refresh i18n keys for all user-facing text.
7. Update docs in `docs/` in same change set (mandatory).

### Documentation Is Mandatory

- Every implementation change must include a docs impact check.
- If there is impact, update the relevant `docs/` files in the same PR/commit.
- If there is no impact, explicitly state "no-doc-impact" in the handoff/summary.
- Do not close implementation tasks while docs are stale.
- For backend changes, keep function/class docstrings current in touched Python files.

## API/WS Contract Safety

Before introducing new event names:
- check [../ws/events-reference.md](../ws/events-reference.md)
- reuse existing events where possible
- if new event needed, document payload and channel usage immediately

## Testing Scope Guidance

- Start with focused checks on touched files/routes.
- Then run broader checks if available.
- Do not fix unrelated failures in same task unless explicitly requested.

### Current test matrix

- Backend unit tests: `backend/tests` via `pytest`
- Frontend unit tests: `frontend/src/**/*.test.js` via `vitest`
- Admin unit tests: `admin/src/**/*.test.js` via `vitest`
- WS unit tests: `ws/tests` via Node test runner
- Cross-app E2E tests: `e2e/tests` via Playwright (targets frontend/admin/backend)

### Backend coverage policy

- All backend modules in `backend/app/modules/` must be covered by tests.
- All backend services in `backend/app/services/` must be covered by tests.
- All backend repositories in `backend/app/repositories/` must be covered by tests.
- Implemented business logic must always be verified by tests before task completion.
- Endpoint contract/smoke tests must assert no unexpected 5xx for documented API routes.

Implemented guard suites:

- `backend/tests/test_birds_of_prey_module.py` verifies birds-of-prey module helper payload and WS publish fan-out behavior.
- Per-game duplicated suites verify service/repository/module coverage for:
	- `blindhike`, `checkpoint_heist`, `code_conspiracy`, `courier_rush`
	- `crazy88`, `echo_hunt`, `exploding_kittens`, `geohunter`
	- `market_crash`, `pandemic_response`, `resource_run`, `territory_control`

### Run commands

From repository root:

- Full matrix: `scripts/run_all_tests.sh`

Manual per codebase:

- Backend: `cd backend && python3 -m pytest -q`
- Frontend: `cd frontend && npm test`
- Admin: `cd admin && npm test`
- WS: `cd ws && npm test`
- E2E: `cd e2e && npm test`
- Docs screenshots refresh: `cd e2e && npm run docs:screenshots`

### Locale translation automation

Scripts in repository root `scripts/` support value-only translation while preserving locale key contracts.

- Generate a target locale from Dutch source files:
	- `python3 scripts/translate_from_nl.py --lang <locale>`
	- optional performance tuning: `--batch-size 64 --workers 4` (increase cautiously)
- Clean-regenerate all locale files from Dutch source (value-only, keys unchanged):
	- `python3 scripts/sync_locales_from_nl.py --batch-size 64 --workers 4`
	- optional source cleanup first: `--normalize-source` (forces source locale values back to Dutch before fan-out)

Source files:

- Frontend Dutch source: `frontend/src/i18n/locales/nl.json`
- Backend Dutch source: `backend/translations/locales/nl.yaml`

Notes:

- Scripts translate values only; keys and structure remain unchanged.
- `scripts/sync_locales_from_nl.py` runs in start-clean mode and fully rewrites discovered target locales from current Dutch source files.
- You can include additional locale targets not present on disk with `--locales <comma-separated-codes>`.
- Requires Python package `deep-translator` (`pip install deep-translator`).

### Environment notes

- Ensure Node and npm are available in `PATH` (for nvm users: source nvm before running JS tests).
- Backend dependency compatibility: keep `chardet<6` to avoid `RequestsDependencyWarning` with `requests` in Python 3.14 virtualenvs.
- E2E targets hosted dev environments by default:
	- frontend: `http://localhost:5173`
	- admin: `http://localhost:5174`
	- backend: `http://localhost:8000`

## Code Change Guardrails

- Keep changes surgical.
- Preserve route semantics and role boundaries.
- Avoid introducing parallel patterns for same problem.

## i18n Workflow

- Add keys in `frontend/src/i18n/locales/en.json` and `nl.json` together.
- Add backend translation keys in `backend/translations/locales/en.yaml` and `nl.yaml` together when domain errors/success keys are added.

## Definition of Done

A feature is complete when:
- behavior works across frontend/backend/ws boundaries
- i18n is complete
- docs are updated
- docs impact has been explicitly handled (updated or "no-doc-impact")
- no new diagnostics errors in touched files
