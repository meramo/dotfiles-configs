# OpenClaw

## What It Is
AI assistant platform with a WebSocket gateway, chat channel integrations (Telegram, Discord), agent system, browser control, cron jobs, and memory/session management.

## Installation & Paths
- Binary: `/opt/homebrew/bin/openclaw` (installed via Homebrew)
- Config: `~/.openclaw/openclaw.json` (JSON, has backup rotation `.bak`, `.bak.1`, etc.)
- Logs: `~/.openclaw/logs/gateway.log` and `~/.openclaw/logs/gateway.err.log`
- Workspace: `~/.openclaw/workspace/`
- Sessions: `~/.openclaw/agents/main/sessions/sessions.json`
- State dir: `~/.openclaw/`

## Gateway
- Runs as a macOS LaunchAgent: `ai.openclaw.gateway`
- Plist: `~/Library/LaunchAgents/ai.openclaw.gateway.plist`
- Default port: **18789** (WebSocket on loopback)
- Browser control port: **18791**
- Process name in `ps`: `openclaw-gateway`
- Auth: token-based (`gateway.auth.token` in config)

## Key Commands
- `openclaw health` — check gateway + channel status
- `openclaw doctor` — diagnostics (use `--fix` to auto-repair)
- `openclaw gateway restart` — restart via launchd
- `openclaw config set <path> <value>` — update config
- `openclaw gateway start --log-level debug` — start with verbose logging

## Debugging Checklist (when "not starting")
1. **Check if gateway is listening**: `lsof -i :18789 -P`
2. **Check process**: `ps aux | grep openclaw-gateway`
3. **Check launchd**: `launchctl print gui/501/ai.openclaw.gateway` (look at `state` and last exit code)
4. **Read error log**: `tail -100 ~/.openclaw/logs/gateway.err.log`
5. **Read gateway log**: `tail -50 ~/.openclaw/logs/gateway.log`
6. **Run doctor**: `openclaw doctor`
7. **Check health**: `openclaw health`

## Known Issues & Fixes

### Sandbox mode requires Docker (2026-02-28)
- **Symptom**: Gateway starts but agent tasks fail immediately. Error log shows `spawn docker ENOENT`. CLI commands fail with `gateway closed (1006 abnormal closure)`.
- **Cause**: `agents.defaults.sandbox.mode` was set to `"all"` but Docker is not installed.
- **Fix**: `openclaw config set agents.defaults.sandbox.mode off` then `openclaw gateway restart`

### Unrecognized config keys crash startup
- **Symptom**: Error log shows `Config invalid` with `Unrecognized key: "fs"` (or similar).
- **Cause**: Stale/invalid keys in `openclaw.json` (e.g. `agents.defaults.fs`).
- **Fix**: `openclaw doctor --fix` removes unknown keys. Or manually edit `~/.openclaw/openclaw.json`.

### Control UI "origin not allowed" (v2026.3.7+)
- **Symptom**: Control UI fails to connect, error log shows `code=1008 reason=origin not allowed (open the Control UI from the gateway host or allow it in gateway.controlUi.allowedOrigins)` with `origin=http://127.0.0.1:18789`.
- **Cause**: v2026.3.7 introduced stricter WebSocket origin validation for the Control UI. The gateway's own origin (`http://127.0.0.1:18789`) was not automatically trusted.
- **Fix**: `openclaw config set gateway.controlUi.allowedOrigins '["http://127.0.0.1:18789"]'` then `openclaw gateway restart`

### Config reload failures
- If config changes are invalid, the gateway logs `config reload skipped (invalid config)` but keeps running with old config. A restart will then fail with the bad config.
- Always check error log after config changes to confirm hot reload succeeded.

## Configured Providers
- OpenAI Codex (primary model: `gpt-5.3-codex`, OAuth auth)
- MiniMax (API key, also portal with OAuth)
- Anthropic (`claude-sonnet-4-6`, API key)
- OpenRouter (Qwen 3.5, API key via env var)

## Channels
- Telegram: `@Joe_IK_bot`
- Discord: `@Joe_IK_Bot` (guild `1474471790947602564`, multiple channels allowlisted)

## Known Warnings (non-critical)
- **Telegram groupPolicy warning**: `channels.telegram.groupPolicy` is `"allowlist"` but no `groupAllowFrom` IDs are configured — all Telegram group messages are silently dropped. DMs still work. Fix by adding sender IDs to `channels.telegram.groupAllowFrom` or changing `groupPolicy` to `"open"`.

## Docker Status
- Docker is **NOT installed** on this machine. Sandbox mode must stay `off`.
