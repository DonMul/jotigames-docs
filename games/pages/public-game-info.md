# Public Game Info Page (`/info/games/:slug`)

## Purpose

This route is the public sales and onboarding page for an individual game type. It should help organizers understand the game fantasy, the live mechanics teams will actually feel, and the kind of event format where the game works best.

## Implementation

- Route: `/info/games/:slug`
- Frontend controller: `frontend/src/pages/GameInfoPage.jsx`
- Shared renderer for supported detailed pages: `frontend/src/components/DetailedGameInfoPage.jsx`
- Per-game metadata source: `frontend/src/lib/detailedGameInfoContent.js`
- Dutch copy source: `frontend/src/i18n/locales/nl.json`

## Content contract

- Public copy must stay grounded in the live gameplay rules from backend services and the game handbook docs.
- Avoid generic marketing claims that are not traceable to real mechanics.
- The "View prices" CTA only renders when monetisation is enabled (`/api/subscription/status` returns `enabled: true`).
- Birds of Prey keeps its custom bespoke layout.
- The remaining supported game types use the shared detailed renderer with page-level content assembly based on locale keys.

## Supported detailed pages

- `/info/games/birds-of-prey`
- `/info/games/blind-hike`
- `/info/games/checkpoint-heist`
- `/info/games/code-conspiracy`
- `/info/games/courier-rush`
- `/info/games/crazy-88`
- `/info/games/echo-hunt`
- `/info/games/exploding-kittens`
- `/info/games/geohunter`
- `/info/games/market-crash`
- `/info/games/pandemic-response`
- `/info/games/resource-run`
- `/info/games/territory-control`

## Required sections

Each detailed public page should explain:

1. What makes the mode exciting.
2. Which live mechanics define the round.
3. How a team experiences the flow step by step.
4. Why the mode is useful for organizers.
5. Which terrains or event styles fit the mode best.

## Maintenance notes

- When backend rules change, review the matching public page copy.
- When detailed page copy changes, keep `GameInfoPage.jsx`, `nl.json`, and the handbook docs aligned.
- When a new game type is added to the catalog, update this file and the matching `docs/games/game-types/*.md` handbook entry.