# Approval and handoff

Phases are approved. Two things get decided here, and the user decides both:
how each phase is sequenced, and who builds.

## Sequencing — stacked by default

On a GitHub repo, every phase after the first starts as **stack on N-1**. Fill
the index's **Sequencing** column that way, then show it once, with a line
under any phase whose facts argue against a stack — not reversible, shares
files with its parent, no CI on a PR based on a feature branch. Ask once
whether to change any cell. They change the ones they want; the rest stay
stacked. Phase 1 has no parent and needs no question.

The three answers a cell can hold, on a GitHub repo:

1. **Stack on N-1** — branch from the parent's branch, PR based on the parent's
   branch. Build can start once N-1 is logged **built**. Merge waits for N-1.
2. **After N-1 merges** — branch from the default branch once the parent has
   merged. Nothing starts before that.
3. **Alone on main** — same as 2, but named so the log says why: the phase is
   not reversible, or its parent's merge has to settle first.

Off GitHub, only 2 applies. Carving already wrote `after N-1 merges` into
every cell; if any cell says otherwise, rewrite it. Skip the question and say
so once.

The lines under a phase are facts only: "not reversible", "shares
`finalize.ts` with phase 1 — a squash merge of 1 will conflict here", "this
repo runs no CI on a PR based on a feature branch". No recommendation beyond
the default. Write their changes into the column before asking who builds.

A stack on a squash-merge repo costs one thing every time: when a parent
squash-merges, the child shows conflicts in the parent's own files. Say it once
at handoff, in one line. Stage 7 handles it — `references/execute.md`.

## Who builds

Ask one question, titled **Build it**, with these three options, plus any
build option the personal context file adds. Use a
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
Always print the form for the agent you are running in. Print the other
agent's form too only when its CLI is installed (`command -v codex`,
`command -v claude`) — not everyone has both.

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
