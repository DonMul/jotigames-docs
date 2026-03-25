# GeoHunter

## Objective

Achieve the highest correct-answer score by game end.

## Core flow

1. Admin configures POIs with radius and Q/A.
2. Teams move into POI range.
3. Teams submit answers.
4. Correct answers increase score.

## Relevant pages

- Admin POIs: `/admin/geohunter/:gameId/pois`
- Admin live overview: `/admin/games/:gameId/live-overview`
- Team dashboard panel: `/team`

## Team panel component

`frontend/src/pages/team/GeoHunterTeamPanel.jsx`

- Leaflet map with POI markers
- GPS tracking with haversine proximity detection
- Question modal with multiple-choice radio buttons or open text input
- Client-side answer checking before submission
- Supports retry with configurable timeout
- Props: `state`, `currentTeamId`, `t`, `onAnswerQuestion`, `answering`

## Bootstrap data

Service override in `backend/app/services/geohunter_service.py` adds:
- `pois[]` — id, title, lat, lon, radius_meters, points, marker_color, is_active, question_type, question_text, correct_answer, choices[]
- `retry_enabled` — whether retries are allowed
- `retry_timeout_seconds` — cooldown between retries
- `highscore[]` — team leaderboard rows

## Realtime highlights

- `team.geohunter.*` → triggers full state reload
- `game.geohunter.*` → triggers full state reload

## Page descriptions

- POIs page: create/edit POIs, answer format, retry behavior.
- Team dashboard panel: proximity and answer interactions.

## Screenshot

![GeoHunter pages](../screenshots/geohunter-pages.png)

## Runtime screenshots

### Team dashboard (`/team`)

Shows in-range POI question flow, answer submission, and immediate scoring context.

![GeoHunter team dashboard](../screenshots/geohunter-team-dashboard.png)

### Admin live overview (`/admin/games/:gameId/live-overview`)

Shows POI activity, answer throughput, and leaderboard movement.

![GeoHunter live overview](../screenshots/geohunter-live-overview.png)
