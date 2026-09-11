# Approval and handoff

Phases are approved. Ask how they want it built — the answer differs per
feature, so do not assume and do not default.

## The question

Ask one question, titled **Build it**, with these three options. Use a
structured question tool if your agent has one; otherwise a numbered list, then
stop and wait.

1. **Fresh session per phase** — you print one `execute` command per phase and
   stop. Nothing fires. Each session runs the drift check before it builds and
   writes the log after it merges.
2. **This session, phase by phase** — you implement phase 1 now with the design
   still in context. They review, then phase 2. The log still gets written
   after each merge; context in your head is not context on disk.
3. **Their own runner** — packets are plain markdown. Print the absolute paths
   and let them feed them to whatever they use. Tell them once: the packet's
   `## Context` expects the log to be kept, and `shape execute` will keep it
   for them if they run that instead.

Say one sentence, and only when it applies: a non-reversible phase argues
against dispatching it unattended. No other recommendation. They pick.

## Rules that are not negotiable

- **Sequential.** Never start phase N before N-1 has merged. A dependent phase
  branched off the default head before its parent lands is building against a
  tree that does not exist.
- **One phase per job.** The packet granularity is the job granularity.
- **The agent never reviews its own diff.** Whoever wrote the code does not
  decide whether it is right.
- **Two correction rounds.** If a third is needed, the packet was wrong —
  rewrite the packet and start fresh rather than patching a confused agent.
- **The log is written after every merge.** Whatever runs the phase, the entry
  in `index.md` is how the next phase learns what this one actually did. A
  live session is not a substitute — it gets closed, and the next implementer
  cannot reach it.

## Fresh session

One copyable line per phase, nothing else. The packet already says the rest.
Print the form for the agent you are running in; if you cannot tell, print both.

```
claude "/shape execute <slug> 1"
codex  "\$shape execute <slug> 1"
```

The raw packet still works — `claude "execute ~/.shape/plans/<slug>/phases/01-…md"`
— but skips the drift check and the log. Print it only for option 3.

## Report

Four lines, maximum:

- design path
- phase count and index path
- what was started, or the command to start phase 1
- anything left undone

Then stop. No next-steps list, no offer of more work.

## Archive

When the last phase is logged merged, `shape archive <slug>` moves the folder to
`$SHAPE_PLANS_DIR/archived/<slug>/`. Nothing is deleted — `.shape/transcript.md`
is the record of why the design is what it is.
