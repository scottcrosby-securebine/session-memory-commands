# session-memory

Four session-continuity workflows for **Claude Code and Codex** — carrying state
across agent sessions without dragging the transcript along. Claude Code uses
slash commands in `commands/`; Codex uses standalone skills in `skills/`.

| Claude Code | Codex | When | What it does |
|---|---|---|---|
| `/primer` | `$session-primer` | Cold start | Orients a new agent, quotes the prior kickoff, reports drift, explains the project |
| `/LoadMemory` | `$load-memory` | Warm resume | Reads saved state, hydrates issues, reports drift without a codebase tour |
| `/BackupMemory` | `$backup-memory` | Session end | Merges and verifies `SESSION_MEMORY.md`, writes the handoff document when asked, commits, never pushes |
| `/handoff-doc` | `$handoff-doc` | Mid-task cutoff | Compacts this conversation into a temporary document a fresh agent can resume from |

The first three carry **project state**. `/handoff-doc` carries **conversation
state** — different lifetimes, and useful together when a long session ends
mid-task: the handoff holds the reasoning, `SESSION_MEMORY.md` holds the state.

## The contract

The first three workflows in both tools revolve around one file,
`SESSION_MEMORY.md`, and one section:

```markdown
## Next Session Kickoff
```

Both backup variants must write it. The primer and load variants quote it by name and
**report its absence as a lost handoff**. That single rule is the point of the
set — a handoff that silently vanishes is worse than no handoff, because nobody
goes looking for it. Codex redacts sensitive values before quoting and marks
those redactions. Both tools can read the same memory file without conversion.

## Why these exist

They were rebuilt after a real failure: a template that did not emit the section
its readers grepped for, so following the documented process destroyed the
handoff with no error. Hardening them surfaced a family of the same defect —
checks that looked like checks but could not fail.

The original Claude Code commands were hardened against these failures:

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

### Claude Code

```
/plugin marketplace add scottcrosby-securebine/session-memory-commands
/plugin install session-memory@session-memory-commands
```

### Codex

From a local checkout, copy the four skill directories into your user skills
directory. These commands are for Bash on Linux or macOS:

```bash
cd /path/to/session-memory-commands
mkdir -p "$HOME/.agents/skills"
for skill in backup-memory load-memory session-primer handoff-doc; do
  if [ -e "$HOME/.agents/skills/$skill" ] || [ -L "$HOME/.agents/skills/$skill" ]; then
    echo "Already installed; review before replacing: $skill"
  else
    cp -R "skills/$skill" "$HOME/.agents/skills/$skill"
  fi
done
# Each skill is a pointer to its command file; copy the rules beside it.
cp commands/BackupMemory.md "$HOME/.agents/skills/backup-memory/rules.md"
cp commands/LoadMemory.md   "$HOME/.agents/skills/load-memory/rules.md"
cp commands/primer.md       "$HOME/.agents/skills/session-primer/rules.md"
cp commands/handoff-doc.md  "$HOME/.agents/skills/handoff-doc/rules.md"
```

For one project, copy those directories into that project's `.agents/skills/`
instead. Each skill is self-contained; copying `commands/` or installing the
Claude Code marketplace plugin is not required for Codex.

In Codex CLI or the IDE extension, use `/skills` to find them or type the `$`
names in the table above. Natural-language requests such as “use load-memory”
also work. If newly installed skills do not appear, restart Codex. These paths
and invocation forms follow the [official Codex skill documentation](https://learn.chatgpt.com/docs/build-skills).

If you already have these skills under `~/.codex/skills` or another configured
location, update that installation instead of adding a second copy. The copy
instructions above do not update existing installations automatically.

### Usage and commit behavior

Start an unfamiliar project with `$session-primer`, resume known work with
`$load-memory`, and save at session end with `$backup-memory`. Use
`$handoff-doc` when another agent needs the current conversation's reasoning.

`/BackupMemory` and `$backup-memory` run the same text. Both write the memory
file and, when asked for a handoff document, a dated file under
`docs/handoffs/`; both commit what they wrote by pathspec and never push. A
backup is taken mid-thought, and pushing publishes, so the push waits for you to
say so. The next session's `/LoadMemory` or `$load-memory` reads the handoff by
the path the kickoff's first line names.

Both versions preserve gotchas and unresolved blockers and distinguish merged,
deployed, and verified work. Git supplies repository state; GitHub checks use
an authenticated `gh` CLI when available. Unavailable GitHub evidence is
reported rather than treated as a successful check.

## Optional companion: the `handoff` skill

**Not required.** All four workflows work on their own, and nothing here
fails without it.

The Codex skills run the same command text as Claude Code, so they look for
Matt Pocock's `handoff` where `/BackupMemory` does, with the same fallback.
The lookup paths and wrapper behavior below apply to both.

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

For Codex, put project conventions in `AGENTS.md`, or install customized skill
copies in the project's `.agents/skills/`. Avoid keeping multiple installed
copies with the same skill name; Codex does not merge their instructions.

## Repository layout

- `commands/`: Claude Code command instructions.
- `.claude-plugin/`: Claude Code plugin and marketplace metadata.
- `skills/<name>/SKILL.md`: Codex skill metadata and instructions.

When changing the memory format or preservation rules, review both the command
and corresponding skill so either tool can resume the other's saved state.

## License

MIT
