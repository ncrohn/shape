# Approval and handoff

Phases are approved. Two things get decided here, and the user decides both:
how each phase is sequenced, and who builds. Do not assume and do not default.

## Sequencing — one phase at a time

Nothing here is decided for the whole plan at once. Walk the index top to
bottom. For each phase, state the facts the index recorded — reversible, files
it shares with its parent, whether it needs CI before merge — in one or two
lines, then ask. Phase 1 has no parent and needs no question.

Three answers, on a GitHub repo:

1. **Stack on N-1** — branch from the parent's branch, PR based on the parent's
   branch. Build can start once N-1 is logged **built**. Merge waits for N-1.
2. **After N-1 merges** — branch from the default branch once the parent has
   merged. Nothing starts before that.
3. **Alone on main** — same as 2, but named so the log says why: the phase is
   not reversible, or its parent's merge has to settle first.

Off GitHub, only 2 applies; skip the question and say so once.

Say one sentence per phase, only when it applies, and only facts: "not
reversible", "shares `finalize.ts` with phase 1 — a squash merge of 1 will
conflict here", "this repo runs no CI on a PR based on a feature branch". No
recommendation. They pick. Write the answer into the index's **Sequencing**
column before moving to the next phase.

## Who builds

Ask one question, titled **Build it**, with these three options. Use a
structured question tool if your agent has one; otherwise a numbered list, then
stop and wait.

1. **Fresh session per phase** — you print one `execute` command per phase and
   stop. Nothing fires. Each session runs the drift check before it builds,
   builds one phase, writes the log, and stops.
2. **This session, phase 1 now** — you implement phase 1 with the design still
   in context, then stop for their review. Phase 2 is a new decision, not a
   continuation. The log still gets written after each merge; context in your
   head is not context on disk.
3. **Their own runner** — packets are plain markdown. Print the absolute paths
   and let them feed them to whatever they use. Tell them once: the packet's
   `## Context` expects the log to be kept, and `shape execute` will keep it
   for them if they run that instead.

Stacking needs git: creating branches, pushing, opening PRs with the parent as
base. Say in one line that `execute` does those, and only those, when a phase
is sequenced as a stack — and never commits the packet agent's work without
their go-ahead at that phase. If they do not want the skill touching git, the
answer is option 3.

## Rules that are not negotiable

- **In order.** A phase starts only when its Sequencing says it can: the parent
  is logged built (stack) or merged (after / alone). Never earlier.
- **One phase per run.** `execute` builds one phase and stops. It never rolls
  into the next one, however clean the last one was. Starting the next phase is
  the user's call, made with the log in front of them.
- **The agent never reviews its own diff.** Whoever wrote the code does not
  decide whether it is right.
- **Two correction rounds.** If a third is needed, the packet was wrong —
  rewrite the packet and start fresh rather than patching a confused agent.
- **The log is written after every build and every merge.** Whatever runs the
  phase, the entry in `index.md` is how the next phase learns what this one
  actually did. A live session is not a substitute — it gets closed, and the
  next implementer cannot reach it.

## Fresh session

One copyable line per phase, nothing else. The packet already says the rest.
Print the form for the agent you are running in; if you cannot tell, print both.

```
claude "/shape execute <slug> 1"
codex  "\$shape execute <slug> 1"
```

The raw packet still works — `claude "execute ~/.shape/plans/<slug>/phases/01-…md"`
— but skips the drift check, the branch setup, and the log. Print it only for
option 3.

## Report

Four lines, maximum:

- design path
- phase count, index path, and the sequencing column in one line
  (`1 → 2 stacked → 3 alone`)
- what was started, or the command to start phase 1
- anything left undone

Then stop. No next-steps list, no offer of more work.

## Archive

When the last phase is logged merged, `shape archive <slug>` moves the folder to
`$SHAPE_PLANS_DIR/archived/<slug>/`. Nothing is deleted — `.shape/transcript.md`
is the record of why the design is what it is.
