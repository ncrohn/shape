# Approval and handoff

Phases are approved. Ask how they want it built — the answer differs per
feature, so do not assume and do not default.

## The question

Ask one question, titled **Build it**, with these three options. Use a
structured question tool if your agent has one; otherwise a numbered list, then
stop and wait.

1. **Fresh session per phase** — you print one command per phase and stop.
   Nothing fires.
2. **This session, phase by phase** — you implement phase 1 now with the design
   still in context. They review, then phase 2.
3. **Their own runner** — packets are plain markdown. Print the absolute paths
   and let them feed them to whatever they use.

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

## Fresh session

One copyable line per phase, nothing else. The packet already says the rest.
Print the form for the agent you are running in; if you cannot tell, print both.

```
claude "execute ~/.shape/plans/<slug>/phases/01-upload-workspace-schema.md"
codex  "execute ~/.shape/plans/<slug>/phases/01-upload-workspace-schema.md"
```

## Report

Four lines, maximum:

- design path
- phase count and index path
- what was started, or the command to start phase 1
- anything left undone

Then stop. No next-steps list, no offer of more work.

## Archive

When the last phase merges, `shape archive <slug>` moves the folder to
`$SHAPE_PLANS_DIR/archived/<slug>/`. Nothing is deleted — `.shape/transcript.md`
is the record of why the design is what it is.
