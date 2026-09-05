# BackupMemory — write the session handoff

**Update** the session-memory file; do not compose it from scratch. Read the
existing file first and carry forward everything you cannot disprove. Target
≤400 words.

## 0. Fix the target filename before anything else

`ls` the repo root. The usual name is `SESSION_MEMORY.md`; create that if none
exists. If several candidates exist (a project may keep scoped variants), you own
the unqualified one — never write a scoped variant unless the user names it.

Whatever you settle on, **substitute that filename everywhere below, including
the verify and commit steps.** Those steps spell out `SESSION_MEMORY.md`
literally; leaving them unchanged verifies and commits a file you did not write.

## 1. Gather state

Batch these:

- `git status --short` and `git status -sb` — dirty? ahead of origin? Record the
  **tracked-only** count (`| grep -v '^??'`); untracked tooling dirs inflate the raw one
- `git log --oneline -5`
- `gh issue list --state open --limit 200 --json number --jq 'length'` for the
  count; pull titles only for the few you will name
- `gh pr list --state open`
- `gh workflow list`, then the build/publish workflow's
  `gh run list --workflow "<name>" --limit 3 --json headSha,conclusion`

A green build usually means artifacts were published. It does **not** mean
deployed — check the repo's own deploy documentation before claiming that.

When recon can't answer: `gh` erroring (no remote, unauthenticated) and
`gh workflow list` returning empty (no CI) are different facts — record which,
rather than writing a build state you never observed. An empty `conclusion`
means the run is still going: record "running".

## 2. Carry forward

Before writing, take from the existing file:

- **Every Gotcha still true.** They are the highest-value lines in the file and
  nothing in step 1 can regenerate them. Add this session's; delete only one you
  can show is wrong.
- **Every 🔴 row you cannot disprove with a command.** "Merged but undeployed"
  survives a green build and a merged PR — only an actual deploy clears it.
- **The `Runtime` block**, unchanged, unless you observed the live system this
  session. Copying a stale line forward with its original date is honest;
  asserting freshness you never checked is not.

## 3. Apply the handoff discipline

If a `handoff` skill is installed — check `~/.claude/skills/handoff/SKILL.md`
first, then `.claude/skills/handoff/SKILL.md` — read it and apply its **content**
rules to Resume and Next Session Kickoff: reference specs, plans, issues and
commits by path or number instead of restating them; name the skills the next
session should invoke; redact secrets. Pointing at that skill rather than copying
it means its improvements land here automatically.

Take only the content discipline. That skill writes a scratch file to the OS temp
directory; this command owns a file committed to the repo, so its storage
instruction does not apply.

If neither path exists, write the handoff without it — this step is optional, not
required — but **say so once** rather than skipping in silence:

> No `handoff` skill installed. `/BackupMemory` works without it; installing it
> sharpens Resume and Next Session Kickoff. Source:
> `mattpocock/skills` → `skills/productivity/handoff/SKILL.md`.

Once per session, not once per run.

## 3b. Write the handoff document

When the request asks for a handoff document as well as a backup — the usual
sentence is "a detailed backup and a handoff document so I can clear the
session, then continue in a new session" — write one file at
`docs/handoffs/<YYYY-MM-DD>-<slug>.md`, where `<slug>` names the piece of work.
A new file every time, beside the previous one, never overwriting: when that
name is taken, append `-2`, `-3`. The previous file is not edited.

Header lines above section 1: `supersedes: <previous handoff path>` (or
`supersedes: none`), then whatever header lines the skill that owns the work
requires — a doctrine phase's are given in doctrine step 5. The previous
handoff is the file the kickoff's first line names, else none. When that skill
is not loaded this session, copy the previous handoff's header lines below
`supersedes:` forward, so the record's path stays in the chain, and refresh the
state header and the quoted record state line with its line number from the
record itself; when the record is not on this host, suffix the quoted state
line with "(copied from <previous handoff>; record not found <today>)".

Sections, in this order and no others:

1. **State in one paragraph** — where the work stands, what is live, the
   immediate next step.
2. **Read these, in this order** — each by repo path, never "the newest file in
   a directory".
3. **What is open** — one line per item, each pointing at where the detail
   lives: an issue, the record's line, a spec section.
4. **Do this first** — the single next action.
5. **Gotchas this phase produced** — only ones the memory file you are about
   to write will not carry.
6. **Suggested skills** — which to invoke, and which first.

The 400-word target is the memory file's; this document is as long as the next
session needs. Rules, each one a failure found in a live repo:

- **§5 "Reference, never duplicate" applies here too.** Cite code by symbol,
  not line number.
- **No scratchpad path and no plugin-cache version path**
  (`~/.claude/plugins/cache/<x>/1.53.0/...`): the first dies on a bounce, the
  second on the next version bump, and both were found dead in committed files.
- **Redact by pattern, not judgement**: `://user:pass@`, `PASSWORD=`,
  `user@host` ssh targets, private addresses, pane and process ids. Test
  credentials count; seven committed handoffs carried them.
- **A repo whose own commands own its session store** (per-workstream
  envelopes) keeps them. This file is for repos that use the memory file.
- **Public repo, or a repo an installer's tooling copies whole**: add
  `docs/handoffs/` to `.gitignore` and say so in the report.
  `gh repo view --json visibility --jq .visibility` answers the first, and when
  `gh` errors (no remote, not authenticated) treat the repo as public;
  `.claude-plugin/plugin.json` or `.claude-plugin/marketplace.json` present
  answers the second.

