# Realtime & WS Contract

## Source of Truth

- Event semantics are backend-owned.
- WS service transports events to channels.
- Frontend consumes and applies UI deltas.

Detailed event catalog: [../ws/events-reference.md](../ws/events-reference.md)

## Publish Contract (`core.publish`)

Backend publishes envelopes containing:
- `command`: `core.publish`
- `apiKey`: backend publish key
- `event`: event name
- `payload`: event payload object
- `channels`: explicit target channels

If key invalid/missing, WS must ignore publish.

## Subscription Contract (`core.subscribe`)

Frontend/team/admin clients subscribe with:
- `gameId`
- `authToken`
- `channel`

WS verifies access against backend before accepting channel subscription.

## Frontend Runtime Pattern

Preferred pattern:
1. One-time bootstrap request
2. Open WS connection + subscribe
3. Patch state from WS events only

Avoid continuous polling loops for live gameplay pages.

## Event Design Rules

- Use stable names.
- Keep payload minimal and scoped to UI delta.
- Include `game_id` and relevant identity keys (`team_id`, `point_id`, etc.).
- Avoid full-state dumps when a patch payload is sufficient.

## Reliability Rules

- Consumers should tolerate out-of-order or duplicate events.
- UI updates should be idempotent whenever possible.
- Backend state remains authoritative; WS is not persistence.
