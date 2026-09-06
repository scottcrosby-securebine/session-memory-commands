---
name: handoff-doc
description: Create an ephemeral conversation handoff for another Codex agent. Use when the user asks to compact the current thread, prepare a mid-task handoff, or preserve reasoning for a fresh session without modifying the repository.
---

# handoff-doc

The rules for this skill are the Claude Code command `commands/handoff-doc.md` in the
session-memory-commands repository. One text, two entry points: Claude Code runs
it as `/handoff-doc`, Codex runs it through this file. Nothing is restated here, so a
correction to the command reaches Codex without a second edit.

Find the text, in this order, and apply it exactly as written, substituting
`$` skill names for the `/` command names it mentions:

1. `rules.md` beside this file — the installer copies `commands/handoff-doc.md` there.
2. The newest `~/.claude/plugins/cache/session-memory-commands/session-memory/*/commands/handoff-doc.md`.
3. A checkout of the repository, `commands/handoff-doc.md`.

If none is found, say so and stop; do not work from memory of what the command
says.
