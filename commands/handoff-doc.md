# handoff-doc — compact this conversation for a fresh agent

Write a handoff document so another agent can pick up where this session left
off, then report its path.

This is a **thin wrapper**, not a reimplementation. Matt Pocock's `handoff` skill
owns the rules; it ships `disable-model-invocation: true`, so it can be read but
never invoked. This command reads it and applies it — so its improvements land
here without a fork, and the skill itself stays untouched.

## 1. Find the rules

Check in order, and read the first that exists:

- `~/.claude/skills/handoff/SKILL.md`
- `.claude/skills/handoff/SKILL.md`

**Follow it exactly**, including wherever it says to save the file. Do not
restate its rules, and do not second-guess them — deferring to it is the whole
point of this command.

## 2. When it is not installed

Say so once, then write the document anyway:

> No `handoff` skill installed — using the built-in fallback. For the full
> version: `mattpocock/skills` → `skills/productivity/handoff/SKILL.md`.

Falling back, a handoff document is worth reading only if it carries:

- **Where the work stands** — the live direction and open reasoning, not
  decisions already settled
- **What to do next**, in priority order
- **Which skills** the next agent should invoke
- **Pointers, not copies** — reference specs, plans, issues, commits and diffs by
  path, number or sha. Anything already captured elsewhere gets linked, never
  pasted; that is what keeps a handoff short enough to be read
- **Nothing sensitive** — no keys, tokens, or personal data

Save it outside the workspace, in the OS temp directory. A conversation handoff
is scratch: it should not turn up in `git status`.

## 3. Arguments

Anything passed to this command describes what the next session will focus on.
Tailor the document to it — a handoff aimed at a known next task is worth far
more than a general summary.

## Not a substitute for /BackupMemory

Different artifacts with different lifetimes:

| | `/handoff-doc` | `/BackupMemory` |
|---|---|---|
| Scope | this conversation | the project |
| Lifetime | ephemeral, temp dir | durable, committed |
| Read by | the next agent, once | every future session |

When a long session ends mid-task, both earn their keep: the handoff carries the
reasoning, `SESSION_MEMORY.md` carries the state.

A handoff document the next session resumes the *project* from — the one the
kickoff names and `/LoadMemory` reads by path — is `/BackupMemory` §3b, not this
command. This one carries only the conversation's reasoning.

## Done when

The document is written, its path is reported, and anything sensitive is redacted.
