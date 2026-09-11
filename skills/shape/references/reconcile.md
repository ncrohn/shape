# Reconcile

The loop reads one section at a time on purpose, and that is exactly how the
document ends up disagreeing with itself. An answer at §6 quietly changes what
§2 settled, and nobody re-reads §2. Carve phases from that and the packets
inherit the contradiction — a cold agent then builds one half of the design
against the other half, and the review loop finds "critical" issues that were
never code bugs at all.

This stage reads the whole thing once, as one document, before anything is
carved from it.

## When

- After the last loop section, always. Not optional, not on request.
- On `--reconcile`, any time.
- Before stage 5 carves, if `design.md` has changed since `reconciled_at` in
  `state.json`. Carving from an unreconciled design is refused; run this first.

## The read

Read `design.md` end to end, in one pass, plus `.shape/transcript.md`. Then
look for exactly these:

1. **Contradiction.** Two sections that cannot both be true. §2 says stamp at
   finalize; §5's rollout backfills from the presign row.
2. **Unwritten decision.** Something settled in the transcript that never made
   it into the section it belongs to. The loop wrote it down at the turn; check
   that it is in the right section, not only the transcript.
3. **Orphaned assumption.** A Phases or Rollout line that depends on something
   a later section cut, moved out of scope, or overruled.
4. **Blind spot.** An `UNKNOWN:` from recon that a phase now depends on and
   that nobody settled. Not every unknown — the ones a packet would have to
   guess at.

Nothing else. This is not a second loop and not a review of their design. Taste
is theirs; a section you would have written differently is not a finding.

## The report

```
── Reconcile · 3 findings ──

1. §2 vs §5 — §2 stamps at finalize; §5 backfills from the presign row,
   which has no finalize timestamp to stamp from.
2. §4 — the transcript settled "null for existing rows, never guessed"
   at turn 4; §4 still says TODO for that case.
3. §7 phase 3 assumes the admin dashboard reads the new field; §8 put the
   dashboard out of scope.

> _
```

Each finding names the sections and states the disagreement in one or two
lines. No fix proposed — that is theirs. Cap at six; more than six means the
loop was not writing things down and you should say that instead.

If there are none, say so in one line and move on. Do not invent one.

## Fixing

One prompt for the whole list, not one per finding. They answer; you write the
settled text into the sections named, in their words, and append the exchange
to `.shape/transcript.md`. A fix that opens a new question goes back through
the loop as `--only <n>`, not through here.

Then write `reconciled_at` (ISO timestamp) into `state.json`, and ask whether to
carve phases. Do not carve unprompted.
