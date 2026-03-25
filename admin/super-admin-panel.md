# Super Admin Panel

This document describes the standalone Super Admin app in `admin/`.

The Super Admin panel is a platform control plane for global configuration and account governance. It is intentionally separate from game runtime admin pages under `/admin/*` in the main frontend app.

## Scope & Ownership

The panel owns:
- super-admin authentication UI
- global game type availability management
- platform user lifecycle management (create, update, delete)

The panel does not own:
- per-game runtime administration (teams, points, rounds, live map)
- game rules, scoring, or role decisions
- websocket event semantics

Backend remains the source of truth for authorization and data mutations.

## App Topology

Source folder:
- `admin/src/`

Primary files:
- `admin/src/App.jsx` route shell, feature pages, session verification
- `admin/src/lib/api.js` API request helper + endpoint wrappers
- `admin/src/lib/auth.js` local session persistence utilities

### Internal Routes

The app route tree is:
- `/login`
- `/dashboard`
- `/dashboard/game-modes`
- `/dashboard/users`
- `/dashboard/users/new`
- `/dashboard/users/:userId/edit`

Unknown paths redirect to `/dashboard` when authenticated or `/login` when unauthenticated.

## Authentication & Session Model

Login flow:
1. User submits email/password at `/login`.
2. App calls `POST /api/auth/user`.
3. App requires the response to include `ROLE_SUPER_ADMIN`.
4. Session payload is stored in browser local storage.
5. App navigates to dashboard.

Stored session key:
- `jotigames_super_admin_session`

Session verification on app start:
- If token + `ROLE_SUPER_ADMIN` are present, app calls `GET /api/super-admin/users` as a verification probe.
- On `401`/`403`, local session is cleared and user is redirected to `/login`.

## API Contract (Current)

Auth:
- `POST /api/auth/user`

Game type availability:
- `GET /api/game/game-types/availability`
- `PUT /api/game/game-types/availability`
  - body: `{ "enabled_game_types": string[] }`

Super-admin users:
- `GET /api/super-admin/users`
- `GET /api/super-admin/users/:userId`
- `POST /api/super-admin/users`
- `PUT /api/super-admin/users/:userId`
- `DELETE /api/super-admin/users/:userId`

All protected calls send `Authorization: Bearer <token>`.

## Feature Behavior

### Overview (`/dashboard`)

Displays summary metrics derived from:
- game type availability list
- user list

Includes quick links to Game Modes and Users pages.

### Game Modes (`/dashboard/game-modes`)

Allows global enable/disable of game types with:
- search/filter/sort
- bulk enable/disable of currently visible rows
- explicit save/discard workflow

Changes are applied through a single backend update request using enabled type names.

### Users (`/dashboard/users`)

Provides:
- searchable/sortable/filterable users table
- paging controls
- CSV export of currently filtered list
- detail drawer for selected user
- links to create/edit user pages

### User Create/Edit

Create:
- `/dashboard/users/new`
- requires email, username, password, roles, verified flag

Edit:
- `/dashboard/users/:userId/edit`
- allows metadata updates and optional password rotation
- supports deletion with explicit confirmation

## Security Rules

- Super Admin role is required at login and route access time.
- Client-side role checks are UX guards only; backend authorization remains authoritative.
- Any unauthorized backend response must immediately invalidate local session.
- Do not expose this app’s controls in team or game-admin runtime routes.

## Operational Notes

Run locally:
```bash
cd admin
npm install
npm run dev
```

Environment:
- `VITE_BACKEND_TARGET` for Vite proxy backend target
- `VITE_API_BASE_URL` for direct API base URL

For setup details, see `admin/README.md`.
