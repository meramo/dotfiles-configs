---
name: chezmoi dotfiles
description: User manages dotfiles and Claude Code config across multiple computers using chezmoi
type: user
---

Uses chezmoi to manage configuration settings and dotfiles across multiple computers.

Claude Code config is also managed via chezmoi:
- `~/.claude/settings.json` — global settings (synced, templated for API keys)
- `~/.claude/CLAUDE.md` — global instructions (synced)
- `~/.claude/shared-memory/` — memory files (synced), symlinked from `~/.claude/projects/<encoded-home>/memory`
- `~/.claude/settings.local.json` — machine-specific permissions (NOT synced)
- A `run_onchange_after` template script creates the symlink, adapting the path per machine
- After Claude writes new memory files, run `chezmoi re-add ~/.claude/shared-memory` to pick them up for syncing
- Template variables (e.g., `nimbleApiKey`) are set in `~/.config/chezmoi/chezmoi.toml`
