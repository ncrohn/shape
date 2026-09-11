# Recon

Your job is terrain, not direction. The user decides where to go; you tell them
what is actually there.

## The bar

Every line in the recon section carries a path. A claim without a path is not
recon — it is a guess, and it belongs in the unknowns list instead.

Label anything you did not read directly:
- no label — you read the file this turn
- `INFERENCE:` — supported indirectly, not confirmed
- `UNKNOWN:` — you tried and could not settle it

Do not label the obvious, and do not add verification passes just to earn a
label.

## What to go find

Six questions. Answer the ones the change touches; skip the rest.

1. **What exists now.** The types, tables, events, endpoints, and flags already
   in the path of this change. Names and paths.
2. **Where the seams are.** The boundaries the change has to cross — service,
   package, module, deployable. Crossing one is usually where a phase splits.
3. **Who reads and who writes.** Every caller of the thing being changed. This
   is the blast radius, and the user cannot guess it.
4. **What the data looks like.** Row counts, null rates, distinct values of the
   field in question. Query it if you can reach it. A backfill decision made on
   a guessed row count is a bad decision.
5. **Prior art in-repo.** Has someone solved this shape already? Cite the file,
   not a description of it.
6. **The forks.** Places where the code genuinely admits more than one path and
   cannot tell you which. These become scaffold sections.

## How to run it

Delegate breadth, read depth yourself.

- Fan out read-only helpers in parallel if your agent has them (Claude Code
  subagents, Codex spawned agents), one per question that needs sweeping. Give
  each a narrow brief and tell it to return paths with one-line facts. Without
  helpers, sweep with grep yourself and keep it to the questions that matter.
- Then read the three to six files that actually matter, yourself. Do not write
  recon from a helper's summary — helpers are wrong often enough to matter,
  and a wrong fact in the scaffold sends the whole design sideways.
- Live data beats a fixture. Say which one you read. A value from a test file or
  a config store is not the running system.

Cap it. Recon is a means, not the deliverable. If it runs past roughly a dozen
tool calls without changing what the scaffold will ask, stop and write.

## What recon is not

- Not a recommendation. "We should write it at finalize" is not a fact.
- Not a risk assessment. Save it for the loop, as an objection.
- Not a menu. Do not lay out options with tradeoffs. Name the fork; let them
  pick.
- Not exhaustive. Nine services doing the same thing is one sentence and one
  example, not nine bullets.
