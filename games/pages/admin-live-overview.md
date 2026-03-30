# Admin Live Overview

Route: `/admin/games/:gameId/live-overview`

## Purpose

The Admin Live Overview is the main runtime control and observation page for game staff during active play.

## What admins do here

- Monitor team status and scoreboard/highscore movement.
- Watch game-type specific live entities on map-first runtime panels (for example markers, POIs, zones, checkpoints, points, eggs).
- Use operational controls and communication actions where available.
- Validate progression and intervene with admin workflows when needed.

## Current runtime layout contract

- Map-first overview for location games (GeoHunter, Echo Hunt, Territory Control, Checkpoint Heist, Market Crash, Birds of Prey, Blind Hike).
- Team metric cards are score-first by default; only game-specific exceptions are shown (for example Blind Hike markers, Exploding Kittens lives).
- Live overview removes configuration/settings noise; configuration stays on dedicated settings pages.
- Crazy 88 live overview focuses on score movement + review queue threads (accept/reject + unlock review assignment), not full task catalog editing.

## Notes

- This route is rendered by `ModuleOverviewPage`.
- Behavior differs by game type while keeping one consistent admin runtime entry point.

## Screenshot

![Admin Live Overview](../screenshots/admin-live-overview.png)