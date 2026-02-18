# gog Auth Gap — Analysis

## Current State

The `gog` skill (Google Workspace CLI for Gmail, Calendar, Drive) is installed but not authenticated. Bernard has zero visibility into:

- Derek's email inbox (cannot read replies, cannot send emails directly)
- Calendar (cannot check upcoming events, cannot be proactive about scheduling)
- Drive (cannot access shared documents)

## Impact

Every email-related task requires manual copy-paste from Derek. The Matt deal thread, for example — Bernard drafted responses but had no ability to confirm whether they were sent, read replies, or monitor for a response. Same for Scott's request to email Ahra@phi.health.

The proactive check-in system (HEARTBEAT.md) calls out email as a periodic check, but that check is effectively disabled.

## How to Fix

Run from the Mac:

```
gog auth
```

This will open a browser OAuth flow. Once authenticated, Bernard can:
- Read and search Gmail
- Send email directly (no copy-paste)
- Read calendar for upcoming events
- Be proactive about time-sensitive threads

## Timing

Needs Derek physically present at the machine to complete the OAuth flow. Should take under 2 minutes. The `gog auth` command is the entire process.

## Priority

Medium-High. Directly unlocks the Matt thread monitoring and makes Bernard significantly more autonomous on communications. Before Monday (when Matt returns from surgery) is the ideal window.
