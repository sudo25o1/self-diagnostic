# QMD Initialization Gap — Why It Doesn't Auto-Start

## What's Happening

When `qmd query` runs inside the significance extension (during `before_agent_start`), it uses a **separate, agent-scoped QMD index** — not the workspace-level index that `qmd update` and `qmd embed` populate.

The extension sets custom XDG environment variables before spawning `qmd`:

```typescript
const qmdDir = path.join(stateDir, "agents", agentId, "qmd");
const xdgConfigHome = path.join(qmdDir, "xdg-config");
const xdgCacheHome = path.join(qmdDir, "xdg-cache");
```

This isolates the agent's QMD state to `/Users/bernard/.openclaw/agents/main/qmd/`. That index has **0 files indexed** and **no collections defined** — it was never set up.

The workspace-level index at `~/.cache/qmd/` (which I just initialized manually) is a **completely separate index** from what the extension actually queries. Running `qmd update` and `qmd embed` in the workspace does nothing for the extension's queries.

---

## Why the Queries Were Returning Garbage

The agent-scoped index at:
```
/Users/bernard/.openclaw/agents/main/qmd/xdg-cache/qmd/index.sqlite
```
...has no collections and no documents. When the extension calls `qmd query "recent tasks"`, it gets no real results, falls back to the hyde (hypothetical document expansion) pass, which generates generic boilerplate like "Understanding Scott RPG game is essential for modern development." That's not memory retrieval — it's hallucination filling the void.

---

## What Needs to Happen

There are two problems to fix, and they're separate:

**Problem 1: The agent-scoped index has no collections.**

On first run (or startup), the code needs to create collections pointing at the workspace memory files, then index them. Currently there is zero code that does this. The `registerService.start()` function in `significance/index.ts` does NOT call `qmd update` or `qmd embed` — it only starts the idle check-in interval. No QMD initialization whatsoever.

**Problem 2: No auto-re-index on gateway restart.**

Even if the agent-scoped index were set up once, there's no code to re-run `qmd update` when the gateway starts to pick up changes to memory files since the last run.

---

## The Fix

In `extensions/significance/index.ts`, inside the `registerService.start()` function, add a QMD init sequence:

```typescript
api.registerService({
  id: "significance",
  start: async (ctx) => {
    // ... existing code ...

    // Initialize agent-scoped QMD index on startup
    await initQmd({ config: api.config, agentId: "main" });
  }
});
```

The `initQmd` function (to add to `src/qmd-context.ts`) needs to:

1. Check if collections exist in the agent-scoped index
2. If not, run `qmd collection add` for each needed path (MEMORY.md, memory/ dir, session logs)
3. Run `qmd update` to index any new/changed files
4. Run `qmd embed` to generate/update vectors (can be async/background — it takes ~20s on M4)

The environment for all these calls must use the same `XDG_CONFIG_HOME` and `XDG_CACHE_HOME` overrides that `queryQmd` already uses — so everything hits the same isolated index.

---

## Quickest Path to Working

The fastest fix is adding this to `registerService.start()` in `significance/index.ts`:

```typescript
// Spawn qmd update + embed in background on startup
const qmdEnv = {
  ...process.env,
  XDG_CONFIG_HOME: xdgConfigHome,
  XDG_CACHE_HOME: xdgCacheHome,
  NO_COLOR: "1",
};

// First time setup: add collections if index is empty
const { execFile } = await import("node:child_process");
const util = await import("node:util");
const execFileAsync = util.promisify(execFile);

try {
  // Add collection for workspace memory files (idempotent — fails silently if exists)
  await execFileAsync("qmd", ["collection", "add", workspaceDir, "--name", "memory", "--mask", "memory/**/*.md"], { env: qmdEnv, cwd: workspaceDir }).catch(() => {});
  await execFileAsync("qmd", ["collection", "add", workspaceDir, "--name", "memory-root", "--mask", "MEMORY.md"], { env: qmdEnv, cwd: workspaceDir }).catch(() => {});
  
  // Update index (fast — only processes changed files)
  await execFileAsync("qmd", ["update"], { env: qmdEnv, cwd: workspaceDir });
  
  // Embed in background (slow — don't await)
  execFile("qmd", ["embed"], { env: qmdEnv, cwd: workspaceDir });
  
  api.logger.info("significance: QMD index initialized");
} catch (err) {
  api.logger.warn(`significance: QMD init failed: ${String(err)}`);
}
```

This would run every gateway start. `qmd update` is fast (only re-indexes changed files). `qmd embed` runs in the background. Collections are idempotent so re-adding on every start is harmless.

---

## Why I Ran `qmd update` in the Wrong Place

When I ran `qmd update` and `qmd embed` earlier, I ran them in the workspace directory without the XDG overrides. That populated the default `~/.cache/qmd/` index — not the agent-scoped index that the extension actually uses. The extension's queries continued hitting an empty index. My fix was essentially writing to the wrong database.

---

## Files to Change

- `extensions/significance/index.ts` — add QMD init to `registerService.start()`
- `extensions/significance/src/qmd-context.ts` — optionally extract the init logic into a reusable `initQmdCollections()` function
- After changes: `npm run build` in `~/bernard-repo`, then restart gateway
