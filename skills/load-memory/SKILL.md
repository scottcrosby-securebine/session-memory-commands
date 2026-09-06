---
name: load-memory
description: Warm-resume a Codex session from repository session memory. Use when the user asks to load, restore, resume, or hydrate SESSION_MEMORY.md state without a full codebase tour.
---

# load-memory

The rules for this skill are the Claude Code command `commands/LoadMemory.md` in the
session-memory-commands repository. One text, two entry points: Claude Code runs
it as `/LoadMemory`, Codex runs it through this file. Nothing is restated here, so a
correction to the command reaches Codex without a second edit.

Find the text, in this order, and apply it exactly as written, substituting
`$` skill names for the `/` command names it mentions:

1. `rules.md` beside this file — the installer copies `commands/LoadMemory.md` there.
2. The newest `~/.claude/plugins/cache/session-memory-commands/session-memory/*/commands/LoadMemory.md`.
3. A checkout of the repository, `commands/LoadMemory.md`.

If none is found, say so and stop; do not work from memory of what the command
says.
