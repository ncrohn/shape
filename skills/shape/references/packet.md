# The handoff packet format

A phase file is a **packet**: one markdown file describing one unit of work for
a coding agent that starts cold. The format is runner-agnostic on purpose. Feed
it to a fresh session, pipe it to a CLI agent on stdin, or paste it into a
ticket — it is plain markdown and it assumes no tooling.

The packet carries the research already done. Verified paths with line anchors,
not "find where the upload handler lives." Everything learned during recon and
the loop that is left out of the packet is lost.

One packet per shippable phase. The same granularity as a reviewable pull
request.

## Template

```markdown
# <what ships when this is done>

## Goal

One paragraph. The outcome, not the steps.

## Decided

These are settled. Do not redesign them; if one looks wrong, stop and say so
in your final message.

- Stamp at `FinalizeUpload`, not `CreatePresignedUrl`. A presigned URL can
  expire without the upload happening, which would leave a row attributed to
  an upload that never landed.
- Existing rows get null, never a guessed workspace. A wrong attribution is
  worse than a missing one.

## Files

Verified before this packet was written — each path exists at the stated line.

- `src/domain/UploadRecord.ts:34` — add `workspaceId`, nullable
- `src/handlers/finalize.ts:142` — write it here

## Approach

Numbered steps. Enough that a cold reader does not have to invent the design.

1. ...
2. ...

## Do not touch

- Anything under `src/billing/` — a separate phase owns it
- The public API schema
- Existing test expectations. If a test fails, the production code is wrong,
  not the test. Stop and say so in your final message.

## Acceptance

The literal command, and what passing looks like.

    npm test -- src/handlers

## Done means

- `workspaceId` persists on new uploads
- The new case is covered by a test
- The acceptance command exits 0
- Nothing reads the field yet

## Rules

- Do not commit, push, switch branches, or bypass hooks. Leave changes in the
  working tree.
- Do not touch files outside the list above without saying why in your final
  message.
- If the packet is wrong, or a file is not where it says, stop and report it.
  Do not improvise a different design.

## Report back

Final message: what changed, what you ran and what it returned, and anything
you could not do. Facts only — no summary of the packet.
```

## Why each section is there

**Decided** is what makes this a shape packet rather than a generic task file.
A cold agent has none of the loop's context and will happily reopen a question
that took an hour to settle. Without this section the user answers it twice.

**Files** is the expensive part, and the reason a packet beats a one-line
prompt. A cold agent that has to locate the code first spends its context on
search and arrives at the edit with less room to think.

**Do not touch** is the scope fence. The agent has none of the planner's
judgment about what is in scope and no memory of the conversation.

**Acceptance** must be a command, not a description. "Make sure tests pass" is
not checkable; a command with an exit code is.

**Rules** repeats the git prohibition even if the project's own instructions
already say it. Committing is the one thing that is expensive to undo.

## Reviewing the result

The agent never reviews its own diff. Read its final message and the working
tree diff, review it yourself, and send findings back into the same context if
the runner supports it.

Two correction rounds. If a third is needed, the packet was wrong — rewrite the
packet and start fresh rather than patching a confused agent.
