---
name: backup-memory
description: Persist Codex project state into SESSION_MEMORY.md. Use when the user asks to save, back up, checkpoint, or update session memory at the end of work, preserving gotchas and unresolved blockers while verifying repository drift.
---

# backup-memory

The rules for this skill are the Claude Code command `commands/BackupMemory.md` in the
session-memory-commands repository. One text, two entry points: Claude Code runs
it as `/BackupMemory`, Codex runs it through this file. Nothing is restated here, so a
correction to the command reaches Codex without a second edit.

Find the text, in this order, and apply it exactly as written, substituting
`$` skill names for the `/` command names it mentions:

1. `rules.md` beside this file — the installer copies `commands/BackupMemory.md` there.
2. The newest `~/.claude/plugins/cache/session-memory-commands/session-memory/*/commands/BackupMemory.md`.
3. A checkout of the repository, `commands/BackupMemory.md`.

If none is found, say so and stop; do not work from memory of what the command
says.
