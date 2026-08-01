# session-memory

Three Claude Code slash commands for **session continuity** — carrying state
across agent sessions without dragging the transcript along.

| Command | When | What it does |
|---|---|---|
| `/primer` | Cold start | Orients an agent that has never seen the repo, quotes the prior session's kickoff, reports drift, explains the project |
| `/LoadMemory` | Warm resume | State only, no codebase tour. Reads the memory file, hydrates issues, reports drift |
| `/BackupMemory` | Session end | Updates `SESSION_MEMORY.md` — merging, not overwriting — then verifies and commits it |
| `/handoff-doc` | Mid-task cutoff | Compacts *this conversation* into a scratch document a fresh agent can resume from |

The first three carry **project state**. `/handoff-doc` carries **conversation
state** — different lifetimes, and useful together when a long session ends
mid-task: the handoff holds the reasoning, `SESSION_MEMORY.md` holds the state.

## The contract

All three revolve around one file, `SESSION_MEMORY.md`, and one section:

```markdown
## Next Session Kickoff
```

`/BackupMemory` must write it. `/primer` and `/LoadMemory` quote it by name and
**report its absence as a lost handoff**. That single rule is the point of the
set — a handoff that silently vanishes is worse than no handoff, because nobody
goes looking for it.

## Why these exist

They were rebuilt after a real failure: a template that did not emit the section
its readers grepped for, so following the documented process destroyed the
handoff with no error. Hardening them surfaced a family of the same defect —
checks that looked like checks but could not fail.

Every instruction here was verified by running it, not by reasoning about it:

- **Drift is measured, not asserted.** Memory rots between sessions, so each
  claim is checked against the repo — branch, tree, PRs, issue state, CI — and
  mismatches are named specifically ("memory says `4b172f9b`, HEAD is
  `8b36d9f7`"), never as "state has drifted".
- **`/BackupMemory` merges.** It carries forward every Gotcha and every
  unresolved blocker it cannot disprove with a command. An earlier version
  rewrote the file, and because the warm-resume path could trigger it, a resync
  silently deleted the highest-value content and then reported success.
- **Unshipped work is the headline.** Uncommitted, unpushed, merged-but-
  undeployed and deployed-but-unverified are tracked as distinct states.
- **The verify block exits 0 when it passes.** Sounds trivial; the original
  reported failure on every clean tree, because `grep -v` exits 1 on empty input.

## Repo-agnostic by construction

Nothing about a particular project is baked in. At runtime the commands
discover:

- the source layout (never assuming `src/`)
- the build/publish workflow by name, since `gh run list --limit 5` can miss it
  entirely in a repo with many workflows
- whether `tree` exists before reaching for it
- the memory filename, before verifying or committing it

They also distinguish failure shapes that look alike: `gh` erroring (no remote,
unauthenticated) is not the same fact as `gh workflow list` succeeding but
returning nothing (no CI), and an empty `conclusion` means a run is still going —
not that it failed.

Tracker cells are treated carefully too. Only a bare `#N` is hydrated; a
placeholder, several ids in one cell, a legacy tracker id, or a **cross-repo**
reference like `otherrepo #492` is skipped and named — that last one would
otherwise resolve against the wrong repository and return an unrelated issue.

## Install

```
/plugin marketplace add scottcrosby-securebine/session-memory-commands
/plugin install session-memory@session-memory-commands
```

## Optional companion: the `handoff` skill

**Not required.** All three commands work fully on their own, and nothing here
fails without it.

`/BackupMemory` will use a `handoff` skill if one is installed, applying its
*content* discipline — reference rather than restate, name the skills the next
session should invoke, redact secrets — to the Resume and Kickoff sections. It
points at the skill rather than copying it, so improvements are inherited. Only
the content rules transfer: a handoff skill that writes to a temp directory does
not override this command's committed file.

When no handoff skill is found, `/BackupMemory` says so once and continues,
rather than skipping in silence. The one it looks for:

| | |
|---|---|
| Skill | `handoff` — Matt Pocock's |
| Source | [`mattpocock/skills`](https://github.com/mattpocock/skills) |
| Path | `skills/productivity/handoff/SKILL.md` |
| Looked for at | `~/.claude/skills/handoff/SKILL.md`, then `.claude/skills/handoff/SKILL.md` |

Note it ships `disable-model-invocation: true`, so it is user-invoked only —
`/BackupMemory` **reads** the file for its rules rather than invoking the skill.
That also means it works when installed globally, without per-repo setup.

### Why `/handoff-doc` exists alongside it

`disable-model-invocation: true` means an agent cannot fire the skill on its own;
only you can, by typing `/handoff`. `/handoff-doc` is a thin model-invocable
wrapper that **reads** those rules and applies them, so you can just ask for a
handoff document in conversation.

It deliberately does not fork the skill. When the skill is present, its rules
win — including where to save the file — so upstream improvements are inherited.
When it is absent, the wrapper falls back to a short generic spec and says which
skill would sharpen it. Matt's skill is never modified, and a stale lockfile pin
or a skill update cannot break this plugin.

## Project overrides

A repo with strong conventions can drop its own copy in `.claude/commands/`,
tightening the generic instructions with real paths and real workflow names.
Keep the shared contract — especially the kickoff section — or the readers in
this plugin will correctly report the handoff as lost.

## License

MIT
