---
name: No source command suggestions
description: Never suggest or ask the user to run "source" commands after editing shell config files
type: feedback
---

Do not suggest running `source ~/.zshrc`, `source ~/.bashrc`, or any `source` command after modifying shell configuration files.

**Why:** The user finds it annoying and unnecessary — they know to source or open a new shell when needed.

**How to apply:** After editing any shell config file (.zshrc, .bashrc, .profile, etc.), just make the edit and move on. Do not remind the user to source it.
