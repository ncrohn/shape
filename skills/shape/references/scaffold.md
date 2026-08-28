# The scaffold

`design.md` is a form the user fills in their editor. Its quality is the whole
skill — a bad scaffold asks generic questions they resent answering, a good one
asks the three questions the code says are actually undecided.

## Shape of the file

```markdown
# Attribute uploads to their workspace

<!-- shape: slug=upload-workspace-attribution repo=acme-api stage=scaffold -->

## What I found

*Facts, with paths. Not recommendations.*

- `UploadRecord` has no workspace field — `src/domain/UploadRecord.ts:34`
- The workspace is derived at read time from the uploader's current
  membership, never from the upload itself — `src/resolvers/upload.ts:18`
- 41,203 upload rows exist; none carry a workspace reference *(prod read
  replica, queried today)*
- `UNKNOWN:` whether the admin UI reads the same derivation — its query is
  defined in the dashboard tool, not in this repo

---

## 1. What breaks, observably

> Someone sees an upload attributed to the wrong workspace. Who, on which
> screen, and what do they do next?

TODO

## 2. Where the workspace gets stamped

> Upload time is two events, not one: `CreatePresignedUrl`
> (`src/handlers/presign.ts:88`) and `FinalizeUpload`
> (`src/handlers/finalize.ts:142`). The row is written at presign, before
> the bytes land. Which event writes the stamp, and does it write or update?

TODO
```

## Rules

**A bare `TODO` on its own line ends every section.** Stage 2 diffs against
`.shape/scaffold.md` to find what they filled; a `TODO` that survives is an
unanswered section. Never write `TODO:` with trailing text — the diff needs it
bare.

**The prompt is a blockquote, above the `TODO`.** It stays visible while they
write below it, and it survives into the final document as the record of what
was asked.

**A prompt states what recon found, then asks.** Two to four lines. "Which event
writes the stamp?" is a generic question. The example in section 2 above is the
same question after recon — it tells them the two events exist, where they live,
and the ordering problem they would otherwise discover at implementation time.

**Never propose an answer in a prompt.** Not "presumably at finalize?" and not a
list of options with tradeoffs. Name the fork, hand them the facts, stop.

**Six to nine sections.** Fewer and you are not covering the design. More and
they will not fill it. Cap the file at roughly 120 lines of scaffold.

## Section selection

Five sections earn their place on nearly every shape, because they are where
designs die:

1. **What breaks, observably** — in terms of what a person sees, not what the
   code does. Forces a real problem statement.
2. **Approach** — their design, in their words. The one section that must never
   be pre-shaped by a narrow prompt.
3. **Blast radius** — who else reads or writes this. Seed it with the callers
   recon found, and ask what they want to happen to each.
4. **Failure mode** — what happens when this is wrong in production. Not "how do
   we test it".
5. **Rollout** — flag, backfill, ordering, and whether it is reversible.

Then one to four sections from the forks recon turned up, and always:

6. **Phases** — their own first cut at shippable slices. Stage 4 argues with
   this cut; it does not invent one from nothing.
7. **Out of scope** — cheap to write, and it is what stops stage 4 sprawling.

Drop any of the five that genuinely does not apply. A pure refactor has no
rollout section. Do not pad.

## Ordering

Problem first, then their approach, then the consequences of it. They should be
able to stop after section 2 and have said the most important thing.

## Hand-off

Surface it, then say three things and nothing else:

- the section count
- the unknowns recon could not settle, so they know what they are deciding blind
- that it is theirs

Then stop. Do not offer to fill a section. Do not keep working in the same turn.
