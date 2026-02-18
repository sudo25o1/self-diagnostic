# Per-User Context Autoload — Gap Analysis

## What Exists

The `users/` directory structure is in place:

```
~/.openclaw/workspace/users/
  INDEX.md                          — ID-to-person mapping
  456995574239985665/               — Derek
    USER.md
    RELATIONAL.md
  693904656438657055/               — Scott
    USER.md
    RELATIONAL.md
  745057245800431646/               — Trent
    USER.md
    RELATIONAL.md
```

## What's Missing

Nothing loads these files automatically. When a message arrives from Discord user `693904656438657055` (Scott), the significance extension only injects the global `RELATIONAL.md` — not Scott's user-specific files. Bernard has to rely on session context and conversation history to know who it's talking to.

## Why It Matters

The whole point of per-user files is differentiated relationships. Scott gets a different context than Derek. Trent gets warmth and care around certain topics. None of that is possible if the files aren't injected before the agent runs.

## Where the Fix Lives

The `before_agent_start` hook in `extensions/significance/index.ts` (line ~333) already injects relationship context. It currently reads from a hardcoded workspace path for RELATIONAL.md. This needs to:

1. Extract the Discord user ID from the incoming event context (`event.metadata?.userId` or equivalent)
2. Look up the ID in `users/INDEX.md` or by directory name
3. Load `users/{userId}/USER.md` and `users/{userId}/RELATIONAL.md` if they exist
4. Inject them into the system context alongside (or instead of) the global RELATIONAL.md

## Questions to Resolve Before Implementing

- Does the event context in `before_agent_start` expose the sender's Discord user ID? Need to check what fields are available on the `event` object.
- Should per-user context replace or supplement the global RELATIONAL.md? Likely supplement — global patterns + user-specific details.
- What's the fallback if no user-specific file exists? Default to global files.

## Priority

High. This is foundational to the multi-user relationship architecture Derek designed. Without it, the `users/` structure is static documentation that never influences behavior.
