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

On a GitHub repo a phase can ship as a **stacked pull request**: branched from
phase N-1's branch, PR based on N-1's branch, so GitHub shows only its own diff
and it can be built before its parent merges. That is an option per phase, not
a mode for the plan. Nobody decides it here — carving records the facts the
decision needs, and the user makes the call for each phase at handoff.

Facts to record per phase, in the index:
- **Reversible?** A phase that cannot be reverted argues for building alone on
  main after its parent merges, so a revert is one PR.
- **Touches the parent's files?** Overlap is where a squash-merged parent gives
  the child phantom conflicts. Note it.
- **Needs CI before merge?** On some repos a draft PR based on a feature branch
  gets no CI. The user needs to know before they stack a phase whose only
  check is CI.

Branch names come from the packet filename: `<slug>/01-upload-workspace-schema`.
Phase 1 branches from the default branch. A later phase branches from its
parent's branch if stacked, from the default branch if not.

The squash-merge cascade — GitHub rebasing the child itself, or the phantom
conflicts when it does not — is stage 7's job, in `references/execute.md`.

Not on GitHub: sequential-on-main only. Never start phase N until N-1 has
merged. Most runners branch off a fresh default-branch head, so a dependent
phase started early is building against a tree that does not exist.

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

| # | Phase | Branch | Ships | Reversible | Overlaps parent | Sequencing |
|---|---|---|---|---|---|---|
| 1 | Schema + write path behind flag | `upload-workspace-attribution/01-schema-write-path` | dark, nothing reads it | yes | — | *decided at handoff* |
| 2 | Reads switch to the new field | `upload-workspace-attribution/02-reads-new-field` | correct attribution | flag off | `finalize.ts` | *decided at handoff* |
| 3 | Backfill 41k rows | `upload-workspace-attribution/03-backfill` | historical uploads attributed | **no** | none | *decided at handoff* |

The **Sequencing** column is filled at handoff, one phase at a time, by the
user: `stack on 1`, `after 1 merges`, or `alone on main`. Carving leaves it
as shown.

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
