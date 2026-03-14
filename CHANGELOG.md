# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed

- Updated `@jungjaehoon/mama-core` compatibility to `^1.3.1`
- Raised the minimum supported OpenClaw version to `2026.2.22`
- Declared Node.js runtime requirement `>=22.13.0`
- Updated native tool definitions to match the current OpenClaw SDK contract by adding tool labels and structured tool result details

### Fixed

- Read plugin-scoped config from `api.pluginConfig` so `plugins.entries.openclaw-mama.config.dbPath` is applied in real OpenClaw runtime
- Removed the local `openclaw.d.ts` shim in favor of current OpenClaw SDK types

### Verified

- Confirmed the plugin loads as a `memory` slot plugin in OpenClaw runtime
- Confirmed `mama_search`, `mama_save`, `mama_load_checkpoint`, and `mama_update` through gateway `/tools/invoke`
- Confirmed `openclaw status --json` reports `memoryPlugin.slot = openclaw-mama` on supported OpenClaw versions

### Docs

- Reworked README for standalone repository usage
- Added explicit `plugins.allow` guidance for trusted local plugin loading
- Added developer-oriented runtime smoke test steps
- Updated architecture notes to reflect direct `mama-core` integration and current runtime behavior

## [0.5.0] - 2026-02-15

### Added

- **Session Lifecycle Hooks**: Complete session memory management
  - `session_start`: Loads checkpoint and recent decisions at session start
  - `session_end`: Auto-saves checkpoint when session ends
  - `before_compaction`: Saves checkpoint before context compression
  - `after_compaction`: Prepares context recovery after compression
- **Post-Compaction Context Enhancement**: Automatically adds note to context after compaction to help restore state
- **Better than Claude Code**: OpenClaw's `after_compaction` hook enables immediate state awareness after compression (Claude Code only has PreCompact)

### Changed

- Enhanced `before_agent_start` hook to detect post-compaction state and inject recovery context

## [0.4.0] - 2026-02-01

### Changed

- **BREAKING**: Renamed from `@jungjaehoon/clawdbot-mama` to `@jungjaehoon/openclaw-mama`
- Updated plugin manifest to `openclaw.plugin.json`
- Changed plugin ID from `clawdbot-mama` to `openclaw-mama`
- Updated imports to use `OpenClawPluginApi` instead of `ClawdbotPluginApi`

### Migration

If upgrading from `@jungjaehoon/clawdbot-mama`:

1. Uninstall old plugin: `openclaw plugins uninstall clawdbot-mama`
2. Install new plugin: `openclaw plugins install @jungjaehoon/openclaw-mama`
3. Update config: Replace `"clawdbot-mama"` with `"openclaw-mama"` in `openclaw.json`

Your decision database (`~/.claude/mama-memory.db`) is preserved.

## [0.3.0] - 2026-01-28

### Added

- Initial OpenClaw Gateway plugin integration
- Direct embedding without HTTP overhead (~5ms vs ~180ms)
- 4 native tools: `mama_search`, `mama_save`, `mama_load_checkpoint`, `mama_update`

### Features

- **Semantic Search** - Vector-based decision retrieval using sqlite-vec
- **Decision Graph** - Track decision evolution with edge types
- **Checkpoint System** - Session state preservation and resumption

### Technical Details

- Uses `@jungjaehoon/mama-core` for core functionality
- TypeBox for schema validation
- Requires OpenClaw >= 2026.1.26 as peer dependency
