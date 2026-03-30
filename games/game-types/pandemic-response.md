# Pandemic Response

## Objective

Achieve the highest response score by resolving hotspots.

## Core flow

1. Admin configures hotspot and severity model settings.
2. Backend worker maintains target active hotspots/pickups and escalates unresolved hotspots on the configured interval.
3. Teams enter hotspot areas to resolve incidents.
4. Response scoring and overflow penalties are applied by config.

## Relevant pages

- Public info page: `/info/games/pandemic-response`
- Admin hotspots/config: `/admin/pandemic-response/:gameId/hotspots`
- Admin live overview: `/admin/games/:gameId/live-overview`
- Team dashboard panel: `/team`

## Team panel component

`frontend/src/pages/team/PandemicResponseTeamPanel.jsx`

- Leaflet map with hotspot circles (red, sized by severity) and pickup circles (blue)
- GPS tracking with haversine proximity detection
- Collect button for pickups, Resolve button for hotspots when in range
- Uses `actionPathOverride` for hotspot endpoint (`hotspot/resolve`)
- Props: `state`, `currentTeamId`, `t`, `onCollectPickup`, `onResolveHotspot`, `collectingPickup`, `resolvingHotspot`

## Bootstrap data

Service override in `backend/app/services/pandemic_response_service.py` adds:
- `hotspots[]` — id, title, lat, lon, radius_meters, points, marker_color, is_active, severity
- `pickups[]` — id, title, lat, lon, radius_meters, points, marker_color, is_active
- `highscore[]` — team leaderboard rows

## API endpoints (team actions)

- `POST /{game_id}/teams/{team_id}/pickup/collect` — default action path
- `POST /{game_id}/teams/{team_id}/hotspot/resolve` — uses actionPathOverride

## Realtime highlights

- `team.pandemic_response.*` → triggers full state reload
- `game.pandemic_response.*` → triggers full state reload

## Page descriptions

- Public info page: detailed landing/how-to-play page grounded in hotspot severity, pickup collection, and resolve actions under live response pressure.
- Hotspots/config page: center/spawn rules, severity timing, penalties, pickup count, and spawn-area polygon editing via map clicks + draggable vertices (double-click vertex removal and closest-edge insertion for new points) with no visible raw GeoJSON field.
- Team dashboard panel: hotspot resolution feedback and team score effects.

## Screenshot

![Pandemic Response pages](../screenshots/pandemic-response-pages.png)

## Runtime screenshots

### Team dashboard (`/team`)

Shows hotspot response flow, pickup interactions, and penalty/reward feedback.

![Pandemic Response team dashboard](../screenshots/pandemic-response-team-dashboard.png)

### Admin live overview (`/admin/games/:gameId/live-overview`)

Shows hotspot lifecycle, team response effectiveness, and live scoreboard impact.

![Pandemic Response live overview](../screenshots/pandemic-response-live-overview.png)