The kickoff's first line then names the file (step 4).

## 4. Write the file

```markdown
# State [descriptive title | YYYY-MM-DD]

## Resume
[Where the work stands → what is live → the immediate next step. Two sentences.]

## Active Work
| Item | Issue | Status | Key Files |
|------|-------|--------|-----------|
[Max 5 rows, but never drop a 🔴 to honor the cap — cut ✅ rows first, then
oldest. One bare `#N` per cell, or — when untracked. A cell holding several ids,
a legacy tracker id, or a cross-repo ref cannot be hydrated by /LoadMemory.]

## Git State
- Branch: `name` @ sha | clean or N dirty | PRs: numbers or none

## Runtime
- [Whatever "live" means here: services, migrations, last deploy + deployed sha.]

## Gotchas (learned this session)
- [Non-obvious discovery that saves the next session real time. Max 3.]

## Next Session Kickoff
handoff: docs/handoffs/<file>.md | state: <open|none>
[What to pick up, in priority order, and any skill to invoke. MANDATORY —
/primer and /LoadMemory read this section by name and report its absence as a
lost handoff.]
```

The kickoff's first line is machine-shaped; `/LoadMemory` §1b says what the
loaders do with it. `state: open` means a doctrine phase is open — its record's
last state line is *Open* or *Blocked* — and `state: none` means anything else,
including ordinary unfinished work. Whenever this session did not derive the
doctrine header lines from the record — no handoff written, or one written by
copying them forward —
carry the previous first line forward (pointing it at the new handoff if you
wrote one), read the record's path from the handoff's header lines, and refresh
the state word from the record's last state line; when no header names a
record, say no record is named; when one is named but not on this host, keep
the previous state word and say the record was not found. When the previous kickoff had no machine-shaped first
line, write `handoff: <the file you wrote this run, or none> | state: none`;
`handoff: none` means no handoff has ever been named.

`## Next Session Kickoff` is required always. Omit `## Gotchas` only when it is
empty *and* the previous file had none, and omit `## Runtime` entirely for a
project with nothing deployable — an empty heading is worse than no heading.

## 5. Rules

**Unshipped work is the headline.** Uncommitted, unpushed, merged-but-undeployed,
or deployed-but-unverified each go in Resume *and* get a 🔴 row. A session that
ends with work stranded must read that way at a glance.

**Merged ≠ deployed ≠ verified.** Separate states. Record the deployed sha.

**Reference, never duplicate.** Architecture and commands live in the repo's own
docs; reasoning lives in the issue; history lives in git. Point at them.

**Redact.** This file is committed. No keys, tokens, or personal data — carry a
pointer to where the secret lives instead of the secret.

**Compact.** Tables over prose. One-liners. Symbols: ✅ ⏳ 🔴.

## 6. Verify, then commit

```bash
root=$(git rev-parse --show-toplevel 2>/dev/null) && cd "$root" || { echo "NOT A GIT REPO — stop here"; exit 1; }
wc -w SESSION_MEMORY.md                              # ≤400
grep -c "^## Next Session Kickoff$" SESSION_MEMORY.md || echo "MISSING — /primer will report a lost handoff"
git status -sb | head -1                             # branch, ahead/behind
git status --short | grep -v '^??' || true           # dirty tracked files; || true keeps a clean tree from exiting 1
```

Reread your Git State line against those last two. The count must match, and a
staged deletion counts as dirty work. Writing "clean" over a dirty tree is the one
error this file cannot survive. Count tracked files only — the `grep -v '^??'` is
load-bearing, since untracked tooling and plan directories can inflate a raw
`git status --short` several-fold.

Then land it locally. Both lines below name only what you wrote or edited this
run: drop any path `git check-ignore -q <path>` matches and any file you did
not touch, and name `.gitignore` only when `git diff -- .gitignore` shows your
line and nothing else, or the file is untracked and holds nothing but your line
— a pathspec commit takes the worktree file whole, so someone else's edit there
would land under your message; leave it and say so.
If nothing remains, say the files are local and ignored, and stop.

```bash
git add -- SESSION_MEMORY.md docs/handoffs/<file>.md .gitignore    # new files: a pathspec commit sees only tracked paths
git commit -m "docs(session): <what changed>" -- SESSION_MEMORY.md docs/handoffs/<file>.md .gitignore
git status -sb               # [ahead N] is expected: nothing here pushes
```

Never push from this command. A backup is taken mid-thought and pushing
publishes; push only when the user says to, and say the commit is local until
then.

Commit by pathspec, never a bare `git commit`. A bare commit ships the
whole index — including anything another step left staged — and a docs-flavoured
message on a non-docs change misrepresents the commit. Leave unrelated staged
work for its own commit and say it is still pending.

Follow the repo's own branching norm (AGENTS.md / CLAUDE.md, which are worth a
read here — AGENTS.md is not auto-loaded): some repos require a branch before
committing to the default branch, with a documented exemption for docs-only
changes. If the norm is unstated, committing the memory update directly is the
common case — but say what you did.

Committing is the terminal step whether or not the repo has a remote: where a
commit was made, say the handoff is committed and unpushed, and stop.
