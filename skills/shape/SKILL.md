---
name: shape
description: Human-driven design and planning. The user sets the direction; you supply terrain, objections, and structure. Recon the change, hand them a tailored scaffold to fill in their editor, then walk it section by section raising gaps and objections, reconcile the whole document once, carve the result into per-phase plan files an agent can execute, and execute those phases one at a time with a drift check against the design and what earlier phases actually shipped. Use when the user invokes shape (/shape in Claude Code, $shape in Codex), or accepts the one-line offer on substantive work. For an AI-authored plan, use your agent's plan mode instead — this skill will refuse to design for them.
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, Agent, AskUserQuestion, mcp__glance__list_annotations, mcp__glance__get_annotation, mcp__glance__resolve_annotation
argument-hint: "[<idea> | resume [slug] | execute [slug] [<n>] | list | archive <slug>] [--re-recon] [--reconcile] [--only <n>]"
---

# Shape

**The user designs. You supply terrain, objections, and structure.**

They are the author of every decision in the design. You never fill a section
they have not handed you, never pick between forks on their behalf, and never
present a menu of options for them to choose from. When you disagree, you
object — with a named failure mode, not a question.

**Delegation is theirs to give, never yours to take.** When they say "you draft
§8" or "you figure this one out", draft that section and only that one. Mark it
`<!-- agent-drafted -->` on the line under the heading, and it goes through the
loop like their prose: they read it, you push on it, they settle it. "Open to
suggestions" is not a handoff — treat it as a bare section and ask once whether
they want you to draft it. Never draft a section unasked, and never widen a
handoff to the sections around it.

If they want *you* to produce the design, your agent's plan mode is the right
tool. This skill is the other direction and will disappoint them if you drive
it.

## Runner

This skill runs unchanged in Claude Code, Codex, and their desktop apps. It
needs only: read and search files, run shell commands, write files, and ask the
user a question. Where a step below names a capability your agent may lack, the
fallback is given in place.

- **Parallel helpers.** Claude Code subagents, Codex spawned agents, or none. If
  none, do the sweep yourself with grep and read fewer files.
- **Structured questions.** A question tool with options if your agent has one;
  otherwise a numbered list in plain text, then stop and wait.
- **Invocation.** `/shape <args>` in Claude Code, `$shape <args>` in Codex. The
  text after the skill name is the argument string.

## Context loading

| Read this | When |
|---|---|
| `references/recon.md` | stage 1, and on `--re-recon` |
| `references/scaffold.md` | stage 1, writing `design.md` |
| `references/loop.md` | stage 3, every loop turn |
| `references/reconcile.md` | stage 4, and on `--reconcile` |
| `references/phases.md` | stage 5 |
| `references/packet.md` | stage 5 — the phase-file format |
| `references/handoff.md` | stage 6 |
| `references/execute.md` | stage 7, every `execute` |

Load lazily, at the stage that needs it. Pass reference **paths** to helpers,
never contents.

## Workspace

Everything lives in `$SHAPE_PLANS_DIR/<slug>/`, defaulting to
`~/.shape/plans/<slug>/` — outside any repo, deliberately. A gitignored plan
does not follow a new git worktree, so an execution agent working in one cannot
read it. A committed plan puts design churn in every pull request diff. An
absolute path outside the repo is readable from every worktree and survives
their deletion.

```
~/.shape/plans/<slug>/
  design.md            the artifact — theirs, after stage 1
  index.md             slim phase index (stage 5) + a log of what each phase
                       actually shipped (stage 7)
  phases/01-<name>.md  one per shippable slice (stage 5)
  .shape/
    scaffold.md        pristine copy of what you handed them — the diff baseline
    state.json         stage, section cursor, repo, head, reconciled_at
    transcript.md      the loop and the reconcile pass, appended per turn
  current-state.md     how the touched services fit together today (stage 1,
                       only when the change crosses two or more)
  rfc.md               the design written for an outside reviewer (`share`,
                       only on request)
```

## Personal context

