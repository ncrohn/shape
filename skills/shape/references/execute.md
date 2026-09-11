# Executing a phase

This runs in a fresh session, and that is the point. The session has no memory
of the loop, so everything the phase needs is read from disk. The record on
disk — `design.md`, `.shape/transcript.md`, and the log in `index.md` — is the
oracle. Never make a live "shaper" session the oracle; it falls asleep, it gets
closed, and the third phase's implementer cannot reach it anyway.

Identical in Claude Code and Codex. The runner only changes who types the
command.

## Pick the phase

1. Resolve the slug: the argument, or the one unarchived shape whose
   `state.json` is at `stage: "execute"`. Ambiguous → `list` and ask.
2. Read `index.md`. The next phase is the first row with no entry under
   `## Log`. An explicit `<n>` overrides.
3. Phase N runs only when the log says N-1 **merged**. Not "done", not "in
   review" — merged. Otherwise say which phase is blocking and stop. A
   dependent phase branched before its parent merges is building against a
   tree that does not exist.

## The drift check — before any code

Three things can have gone stale since the packet was written, and a cold
agent cannot tell from the packet alone. Check all three.

1. **Packet vs design.** Read `design.md` end to end — the whole document,
   not the sections the packet cites. A `## Decided` line the design no longer
   says is stale.
2. **Packet vs what shipped.** Read every earlier phase's log entry. An earlier
   phase that deviated from its packet — moved a check into a different layer,
   dropped a field, renamed something — leaves this packet assuming the plan
   instead of the code. This is the drift that a live session used to catch by
   remembering; the log is what replaces the memory.
3. **Packet vs code.** Re-verify every path and line in `## Files`. Merges
   since the packet was written move lines.

Anything stale: update the packet first, show the change in a few lines, and
wait. Do not start building on a stale packet and fix it in review — that is
the loop the log exists to break.

A `## Decided` line that looks wrong is not drift. It is an objection: raise it
to the user with the failure scenario and the evidence, and let them rule. Do
not rewrite a decision.

## The work

Same rules as handoff, restated because the session is cold:

- Whoever writes the code does not review it. If this session implements, the
  review is theirs or a separate agent's, from the packet's **Report back** and
  the working-tree diff.
- Two correction rounds. A third means the packet was wrong — rewrite it and
  start fresh, do not keep patching a confused agent.
- The packet's **Rules** hold: no commit, push, branch switch, or hook bypass
  unless the user says so.

## After the merge — write the log

Append to `## Log` in `index.md`:

```markdown
### 2 — Reads switch to the new field
Merged: PR #1234, 2026-09-11
Shipped as planned, except: the flag check moved into the resolver (packet
said the handler) — the handler has no request context.
Reopened: none.
Next phase affected: phase 3's Files still points at the handler for the
flag; updated.
```

Four facts: where it merged, how it deviated from the packet, whether any
`## Decided` item was reopened and how the user ruled, and what it changes for
the phases still to come. Facts only. "Shipped as planned" with nothing after it
is a complete entry.

Then re-read every remaining packet against this entry. A deviation that
changes what a later packet assumes gets fixed in that packet now, with the
user, and named in the log entry. This is how phase N inherits the context
phase N-1 had.

Update `state.json`. When the last phase is logged merged, say so and offer
`archive`.

## Do not

- Run phase N while N-1 is unmerged.
- Rewrite `design.md` silently. A design change goes through the user, and it
  gets a transcript line.
- Skip the log because the phase shipped cleanly. The next phase's drift check
  reads it either way; an empty log reads as "nobody checked".
- Ask the user to remember what an earlier phase did. That is what the log is
  for.
