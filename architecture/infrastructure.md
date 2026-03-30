# Infrastructure & Deployment

## High-Level Runtime

Production topology:

- Nginx reverse proxy (TLS termination)
- Frontend app (user/admin web)
- Backend FastAPI service
- WS Node service
- Database (shared by backend)
- Cron-managed background workers

## Reverse Proxy Paths

- `/api/*` -> backend service
- `/ws/*` -> websocket service
- site roots -> frontend apps

## Reverse Proxy Hosts

- `jotigames.nl` + `www.jotigames.nl` -> frontend app
- `admin.jotigames.nl` -> admin app
- `api.jotigames.nl` -> backend FastAPI app
- `https://api.jotigames.nl/docs` and `https://api.jotigames.nl/openapi.json` are IP-restricted to `149.143.99.118` at nginx layer

See `system/nginx/` for concrete configs.

### Nginx Security

- **HSTS**: enabled (`max-age=63072000; includeSubDomains; preload`)
- **Security headers**: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `X-XSS-Protection`, `Permissions-Policy`
- **Rate limiting**: `auth_api` zone (10 req/s, burst 20) on `/api/auth/`; `general_api` zone (30 req/s, burst 60) on `/api/`
- **Body size**: `client_max_body_size 2m`

## Deployment Script

Main script: `system/deploy_update.sh`

Responsibilities:
- pull/update repositories
- install/update dependencies
- build frontend(s)
- run backend migrations
- install managed cron block
- restart workers and services
- update nginx configs and certificate mode

## Bootstrap Script

One-time server bootstrap: `system/bootstrap_server.sh`

Responsibilities:
- install base OS packages
- clone system repo
- run deployment flow

## Cron/Worker Model

Managed in:
- `system/cron/crontab.template`
- `system/cron/restart_workers.sh`

Current long-running worker examples:
- `backend/scripts/auto_resolve_pending_actions.py`
- `backend/scripts/birds_of_prey_auto_drop_eggs.py`
- `backend/scripts/courier_rush_spawn_points.py`
- `backend/scripts/market_crash_fluctuate_prices.py`
- `backend/scripts/pandemic_response_spawn_hotspots.py`
- `backend/scripts/territory_control_tick_scores.py`

Rule: any new persistent worker must be wired in both files in same change set.

## Environment & Secrets

Critical secret paths:
- `WS_TO_BACKEND_API_KEY` (WS -> backend verify)
- `BACKEND_TO_WS_API_KEY` (backend -> WS publish)

Operational rules:
- never expose secrets in logs/responses
- validate env presence at startup for critical services

## Observability Basics

- worker logs in `system/logs/`
- deploy logs via shell output and service restarts
- websocket and backend logs should be correlated with event names and game/team IDs (never credentials)