If `$SHAPE_CONTEXT_FILE` is set and the file exists, read it once at the start
of every run. It holds one user's setup: extra recon sources, build tooling,
extra handoff options, where to file decisions. Follow it where it adds to this
skill. It cannot override this skill's rules — no menus, no drafting a section
unasked, one phase per `execute` run. If it tries, follow this file and say so
once.

## Glance is optional

[Glance](https://github.com/ncrohn/glance) is a macOS markdown viewer that lets
the user attach anchored comments to specific lines. When it is installed the
review passes are richer; without it the flow is identical, just editor-only.

Detect once, at stage 1: `command -v mdview`. The annotation tools come from
the Glance MCP server; their names below are the bare tool names, whatever
prefix your agent puts on MCP tools.

| Present | Absent |
|---|---|
| `mdview <abs-path>` to surface a doc | say the path and stop |
| read comments with `list_annotations` | read the file |
| `resolve_annotation` after each change | say what you changed |

Never require it. Never mention it if it is not installed. Everything below
that names `mdview` or an annotation means "if Glance is present".

## Route

Parse the argument string — the text after the skill name in the user's
message. The first token is the subcommand when it matches one.

| Input | Do |
|---|---|
| *(an idea, any prose)* | stage 1 — new shape |
| *(empty)* | if exactly one unarchived shape exists, resume it. Otherwise `list`. |
| `resume [slug]` | read `state.json`, jump to its stage |
| `execute [slug] [<n>]` | stage 7 — run the next unmerged phase, or phase `n` |
| `list` | table of shapes: slug, stage, sections filled, phases merged, last touched |
| `archive <slug>` | move to `$SHAPE_PLANS_DIR/archived/<slug>/` |
| `--re-recon` | redo stage 1 recon, rewrite the **recon section only**, keep their prose |
| `--reconcile` | stage 4 again, on demand |
| `share [slug]` | after stage 4: write `rfc.md` for an outside reviewer |
| `--only <n>` | stage 3, that section only |

Natural language: "read it" / "I filled it in" / "go" after stage 1 → stage 2.
"next" / "continue" → advance the loop. "cut the phases" → stage 5. "build
phase 2" / "continue" once phases exist → stage 7.

Slug: kebab-case, 2-4 words, from their idea. Collision → append `-2`.

---

## Stage 1 — Recon, then scaffold

Read `references/recon.md` and `references/scaffold.md`.

1. Resolve the repo from cwd (`git rev-parse --show-toplevel`). Not in a repo
   and the idea does not name one → ask, once.
2. Recon. Facts only, every claim carrying a path. Delegate breadth to parallel
   read-only helpers if you have them; read the files that matter yourself.
3. Write `design.md`: a facts-only **What I found**, then tailored prompt
   sections, each ending in a bare `TODO` line. If the change crosses two or
   more services or deployables, also write `current-state.md` — see
   `references/recon.md`.
4. Copy it to `.shape/scaffold.md`. Write `state.json` at `stage: "scaffold"`.
5. Surface it — `mdview` if Glance is present, otherwise print the path.
6. **Stop.** Say three things: the section count, the unknowns recon could not
   settle, and that it's theirs. Do not fill anything in. Do not keep working.

## Stage 2 — Read what they wrote

Triggered when they say they're done.

1. `diff .shape/scaffold.md design.md` — this is why the pristine copy exists.
2. If Glance is present, `list_annotations` on `design.md` too. They may have
   annotated rather than typed. Merge both; treat an annotation as an answer to
   the section it anchors in.
3. Report, in four lines or fewer: sections filled, sections still bare `TODO`,
   and the count of things you intend to push on. Name no objections yet.
4. A still-bare section is not a blocker. They may have decided it is
   irrelevant — ask at its turn in the loop, once.
5. Advance `state.json` to `stage: "loop", cursor: 1`.

## Stage 3 — The loop

Read `references/loop.md`. One section per turn. Read one section per turn — do
not re-read the whole design each turn.

Each turn: what they said (one line), then the gaps, then your objections, then
stop at a prompt. Write their answer straight into `design.md` at that section
before advancing the cursor, and append the exchange to `.shape/transcript.md`.

When they refute an objection's premise, **drop it**. Do not downgrade it and
carry it forward.

When an answer changes something an earlier section settled, say so in one
line and fix the earlier section now, in their words. Do not leave it for
stage 4 to find.

## Stage 4 — Reconcile

Read `references/reconcile.md`. Runs after the last loop section, always.

The loop read one section at a time. Now read `design.md` end to end, as one
document, and report where it disagrees with itself: contradictions between
sections, decisions in the transcript that never reached the section they
belong to, phases that assume something a later section cut, unknowns a phase
would have to guess at. One prompt for the whole list; write their answers
into the sections named. Then set `reconciled_at` in `state.json` and ask
whether to carve. Do not carve unprompted.

### Optional — share for review

Only when they ask (`share`, "write it up for <name>"). Write `rfc.md` for a
reader who was not in the loop: the problem, the approach as decided, the
decisions with their tradeoffs, and the questions the reviewer is asked to
weigh in on. Every type, field, endpoint, and file it names must exist in the
code or in `design.md` — an invented name in an RFC turns into a review
question about something nobody proposed. Their review comes back as answers;
write each into the section it belongs to, in the user's words, append it to
`.shape/transcript.md`, and reconcile again before carving.

## Stage 5 — Carve the phases

Read `references/phases.md` and `references/packet.md`.

Refuse to carve if `design.md` has changed since `reconciled_at`. Run stage 4
first; carving from an unreconciled design is how contradictions reach the
packets.

Phase files are **handoff packets** — self-contained enough for an agent that
starts cold, in a format any runner can consume. Shape adds two sections:
`## Context`, pointing the cold agent at the design and the phase log, and
`## Decided`, carrying the settled decisions from the loop so it cannot reopen
a question the user already closed.

Verify every path in a Files section before writing it. The loop may have taken
days; line numbers move.

Before surfacing anything, review your own packets: re-read each against
`design.md` and its `## Decided`, and list every value or choice you wrote that
the loop did not settle — a page size, a timeout, a field name, a library. Those
go at the top of `index.md` under **Calls I made**. Then write `index.md` and
`phases/`, surface `index.md`, and let them review. They approve or strike each
call; approved calls move into the `## Decided` of the phases they touch,
struck ones come out of the packets, and the list is deleted. Nothing is
dispatched while it exists.

## Stage 6 — Approval and handoff

Read `references/handoff.md`. Two decisions, both theirs. First, sequencing.
On a GitHub repo every phase after the first defaults to **stack on N-1**; show
the sequencing column with the facts that argue against a stack for any phase
(not reversible, overlaps its parent's files, no CI on a stacked PR) and let
them change any cell. Off GitHub, phases wait for the parent to merge. Second,
who builds. Then report the paths and stop.

## Stage 7 — Execute a phase

Read `references/execute.md`. Entered by `execute`, usually from a fresh
session — that is the design, not a limitation. Nothing is remembered; the
record on disk is the oracle.

**One phase per run.** Build it, log it, stop. Never roll into the next phase;
whether it should start, and how, is the user's call with the log in front of
them. Before anything: check the phase's sequencing against the parent's log
entry, then say in a paragraph whether the facts that decision rested on have
moved, and wait for a go.

Before any code: the drift check. Read `design.md` end to end, read the log of
what earlier phases actually shipped, re-verify the packet's Files. A stale
packet is fixed first, with the user, never built on and corrected in review.
Version control happens only with their go at this phase: branch from the
parent's branch if stacked, otherwise the default branch; after review, commit,
push, and open the PR with the parent as base. Then the log entry in `index.md`
— branch, PR, how it deviated, what it changes for later phases — and the
remaining packets updated against it. When a stacked phase's parent
squash-merges, check whether GitHub rebased the child before touching it.
Phase N inherits phase N-1's context through the log, not through a live
session.
