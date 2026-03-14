# @jungjaehoon/openclaw-mama

MAMA memory plugin for OpenClaw Gateway.

This package embeds `@jungjaehoon/mama-core` directly into OpenClaw as a `memory` slot plugin, so the gateway can expose MAMA tools without an HTTP bridge.

## Requirements

- OpenClaw `2026.2.22` or newer
- Node.js `22.13.0` or newer

Older OpenClaw releases can load the plugin and run the tools, but `openclaw status` may report third-party memory plugins as unavailable even when they are working.

## What It Provides

- 4 native tools: `mama_search`, `mama_save`, `mama_load_checkpoint`, `mama_update`
- session lifecycle hooks for recall, checkpointing, and compaction recovery
- direct module integration with `@jungjaehoon/mama-core`
- plugin-scoped config via `plugins.entries.openclaw-mama.config`

## Compatibility Notes

- minimum tested OpenClaw version: `2026.2.22`
- tested against OpenClaw runtime via gateway startup, `status`, and `/tools/invoke`
- `plugins.entries.openclaw-mama.config.dbPath` is supported and verified in real runtime
- `@jungjaehoon/mama-core` is pinned to the current compatibility line: `^1.3.1`

## Installation

### From npm

```bash
openclaw plugins install @jungjaehoon/openclaw-mama
```

### From local source

```bash
git clone https://github.com/jungjaehoon-lifegamez/openclaw-mama.git
cd openclaw-mama
npm install

# Link the current checkout into an isolated OpenClaw profile
npx openclaw --profile codex-mama-e2e plugins install . --link
```

## Recommended Config

For local development and explicit trust pinning:

```json
{
  "gateway": {
    "mode": "local"
  },
  "plugins": {
    "allow": ["openclaw-mama"],
    "load": {
      "paths": ["/path/to/your/cloned/openclaw-mama"]
    },
    "slots": {
      "memory": "openclaw-mama"
    },
    "entries": {
      "openclaw-mama": {
        "enabled": true,
        "config": {
          "dbPath": "/path/to/your/openclaw-mama.db"
        }
      },
      "memory-core": {
        "enabled": false
      },
      "memory-lancedb": {
        "enabled": false
      }
    }
  }
}
```

Why `plugins.allow` matters:

- without it, OpenClaw warns that discovered non-bundled plugins may auto-load
- adding `["openclaw-mama"]` removes that warning and makes trust explicit

## Tools

### mama_search

Semantic search over saved decisions.

### mama_save

Save a decision or checkpoint.

### mama_load_checkpoint

Load the latest checkpoint plus recent decisions.

### mama_update

Update a saved decision outcome.

## Runtime Smoke Test

This is the shortest useful developer validation flow.

### 1. Install into an isolated profile

```bash
npx openclaw --profile codex-mama-e2e plugins install . --link
npx openclaw --profile codex-mama-e2e config set plugins.allow '["openclaw-mama"]'
npx openclaw --profile codex-mama-e2e config set plugins.slots.memory '"openclaw-mama"'
npx openclaw --profile codex-mama-e2e config set plugins.entries.openclaw-mama.enabled true
npx openclaw --profile codex-mama-e2e config set plugins.entries.openclaw-mama.config.dbPath '"/path/to/your/mama-e2e.db"'
```

### 2. Start the gateway

```bash
npx openclaw --profile codex-mama-e2e gateway run --force --verbose
```

### 3. Check health and status

```bash
npx openclaw --profile codex-mama-e2e health --json
npx openclaw --profile codex-mama-e2e status --json
```

Expected status signals:

- `memoryPlugin.slot` is `openclaw-mama`
- gateway is reachable
- older `unavailable` wording should not appear on supported OpenClaw versions

### 4. Invoke the tools through the gateway

```bash
curl -s -X POST 'http://127.0.0.1:18789/tools/invoke' \
  -H 'Authorization: Bearer <gateway-token>' \
  -H 'Content-Type: application/json' \
  --data '{"tool":"mama_search","args":{"query":"openclaw compat","limit":3}}'

curl -s -X POST 'http://127.0.0.1:18789/tools/invoke' \
  -H 'Authorization: Bearer <gateway-token>' \
  -H 'Content-Type: application/json' \
  --data '{"tool":"mama_save","args":{"type":"decision","topic":"runtime_smoke","decision":"Use openclaw-mama","reasoning":"Runtime smoke test. builds_on: smoke_seed","confidence":0.9}}'
```

Verified in this branch:

- `mama_search`
- `mama_save`
- `mama_load_checkpoint`
- `mama_update`

## Session Lifecycle Hooks

The plugin registers these OpenClaw hooks:

| Hook                 | Trigger                    | Action                                    |
| -------------------- | -------------------------- | ----------------------------------------- |
| `session_start`      | Session begins             | Load checkpoint and recent decisions      |
| `before_agent_start` | Before each agent turn     | Inject relevant memories into context     |
| `agent_end`          | After agent completes      | Detect decision patterns (auto-capture)   |
| `session_end`        | Session ends               | Auto-save checkpoint                      |
| `before_compaction`  | Before context compression | Save checkpoint with pre-compaction state |
| `after_compaction`   | After context compression  | Prepare recovery context for next turn    |

## Architecture

```
OpenClaw Gateway
└── MAMA Plugin (this package)
    └── @jungjaehoon/mama-core
        ├── mama-api.js (high-level API)
        ├── memory-store.js
        └── embeddings.js (Transformers.js)
```

Notes:

- there is no HTTP bridge between OpenClaw and MAMA core
- current `mama-core` uses pure TypeScript cosine similarity and no longer requires plugin-side sqlite-vec setup
- the plugin now relies on OpenClaw's current SDK tool contract, including `label` and structured `details`

## Migration from clawdbot-mama

If upgrading from `@jungjaehoon/clawdbot-mama`:

1. Uninstall old plugin: `openclaw plugins uninstall clawdbot-mama`
2. Install new plugin: `openclaw plugins install @jungjaehoon/openclaw-mama`
3. Update config: Replace `"clawdbot-mama"` with `"openclaw-mama"` in `openclaw.json`

Your decision database (`~/.claude/mama-memory.db`) is preserved.

## Development Notes

- Run typecheck: `npm run typecheck`
- Run tests: `npm test`
- For config-isolated runtime checks, prefer a dedicated OpenClaw profile
- If you see the warning about discovered non-bundled plugins, set `plugins.allow`

## License

MIT
