# WS Events Reference

This is the centralized contributor-facing catalog of websocket events used across JotiGames.

- Backend publishes domain events.
- WS transports to channels.
- Frontend consumes events and patches runtime state.

Detailed payload examples are maintained in `backend/docs/ws-events-reference.md`.

## Channel Targets

- `channel:{game_id}` (game-wide)
- `channel:{game_id}:{team_id}` (team-private)
- `channel:{game_id}:admin` (admin-private)

## Game Events

- `game.blind_hike.marker.added`
- `game.birds_of_prey.team.score`
- `game.market_crash.team.score`
- `game.market_crash.prices.updated`
- `game.exploding_kittens.highscore.adjust`
- `game.general.team.update`
- `game.general.team.add`
- `game.general.team.remove`
- `game.chat.message`

## Admin Events

- `admin.blind_hike.marker.added`
- `admin.birds_of_prey.team.location.updated`
- `admin.birds_of_prey.egg.added`
- `admin.birds_of_prey.egg.removed`
- `admin.birds_of_prey.team.score`
- `admin.market_crash.team.location.updated`
- `admin.market_crash.team.score`
- `admin.market_crash.trade.executed`
- `admin.market_crash.prices.updated`
- `admin.message.team`
- `admin.general.team.update`
- `admin.general.team.add`
- `admin.general.team.remove`
- `admin.exploding_kittens.card.adjust_amount`
- `admin.exploding_kittens.state.activate`
- `admin.exploding_kittens.state.deactivate`
- `admin.exploding_kittens.lives.updated`
- `admin.exploding_kittens.action.add`
- `admin.exploding_kittens.action.remove`

## Team Events

- `team.blind_hike.marker.added`
- `team.birds_of_prey.self.updated`
- `team.birds_of_prey.enemy_eggs.visible`
- `team.birds_of_prey.egg.added`
- `team.birds_of_prey.egg.removed`
- `team.market_crash.self.updated`
- `team.market_crash.nearby_points.updated`
- `team.market_crash.prices.updated`
- `team.general.team.update`
- `team.general.message`
- `team.exploding_kittens.card.add`
- `team.exploding_kittens.card.remove`
- `team.exploding_kittens.state.activate`
- `team.exploding_kittens.state.deactivate`
- `team.exploding_kittens.lives.updated`
- `team.exploding_kittens.action.add`
- `team.exploding_kittens.action.remove`

## Transport Event

- `core.connected`

## Contract Rules

1. Reuse an existing event before introducing a new one.
2. If adding an event, update this file and backend payload reference in same change set.
3. Keep names and payload shapes synchronized across backend emitter, ws transport, and frontend consumers.
4. Keep payloads patch-oriented and minimal.
