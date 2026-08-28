# shape

A planning skill for [Claude Code](https://claude.com/claude-code) where **you**
design and Claude is the sounding board.

Most planning tools have the roles backwards. Claude writes the plan, you read
it and say "looks good" — which makes you the reviewer of someone else's design
instead of the author of your own. `shape` inverts that. Claude does the
research and the arguing. Every decision is yours.

## The flow

**1. You say what you want, in a sentence.**

```
/shape uploads need to remember which workspace they came from
```

**2. Claude recons the code and hands you a scaffold.** Not a draft design — a
form, with the questions the code says are actually undecided:

```markdown
## What I found

- `UploadRecord` has no workspace field — src/domain/UploadRecord.ts:34
- The workspace is derived at read time from the uploader's current
  membership, never from the upload — src/resolvers/upload.ts:18
- 41,203 upload rows exist; none carry a workspace reference

## 2. Where the workspace gets stamped

> Upload time is two events, not one: `CreatePresignedUrl`
> (src/handlers/presign.ts:88) and `FinalizeUpload`
> (src/handlers/finalize.ts:142). The row is written at presign, before
> the bytes land. Which event writes the stamp, and does it write or update?

TODO
```

Then it stops. It does not fill anything in.

**3. You fill it in, in your editor.** Take ten minutes or take two days.

**4. Claude walks it with you, one section per turn** — gaps and objections
against what you wrote:

```
── §2/7 · Where the workspace gets stamped ──

You said: stamp at finalize, not presign — a presigned URL can expire
without the upload ever happening.

Objection: FinalizeUpload runs in a webhook from the storage provider
(src/handlers/finalize.ts:142), which carries no session. The membership
lookup you are describing needs a user context that isn't there.

> _
```

**5. It carves the design into phase files** — one shippable pull request each,
self-contained enough to hand to an agent that starts cold.

## What it will not do

- Fill in a section for you
- Pick between forks on your behalf
- Give you a menu of options with tradeoffs

When Claude disagrees it objects, with a named failure mode and a cited line —
capped at two per section. Refute the premise and the objection is dropped, not
downgraded and carried forward.

If you want Claude to write the plan, use plan mode. This skill will refuse.

## Install

```
/plugin marketplace add ncrohn/claude-plugins
/plugin install shape@ncrohn-plugins
```

Or clone it straight into your skills directory:

```
git clone https://github.com/ncrohn/shape.git
ln -s "$PWD/shape/skills/shape" ~/.claude/skills/shape
```

## The one-line offer

A skill cannot volunteer itself. If you want Claude to suggest `shape` on work
that warrants it, add this to your `CLAUDE.md`:

```markdown
## Shaping substantive work

`/shape` is my design flow: you recon the change, hand me a scaffold, I fill it
in, then we loop it section by section. I drive the design. You supply terrain,
objections, and structure — never the decisions.

**Offer it, don't start it.** When an ask touches more than one file *and* has a
fork you can't resolve from the code, add exactly one line — "This looks like
`/shape` work." — and then get on with the task if I don't take it. One offer
per ask. Never repeat it, never wait for an answer.

Don't offer for: single-file changes, mechanical sweeps and renames however
wide, test additions, or questions. Breadth alone is not a fork.
```

The bar matters more than it looks. Breadth alone is not a fork — a rename
across forty files needs no design. A two-file change with an undecided schema
question does.

## Configuration

Plans are written to `~/.shape/plans/<slug>/`, outside any repository. Set
`SHAPE_PLANS_DIR` to move them.

They live outside the repo on purpose. A gitignored plan does not follow a new
git worktree, so an execution agent working in one cannot read it. A committed
plan puts design churn in every diff.

## Glance

Optional. If [Glance](https://github.com/ncrohn/glance) is installed, `shape`
opens the scaffold and the phase index in it, and reads your anchored comments
as answers. Without it the flow is identical — you edit the file and say when
you're done.

```
brew install ncrohn/glance/glance
```

## Commands

| Command | Does |
|---|---|
| `/shape <idea>` | start a new shape |
| `/shape resume [slug]` | pick up where you left off |
| `/shape list` | every shape and its stage |
| `/shape archive <slug>` | retire a finished one |
| `/shape --re-recon` | redo recon, keep your prose |
| `/shape --only <n>` | loop one section |

## License

MIT
