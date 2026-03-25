# Shared Structure (All Games)

This document describes the shared flow across all game types in JotiGames.

## Admin lifecycle

1. Create game via `/admin/games/new`.
2. Open game details at `/admin/games/:gameId`.
3. Configure game-specific entities/settings on the module page.
4. Add teams and members.
5. Monitor runtime in `/admin/games/:gameId/live-overview`.

## Team lifecycle

1. Team logs in using team code.
2. Team opens `/team`.
3. Team performs game-specific interactions.
4. Team receives realtime state updates and score changes.

## Shared relevant pages

- Team dashboard: `/team`
- Admin game details: `/admin/games/:gameId`
- Admin live overview: `/admin/games/:gameId/live-overview`

## Screenshot

![Shared gameplay lifecycle](screenshots/shared-lifecycle.png)