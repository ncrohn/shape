# Carving phases

Reconcile first. If `design.md` is newer than `reconciled_at` in `state.json`,
stop and run stage 4. A contradiction carved into two packets becomes two
agents building against each other, and the review loop cannot fix it.

Start from **their** phase section. They already cut it once; your job is to
argue with that cut, not to replace it. If you move a seam, say which one and
why in one line.

## What a phase is

One shippable, independently reviewable pull request — and one handoff packet,
one agent job. Merging it leaves the system working and better. "The backend
half" is not a phase; it is half a stack.

Test each candidate:
- **Ships alone?** Merged by itself on a Friday, does anything break?
- **Reviewable in one sitting?** Roughly under 400 lines of real change.
- **One seam?** A phase crossing three services is three phases.
- **Reversible?** If not, say so. It changes review posture, and it argues
  against dispatching it unattended.

Dead-code-first is a legitimate phase: schema plus write path behind a flag,
nothing reading it. Prefer it over a big-bang cut.

Three to six phases. More than six and the design is really two projects — say
so.

## Stacking

Prefer sequential-on-main over a stack of dependent branches. Squash-merging a
parent rewrites its commits, which breaks every child with phantom conflicts.

Most runners branch each job off a fresh default-branch head, so a dependent
phase dispatched before its parent merges is building against a tree that does
not exist. Never dispatch phase N until N-1 has merged, and say so out loud at
handoff.

## The phase file is a packet

Read `references/packet.md` and follow it exactly. Sections, in order:

**Goal · Context · Decided · Files · Approach · Do not touch · Acceptance ·
Done means · Rules · Report back.**

`## Context` points the cold agent at `design.md` and the log in `index.md`,
and tells it to read both before starting. `## Decided` carries the settled
decisions from the loop. Together they are the reason this skill exists —
without them, the executing agent reopens questions the user already closed,
or builds against the plan when an earlier phase already shipped something
different.

Copy the **Rules** and **Report back** blocks rather than paraphrasing them. The
git prohibition is repeated there on purpose.

## The Files section is the expensive part

Every path is verified before the packet is written — the file exists, at the
stated line, saying what you claim. You already read these during recon; check
the line numbers again, because the loop may have taken days.

A packet whose Files section is stale sends a cold agent hunting, and it spends
its context on search instead of the change.

## Naming

`phases/<nn>-<kebab-name>.md`. Many runners derive a branch name from the packet
filename, so the name should read as a branch: `01-upload-workspace-schema.md`,
not `01-phase-one.md`.

## index.md

Slim. A map, not a summary of the design.

```markdown
# Attribute uploads to their workspace — phases

Design: `~/.shape/plans/upload-workspace-attribution/design.md`

| # | Phase | Ships | Start after | Reversible |
|---|---|---|---|---|
| 1 | Schema + write path behind flag | dark, nothing reads it | now | yes |
| 2 | Reads switch to the new field | correct attribution | 1 merged | flag off |
| 3 | Backfill 41k rows | historical uploads attributed | 2 merged | no |

Out of scope: the admin dashboard. It owns its own query; tracked separately.

## Log

*Written by stage 7 as each phase merges. Empty until then.*
```

The `## Log` section is empty at carve time and is the reason the index exists
after carve time. Each merged phase gets an entry — where it merged, how it
deviated from its packet, what that changes for later phases — and the next
phase's drift check reads it. See `references/execute.md`.

## Review pass

Surface `index.md` and stop. If Glance is present, let them annotate — handle
`drifted` and `orphaned` anchors by asking, never by guessing, then apply each
change and resolve it. Otherwise take their notes in the terminal.

Nothing is dispatched until they have seen the slices.
