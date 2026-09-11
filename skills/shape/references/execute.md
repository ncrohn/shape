# Executing a phase

This runs in a fresh session, and that is the point. The session has no memory
of the loop, so everything the phase needs is read from disk. The record on
disk — `design.md`, `.shape/transcript.md`, and the log in `index.md` — is the
oracle. Never make a live "shaper" session the oracle; it falls asleep, it gets
closed, and the third phase's implementer cannot reach it anyway.

**One phase per run.** Build it, write the log, stop. Never continue into the
next phase, however clean this one was. Whether the next phase should start,
and how, is a decision the user makes with the log in front of them — not a
loop this skill runs.

Identical in Claude Code and Codex. The runner only changes who types the
command.

## Pick the phase

1. Resolve the slug: the argument, or the one unarchived shape whose
   `state.json` is at `stage: "execute"`. Ambiguous → `list` and ask.
2. Read `index.md`. The next phase is the first row with no entry under
   `## Log`. An explicit `<n>` overrides.
3. Check its **Sequencing** cell against the parent's log entry:
   - `stack on N-1` — the parent must be logged **built** (branch pushed, PR
     open).
   - `after N-1 merges` / `alone on main` — the parent must be logged
     **merged**. Not "done", not "in review" — merged.
   - Empty — the sequencing decision was never made. Ask it now, with the
     facts from the index row, and write the answer into the cell before
     going on. Do not pick.

   Blocked → say which phase is blocking and why, and stop.

## Is this still the right move?

Before touching anything, one short paragraph to the user: what this phase
ships, what it is sequenced on, and anything in the log or the tree that has
changed since that was decided — the parent shipped differently, main moved
under the stack, a PR is still unreviewed. Then wait for a go. Sequencing was
decided at handoff with the facts available then; this is the moment to notice
the facts moved.

Do not reopen the decision without a fact. "Are you sure?" is not a fact.

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
3. **Packet vs code.** Re-verify every path and line in `## Files`, on the
   branch the phase will actually build on — the parent's branch if stacked,
   the default branch if not. Merges since the packet was written move lines.

Anything stale: update the packet first, show the change in a few lines, and
wait. Do not start building on a stale packet and fix it in review — that is
the loop the log exists to break.

A `## Decided` line that looks wrong is not drift. It is an objection: raise it
to the user with the failure scenario and the evidence, and let them rule. Do
not rewrite a decision.

## Branch setup

Only after the go, and only what the Sequencing cell needs.

- `stack on N-1`: fetch, branch `<slug>/<nn>-<name>` from `origin/<parent
  branch>`. Not from the parent's local branch — the parent may have been
  pushed to since.
- `after N-1 merges` / `alone on main`: fetch, branch from the default branch.
- Update the packet's `## Context` branch line if it differs from what was
  written at carve time.

Nothing is committed yet. The packet agent works in this tree and leaves its
changes uncommitted, per the packet's **Rules**.

## The work

Same rules as handoff, restated because the session is cold:

- Whoever writes the code does not review it. If this session implements, the
  review is theirs or a separate agent's, from the packet's **Report back** and
  the working-tree diff.
- Two correction rounds. A third means the packet was wrong — rewrite it and
  start fresh, do not keep patching a confused agent.
- The packet's **Rules** hold for the packet agent: no commit, push, branch
  switch, or hook bypass.

## After review — commit, push, open the PR

With the user's go at this phase, and not before: commit on the phase branch,
push, open the PR. Stacked → `--base <parent branch>`. Otherwise the default
branch. Draft unless they say ready.

Say once if it applies: on some repos a draft PR based on a feature branch gets
no CI. If they need CI, the PR has to be marked ready for review. Never retarget
a stacked PR to the default branch to get CI — that flattens the stack into one
cumulative diff, and GitHub may refuse it anyway once it treats the PRs as a
stack.

Write the **built** log entry (below). Stop.

## When the parent of a stacked phase merges

Squash merge puts a commit on main that shares no history with the child. Two
outcomes, and you find out which with one command, not by guessing:

```
gh pr view <child> --json baseRefName,mergeStateStatus
```

- **GitHub rebased the child.** Base is now the default branch, state `CLEAN`,
  one clean commit pushed to the child branch. Do nothing locally except
  `git fetch` and `git reset --hard origin/<child>` — a local merge here is
  redundant, and a push after it is rejected.
- **Phantom conflicts.** Base flipped but state is `CONFLICTING` or `DIRTY`,
  in the parent's own files. Resolve in the child: `git merge origin/<default>`,
  take the child's side for the conflicted files, then **prove** nothing was
  lost before pushing:

  ```
  # every file differing from main must be one this phase actually changed
  for f in $(git diff --name-only origin/<default>...HEAD); do
    git diff --quiet <this-phase's-branch-point> HEAD -- "$f" \
      && echo "LOST: $f — differs from main but untouched here"
  done
  ```

  Anything printed is parent work the resolution reverted. Recover it by
  cherry-picking the parent's missing commits forward. Do not force-push over
  a pushed bad merge.

  The child's commit list will still show the parent's commits after a correct
  resolution. Diffs are content-based; review the diff, ignore the list.

Each merge cascades: fix this child, merge it, the grandchild needs the same
check. One at a time.

## Write the log

Two entries per phase over its life, both under `## Log` in `index.md`.

**Built** — when the PR is open:

```markdown
### 2 — Reads switch to the new field · built
Branch: `upload-workspace-attribution/02-reads-new-field`, PR #1234,
stacked on phase 1, draft, 2026-09-11
Deviations from packet: the flag check moved into the resolver (packet said
the handler) — the handler has no request context.
Reopened: none.
Next phase affected: phase 3's Files still points at the handler for the
flag; updated.
```

**Merged** — when it lands:

```markdown
### 2 — merged
2026-09-14, squash. Child (phase 3): GitHub rebased it, state CLEAN.
```

Facts only. "Shipped as planned" with nothing after it is a complete built
entry. The merged entry names what happened to the child, because the next
run's first question is whether the stack is still sound.

After a built entry, re-read every remaining packet against it. A deviation that
changes what a later packet assumes gets fixed in that packet now, with the
user, and named in the entry. This is how phase N inherits the context phase
N-1 had.

Update `state.json`. When the last phase is logged merged, say so and offer
`archive`.

## Do not

- Build more than one phase in a run.
- Start a phase before its Sequencing cell says it can.
- Commit, push, or open a PR before the user's go at this phase.
- Rewrite `design.md` silently. A design change goes through the user, and it
  gets a transcript line.
- Skip the log because the phase shipped cleanly. The next phase's drift check
  reads it either way; an empty log reads as "nobody checked".
- Ask the user to remember what an earlier phase did. That is what the log is
  for.
