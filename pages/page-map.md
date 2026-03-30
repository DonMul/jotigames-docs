# Page & Route Map

This document explains role-based page topology and intended responsibilities.

For detailed game-page descriptions and screenshots, see [../games/index.md](../games/index.md).

## Public

- `/` Home
- `/about`
- `/pricing`
- `/faq`
- `/register`
- `/login`
- `/team-login`
- `/info/games/:slug`

Notes:
- When monetisation is disabled (`ENABLE_MONETISATION=false`), pricing links are hidden from navigation and `/pricing` redirects to `/`.
- Game type links in the public navigation are displayed alphabetically by their localized display name.
- `/info/games/:slug` hero renders a larger game logo directly (without the previous boxed logo container).
- `/info/games/birds-of-prey` uses a bespoke Birds of Prey landing/how-to-play layout grounded in the live gameplay rules: egg visibility radius, protection radius, auto-drop pacing, and live leaderboard pressure.
- `/info/games/:slug` now provides deep game-specific landing/how-to-play content for every supported game type. Birds of Prey keeps its dedicated bespoke implementation, while the remaining game types render through a shared detailed page shell with locale-driven copy assembled in `GameInfoPage.jsx` from Dutch translation keys grounded in live backend rules.

## Team Routes (`/team/*`)

- `/team`
  - team runtime dashboard, game-specific panel rendering
- `/team/edit`
  - team self-update (name + logo only)
- `/team/scan/:qrToken`
  - scan/resolve flow pages where relevant
- `/team/enter`
  - team entry helpers
- `/team/games/:gameId/play`
  - team play module surface where used

## Admin Routes (`/admin/*`)

### Core Game Management

- `/admin/games`
  - primary actions: create game (subscription action moved to account settings)
- `/admin/games/new`
- `/admin/games/:gameId`
- `/admin/games/:gameId/edit`
- `/admin/games/:gameId/overview`
- `/admin/games/:gameId/live-overview`
- `/admin/games/:gameId/teams/new`
- `/admin/games/:gameId/teams/:teamId/edit`
- `/admin/games/:gameId/members/new`
- `/admin/games/:gameId/members/:userId/edit`

### Game-Specific Admin Pages

- GeoHunter: `/admin/geohunter/:gameId/pois`
- Resource Run: `/admin/resource-run/:gameId/nodes`
- Territory Control: `/admin/territory-control/:gameId/zones`
- Blind Hike: `/admin/blindhike/:gameId/configure`
- Echo Hunt: `/admin/echo-hunt/:gameId/beacons`
  - `/admin/echo-hunt/:gameId/beacons/new`
  - `/admin/echo-hunt/:gameId/beacons/:beaconId/edit`
- Checkpoint Heist: `/admin/checkpoint-heist/:gameId/checkpoints`
- Courier Rush: `/admin/courier-rush/:gameId/configure`
- Pandemic Response: `/admin/pandemic-response/:gameId/hotspots`
- Market Crash:
  - `/admin/market-crash/:gameId/resources`
  - `/admin/market-crash/:gameId/resources/new`
  - `/admin/market-crash/:gameId/resources/:resourceId/edit`
  - `/admin/market-crash/:gameId/points`
  - `/admin/market-crash/:gameId/points/new`
  - `/admin/market-crash/:gameId/points/:pointId/edit`
- Birds of Prey: `/admin/birds-of-prey/:gameId/configure`
- Code Conspiracy: `/admin/code-conspiracy/:gameId/configure`
- Crazy 88: `/admin/crazy88/:gameId/tasks`

### Admin Team Play Helper

- `/admin/games/:gameId/teams/:teamId/play`

## Account Routes (`/account/*`)

- `/account/profile`
  - user self-service profile (email, display name, password)
- `/account/subscription`
  - subscription management page (moved from direct `/admin/subscription` access)
- `/account/payments`
  - payment history

Notes:
- When monetisation is disabled, account sidebar hides `subscription` and `payments`, and direct navigation to those routes redirects to `/account/profile`.

## Super Admin App (`admin/`)

Standalone control-plane routes:

- `/login`
- `/dashboard`
- `/dashboard/game-modes`
- `/dashboard/users`
- `/dashboard/users/new`
- `/dashboard/users/:userId/edit`

Notes:
- This app is separate from main frontend runtime routes.
- Access is restricted to users with `ROLE_SUPER_ADMIN`.

## UX Separation Rules

- Do not expose admin capabilities under team routes.
- Avoid cross-role controls on same page.
- Keep live-overview responsibilities in admin pages.
