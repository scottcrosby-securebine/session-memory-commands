---
name: session-primer
description: Cold-start a Codex session in an unfamiliar repository. Use when the user asks to initialize, orient, prime, or explain a project while restoring SESSION_MEMORY.md state and checking it for drift.
---

# session-primer

The rules for this skill are the Claude Code command `commands/primer.md` in the
session-memory-commands repository. One text, two entry points: Claude Code runs
it as `/primer`, Codex runs it through this file. Nothing is restated here, so a
correction to the command reaches Codex without a second edit.

Find the text, in this order, and apply it exactly as written, substituting
`$` skill names for the `/` command names it mentions:

1. `rules.md` beside this file — the installer copies `commands/primer.md` there.
2. The newest `~/.claude/plugins/cache/session-memory-commands/session-memory/*/commands/primer.md`.
3. A checkout of the repository, `commands/primer.md`.

If none is found, say so and stop; do not work from memory of what the command
says.
