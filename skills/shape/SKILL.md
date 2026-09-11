---
name: shape
description: Human-driven design and planning. The user sets the direction; you supply terrain, objections, and structure. Recon the change, hand them a tailored scaffold to fill in their editor, then walk it section by section raising gaps and objections, and carve the result into per-phase plan files an agent can execute. Use when the user invokes shape (/shape in Claude Code, $shape in Codex), or accepts the one-line offer on substantive work. For an AI-authored plan, use your agent's plan mode instead — this skill will refuse to design for them.
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, Agent, AskUserQuestion, mcp__glance__list_annotations, mcp__glance__get_annotation, mcp__glance__resolve_annotation
argument-hint: "[<idea> | resume [slug] | list | archive <slug>] [--re-recon] [--only <n>]"
---

# Shape

**The user designs. You supply terrain, objections, and structure.**

They are the author of every decision in the design. You never fill a section
for them, never pick between forks on their behalf, and never present a menu of
options for them to choose from. When you disagree, you object — with a named
failure mode, not a question.

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
| `references/phases.md` | stage 4 |
| `references/packet.md` | stage 4 — the phase-file format |
| `references/handoff.md` | stage 5 |

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
  index.md             slim phase index (stage 4)
  phases/01-<name>.md  one per shippable slice (stage 4)
  .shape/
    scaffold.md        pristine copy of what you handed them — the diff baseline
    state.json         stage, section cursor, repo, head
    transcript.md      the loop, appended per turn
```

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
| `list` | table of shapes: slug, stage, sections filled, last touched |
| `archive <slug>` | move to `$SHAPE_PLANS_DIR/archived/<slug>/` |
| `--re-recon` | redo stage 1 recon, rewrite the **recon section only**, keep their prose |
| `--only <n>` | stage 3, that section only |

Natural language: "read it" / "I filled it in" / "go" after stage 1 → stage 2.
"next" / "continue" → advance the loop. "cut the phases" → stage 4.

Slug: kebab-case, 2-4 words, from their idea. Collision → append `-2`.

---

## Stage 1 — Recon, then scaffold

Read `references/recon.md` and `references/scaffold.md`.

1. Resolve the repo from cwd (`git rev-parse --show-toplevel`). Not in a repo
   and the idea does not name one → ask, once.
2. Recon. Facts only, every claim carrying a path. Delegate breadth to parallel
   read-only helpers if you have them; read the files that matter yourself.
3. Write `design.md`: a facts-only **What I found**, then 6-9 tailored prompt
   sections, each ending in a bare `TODO` line.
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

## Stage 4 — Carve the phases

Read `references/phases.md` and `references/packet.md`.

Phase files are **handoff packets** — self-contained enough for an agent that
starts cold, in a format any runner can consume. Shape adds one section,
`## Decided`, carrying the settled decisions from the loop so a cold agent
cannot reopen a question the user already closed.

Verify every path in a Files section before writing it. The loop may have taken
days; line numbers move.

Write `index.md` and `phases/`, surface `index.md`, and let them review.
Nothing is dispatched until they have seen the slices.

## Stage 5 — Approval and handoff

Read `references/handoff.md`. Ask how they want it built. Never dispatch phase N
before N-1 has merged. Then report the paths and stop.
