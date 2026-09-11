# shape

A planning skill where **you** design and the agent is the sounding board.
Works in [Claude Code](https://claude.com/claude-code) and
[Codex](https://developers.openai.com/codex), CLI or desktop app.

Most planning tools have the roles backwards. The agent writes the plan, you
read it and say "looks good" — which makes you the reviewer of someone else's
design instead of the author of your own. `shape` inverts that. The agent does
the research and the arguing. Every decision is yours.

## The flow

**1. You say what you want, in a sentence.**

```
/shape uploads need to remember which workspace they came from
```

(`$shape …` in Codex.)

**2. The agent recons the code and hands you a scaffold.** Not a draft design — a
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

**4. The agent walks it with you, one section per turn** — gaps and objections
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

**5. It re-reads the whole document once.** The loop went section by section;
this pass catches the places where §6's answer quietly changed what §2 settled,
before anything is carved from it.

**6. It carves the design into phase files** — one shippable pull request each,
self-contained enough to hand to an agent that starts cold.

**7. You execute phases one at a time**, usually each in a fresh session:

```
/shape execute upload-workspace-attribution
```

One phase per run, never more. Before it writes code, the session reads the
design end to end, reads the log of what earlier phases actually shipped,
re-verifies the packet's file paths, and asks whether the phase is still the
right next move. After review it opens the pull request and writes the log
entry the next phase will read. Phase 3 learns what phase 2 really did from
disk, not from a session you had to keep alive.

On GitHub, phases can ship as **stacked pull requests** — phase 2 branched from
phase 1's branch, its PR based on phase 1, so each PR shows only its own diff
and phase 2 can be built before phase 1 merges. That is decided per phase at
handoff, by you, with the facts in front of you: whether it is reversible,
whether it touches the parent's files, whether the repo runs CI on a PR based
on a feature branch. When a parent squash-merges, `execute` checks whether
GitHub rebased the child before touching anything.

## What it will not do

- Fill in a section for you
- Pick between forks on your behalf
- Give you a menu of options with tradeoffs

When the agent disagrees it objects, with a named failure mode and a cited line —
capped at two per section. Refute the premise and the objection is dropped, not
downgraded and carried forward.

If you want the agent to write the plan, use its plan mode. This skill will
refuse.

## Install

As a plugin. Claude Code:

```
/plugin marketplace add ncrohn/claude-plugins
/plugin install shape@ncrohn-plugins
```

Codex:

```
codex plugin marketplace add ncrohn/claude-plugins
codex plugin add shape@ncrohn-plugins
```

Or clone it and link the skill folder. Codex reads `~/.agents/skills`; Claude
Code reads `~/.claude/skills`. Link both and one clone serves both agents:

```
git clone https://github.com/ncrohn/shape.git
ln -s "$PWD/shape/skills/shape" ~/.agents/skills/shape
ln -s "$PWD/shape/skills/shape" ~/.claude/skills/shape
```

## The one-line offer

A skill cannot volunteer itself. If you want the agent to suggest `shape` on
work that warrants it, add this to your `CLAUDE.md` or `AGENTS.md`:

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
| `/shape execute [slug] [n]` | run the next unmerged phase, or phase `n`, with the drift check |
| `/shape list` | every shape and its stage |
| `/shape archive <slug>` | retire a finished one |
| `/shape --re-recon` | redo recon, keep your prose |
| `/shape --reconcile` | re-read the whole design for contradictions, on demand |
| `/shape --only <n>` | loop one section |

In Codex, replace `/shape` with `$shape`. Everything after the skill name is
the argument string.

## What differs per agent

Nothing in the flow. The skill needs file read/search, shell, file write, and a
way to ask you a question. Where an agent lacks a nicety it degrades:

| Capability | Claude Code | Codex | Without |
|---|---|---|---|
| Parallel recon | subagents | spawned agents | grep sweep in the main thread |
| Build-it question | option picker | option picker | numbered list, then wait |
| Fresh-session dispatch | `claude "/shape execute <slug>"` | `codex "$shape execute <slug>"` | print the packet path |

## License

MIT
