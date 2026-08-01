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
instruction does not apply. Skip this step only when neither path exists.

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
[What to pick up, in priority order, and any skill to invoke. MANDATORY —
/primer and /LoadMemory read this section by name and report its absence as a
lost handoff.]
```

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

Then land it. A handoff left uncommitted is a handoff lost:

```bash
git commit -m "docs(session): <what changed>" -- SESSION_MEMORY.md
git pull --rebase && git push
git status -sb               # no [ahead N] means the push landed
```

Commit by pathspec, not `git add` + bare `git commit`. A bare commit ships the
whole index — including anything another step left staged — and a docs-flavoured
message on a non-docs change misrepresents the commit. Leave unrelated staged
work for its own commit and say it is still pending.

Follow the repo's own branching norm (AGENTS.md / CLAUDE.md, which are worth a
read here — AGENTS.md is not auto-loaded): some repos require a branch before
committing to the default branch, with a documented exemption for docs-only
changes. If the norm is unstated, committing the memory update directly is the
common case — but say what you did.

If the push fails on a conflict, resolve and retry until it succeeds. If the repo
has **no remote or no upstream branch**, committing locally is the terminal step —
say the handoff is committed but unpushed, and stop rather than looping.
