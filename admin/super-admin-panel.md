# Super Admin Panel

This document describes the standalone Super Admin app in `admin/`.

The Super Admin panel is a platform control plane for global configuration and account governance. It is intentionally separate from game runtime admin pages under `/admin/*` in the main frontend app.

## Scope & Ownership

The panel owns:
- super-admin authentication UI
- global game type availability management
- platform user lifecycle management (create, update, delete)
- full game management and live overview (all games, regardless of ownership)

The panel does not own:
- game rules, scoring, or role decisions
- websocket event semantics

Backend remains the source of truth for authorization and data mutations.

## App Topology

Source folder:
- `admin/src/`

Primary files:
- `admin/src/App.jsx` – route shell, session verification, top-level layout
- `admin/src/components/DashboardLayout.jsx` – sidebar + header chrome, child route mounting
- `admin/src/components/Sidebar.jsx` – dark sidebar with nav icons and user info
- `admin/src/components/UserForm.jsx` – reusable create/edit user form
- `admin/src/pages/LoginPage.jsx` – login page
- `admin/src/pages/OverviewPage.jsx` – dashboard overview with stat cards
- `admin/src/pages/GamesPage.jsx` – all-games table with inline detail, teams, overview, and game management
- `admin/src/pages/GameModesPage.jsx` – game type availability management
- `admin/src/pages/UsersPage.jsx` – user list with table, filters, pagination
- `admin/src/pages/UserCreatePage.jsx` – create user form
- `admin/src/pages/UserEditPage.jsx` – edit/delete user form
- `admin/src/components/ui/` – shadcn-style UI primitives (Button, Card, Input, Badge, Table, Select, Label, Separator, Skeleton)
- `admin/src/lib/api.js` – API request helper + endpoint wrappers
- `admin/src/lib/auth.js` – local session persistence utilities
- `admin/src/lib/utils.js` – `cn()` class merge utility, `formatDate`, `normalizeText`, `toCsvValue`, `roleDisplayName`, `gameTypeDisplayName`
- `admin/src/lib/theme.js` – dark mode detection, persistence, and toggle utility

### UI Stack

- **Tailwind CSS v3** – utility-first styling with PostCSS
- **shadcn-style components** – custom UI primitives following shadcn/ui patterns (class-variance-authority, clsx, tailwind-merge)
- **Lucide React** – icon library
- **@radix-ui/react-slot** – polymorphic Button component
- **Design**: Light/dark theme with system preference detection, dark sidebar, minimalistic control-panel aesthetic
- **Dark Mode**: Uses Tailwind `darkMode: 'class'` strategy. System preference (`prefers-color-scheme`) is used as default. Manual toggle persists in localStorage under `jotigames-admin-theme`. Flash prevention via inline `<script>` in `index.html`.

### Internal Routes

The app route tree is:
- `/login`
- `/dashboard`
- `/dashboard/games`
- `/dashboard/game-modes`
- `/dashboard/users`
- `/dashboard/users/new`
- `/dashboard/users/:userId/edit`

Unknown paths redirect to `/dashboard` when authenticated or `/login` when unauthenticated.

## Authentication & Session Model

Login flow:
1. User submits email/password at `/login`.
2. App calls `POST /api/auth/user`.
3. App requires the response to include `ROLE_SUPER_ADMIN` or `ROLE_ADMIN`.
4. Session payload (including `username`) is stored in browser local storage.
5. App navigates to dashboard.

Stored session key:
- `jotigames_super_admin_session`

Theme preference key:
- `jotigames-admin-theme` (values: `light`, `dark`, or absent for system default)

Session verification on app start:
- If token + admin/super-admin role are present, app calls `GET /api/super-admin/users` as a verification probe.
- On `401`/`403`, local session is cleared and user is redirected to `/login`.

### Role Hierarchy

Three backend roles exist:
- `ROLE_USER` – base role, always assigned
- `ROLE_ADMIN` – platform admin with access to admin panel and most management functions
- `ROLE_SUPER_ADMIN` – full platform admin; can additionally manage user roles

Frontend displays human-readable names:
- `ROLE_USER` → User
- `ROLE_ADMIN` → Admin
- `ROLE_SUPER_ADMIN` → Super Admin

Only users with `ROLE_SUPER_ADMIN` can see and edit the Roles section in the user form. The Verification section is visible to all admin-panel users.

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
- `GET /api/super-admin/users/:userId/games` – list games owned by user

Game management (reuses existing game endpoints with admin bypass):
- `GET /api/game` – list all games (admin bypass returns all, not just owned)
- `GET /api/game/:gameId` – get game detail
- `GET /api/game/:gameId/teams` – list teams
- `GET /api/game/:gameId/members` – list members
- `POST /api/game/:gameId/reset` – reset game
- `DELETE /api/game/:gameId` – delete game
- `DELETE /api/game/:gameId/teams/:teamId` – delete team
- `GET /api/<module-prefix>/:gameId/overview` – game-type-specific live overview

All protected calls send `Authorization: Bearer <token>`.

## Feature Behavior

### Overview (`/dashboard`)

Displays summary metrics derived from:
- game type availability list
- user list

### Games (`/dashboard/games`)

Displays all games on the platform (not filtered by ownership) with:
- search/filter by name, ID, or game type
- sortable by name, type, start, end
- pagination controls
- status badges (Active, Upcoming, Ended)
- inline expandable detail row per game with:
  - game metadata (ID, code, type, date range)
  - live overview summary (entity counts per game type)
  - game members with roles
  - teams table with scores, lives, and delete action
  - game reset action

Platform admins bypass per-game ownership checks via `principal.is_admin` in backend.

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
- inline expandable detail row per user (shown directly below the clicked row)
- games-owned table within each user detail row
- links to create/edit user pages

### User Create/Edit

Create:
- `/dashboard/users/new`
- requires email, username, password, roles, verified flag

Edit:
- `/dashboard/users/:userId/edit`
- allows metadata updates and optional password rotation
- supports deletion with explicit confirmation
- roles section only visible to users with `ROLE_SUPER_ADMIN`
- verification section always visible

## Security Rules

- Admin or Super Admin role is required at login and route access time.
- Role management (assigning roles to users) is restricted to `ROLE_SUPER_ADMIN` only.
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
