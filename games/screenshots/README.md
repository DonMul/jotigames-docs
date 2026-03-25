# Screenshots

This folder contains screenshot assets referenced by the game documentation pages.

Screenshots are generated automatically from live pages with Playwright.

## Generate / refresh screenshots

From repository root:

```bash
cd e2e
npm run docs:screenshots
```

The command:

- logs in with `E2E_USER_EMAIL` / `E2E_USER_PASSWORD`
- creates temporary game contexts for each documented game type through backend API
- creates a temporary team for team dashboard capture
- captures every documented games page route
- captures per-game team runtime (`/team`) and admin runtime (`/admin/games/:gameId/live-overview`) screenshots
- updates markdown image links in `docs/games/**/*.md` to point at current `.png` assets
- cleans up temporary games after capture by default

Environment variables:

- `FRONTEND_BASE_URL` (default `http://localhost:5173`)
- `BACKEND_BASE_URL` (default `http://localhost:8000`)
- `E2E_USER_EMAIL` (default `jotigames-copilot@jmul.net`)
- `E2E_USER_PASSWORD` (default `Test1234`)
- `DOCS_SCREENSHOTS_KEEP_GAMES=1` to keep created game fixtures for debugging

## Naming convention

- Core pages: `team-dashboard.png`, `admin-live-overview.png`, `game-configuration.png`
- Shared: `shared-lifecycle.png`
- Game pages: `<game>-pages.png`
- Per-game runtime: `<game>-team-dashboard.png`, `<game>-live-overview.png`

## Capture checklist

- Team Dashboard (`/team`)
- Admin Live Overview (`/admin/games/:gameId/live-overview`)
- Game configuration surfaces (`/admin/games/new`, `/admin/games/:gameId`, per-game admin route)
- Every game-specific admin page listed in `docs/games/game-types/`
- Team dashboard per game type (`/team`)
- Live overview per game type (`/admin/games/:gameId/live-overview`)
