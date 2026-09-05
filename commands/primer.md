# Primer — cold-start orientation

Produce a situation report for an agent that has never seen this repo, then
explain the project back to the user.

Need state only, not a codebase tour? `/LoadMemory` is the cheap path — say so
and stop.

CLAUDE.md is already in your context; the harness loads it every session, so
never spend a read on it. **AGENTS.md is not** — it is not auto-loaded unless
something `@`-imports it, and it is usually where issue-tracking and
session-completion norms live. Read it if present.

## 1. Recon

Discover the project's shape — never assume `src/`. Batch these, then read what
the listing reveals:

- `ls` the root, then `ls` the two or three real source directories it shows
- `command -v tree` before reaching for a tree; on many hosts it is absent
- `git status -sb` and `git log --oneline -5`
- `gh issue list --state open --limit 200 --json number --jq 'length'` for the
  count. Only pull titles for the handful the kickoff names — some repos carry
  thousands of open issues and dumping them buys nothing.
- `gh pr list --state open`
- `gh workflow list`, then pick the build/publish workflow and
  `gh run list --workflow "<name>" --limit 3 --json headSha,conclusion`

Then read, in this order:

- `README.md` if it exists, and any `docs/` index — the root README is usually
  the product, the docs index the engineering map
- the session-memory file (commonly `SESSION_MEMORY.md`), if the root listing
  showed one

### When recon can't answer

- **`gh` errors** (no remote, unauthenticated): say so once, continue git-only.
- **`gh workflow list` succeeds but is empty**: the repo has no CI. Say that
  rather than reporting the build as not-green.
- **Several plausible build workflows** (large repos often have a dozen): name
  the one you picked and mark the verdict as based on your pick, or say it is
  ambiguous. Never let a guess become a silent "built ✓".
- **A run is still in progress**: `conclusion` comes back empty. Report
  "running", never "not green".

## 2. Surface the kickoff first

If the memory file has a `## Next Session Kickoff` section, quote it verbatim at
the very top of your reply, ahead of everything else. It is a live instruction
from the prior session, not history.

Then act on its first line as `/LoadMemory` §1b says: read the handoff
it names by path, and invoke the doctrine skill before anything after this step
when the state is open. That step is written there once and not restated here;
what §1b would put in its Handoff line goes in the drift table's handoff row.

If the file exists but that section is absent, say so plainly — `/BackupMemory`
is contracted to write it, so its absence means the last handoff was lost. If no
memory file exists at all, note that and offer `/BackupMemory` at session end.

## 3. Report drift

**Drift** is any gap between what the memory file claims and what the repo
actually shows. Memory is written by a past session and rots between runs, so
trust the repo and name every mismatch specifically: "memory says `4b172f9b`,
HEAD is `8b36d9f7`" beats "state has drifted."

| Memory claims | Verify against |
|---|---|
| Branch and commit | `git status -sb`, `git log -1` |
| Clean tree | `git status --short \| grep -v '^??'` — tracked only; untracked tooling dirs inflate a raw count and fake drift |
| PR state | `gh pr list --state open` |
| A specific issue is open | `gh issue view <N> --json state`, one at a time, and only for bare `#N` — see the skip rule in `/LoadMemory` §3. Never infer closure from absence in a list |
| "built" | the workflow run above, matched to HEAD's sha |
| "deployed" | a green build usually means artifacts were published, not deployed. Treat as unverified unless the repo documents a deploy check and you run it |
| A handoff path in the kickoff | `test -f` it. A named file that is missing is drift, never silence |

Silence on a row means you checked it and it held.

Found drift? Offer to resync: "Memory is stale — run `/BackupMemory`?" Leaving it
stale means the next session inherits the same wrong file.

## 4. Explain the project

- Structure — the real directories you listed, never a guessed layout
- Purpose and goals
- Key files and what each is for
- Important dependencies
- Important configuration files
- Open work — the issue count from §1, and whatever the kickoff points at

## Done when

The kickoff is quoted or its absence flagged, every drift row is checked, and
every §4 bullet is covered.
