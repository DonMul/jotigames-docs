# Operations: Workers, Cron, Runbooks

## Worker Architecture

Long-running game workers are started through cron `@reboot` entries with `flock` lock files.

Managed files:
- `system/cron/crontab.template`
- `system/cron/restart_workers.sh`

Rule: add every new worker to both files.

## Current Workers

- `backend/scripts/auto_resolve_pending_actions.py`
  - Exploding Kittens pending action timeout resolver.
- `backend/scripts/birds_of_prey_auto_drop_eggs.py`
  - Periodic Birds of Prey egg drops.
- `backend/scripts/market_crash_fluctuate_prices.py`
  - Continuous Market Crash point-price fluctuation worker.

## Deploy/Restart Flow

- Deploy script: `system/deploy_update.sh`
- It must invoke worker restart script to avoid requiring reboot.
- Manual restart command:

```bash
bash system/cron/restart_workers.sh
```

## Log Locations

Worker logs are written under:
- `system/logs/*.log`

Recommended checks:
- verify lock and log files exist
- verify process alive after restart
- verify no duplicate worker processes

## Incident Runbook (Quick)

### Worker appears stopped

1. Check latest logs in `system/logs/`.
2. Run `system/cron/restart_workers.sh`.
3. Confirm process list and lock files.
4. Verify game behavior impacted by worker.

### WS events not reaching clients

1. Confirm backend can publish to WS (key present).
2. Confirm WS server running and subscriptions accepted.
3. Validate event name/channel in `docs/ws/events-reference.md`.
4. Check frontend subscription channels for current role/game/team.

### Game state mismatch report

1. Treat backend DB state as authoritative.
2. Reproduce action via API and inspect resulting events.
3. Validate frontend patch logic for affected event.
4. Add regression test/fix and update docs if contract changed.
