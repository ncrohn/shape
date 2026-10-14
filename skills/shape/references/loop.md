# The loop

One section per turn. Read that section only — a nine-section shape must not
accumulate nine sections of context.

## The turn

```
── §2/7 · Where the workspace gets stamped ──

You said: stamp at finalize, not presign — a presigned URL can expire
without the upload ever happening.

Gap: nothing about an upload that finalizes after the uploader has left
the workspace. Does it take the workspace they had at presign, or now?

Objection: FinalizeUpload runs in a webhook from the storage provider
(src/handlers/finalize.ts:142), which carries no session. The membership
lookup you are describing needs a user context that isn't there. Today's
derivation works precisely because it happens at read time, inside a
request.

> _
```

Four parts, in this order, then stop:

1. **You said** — one line, their position in their words. If you cannot state
   it in one line, they did not answer the section; say that instead.
2. **Gaps** — what the section does not say that the next stage needs. A gap is
   a question. At most two.
3. **Objections** — what you think is wrong. At most two.
4. **The prompt.** Then stop. Do not answer your own objection.

Skip any part with nothing in it. Never write "no gaps" or "nothing to object
to" — silence says it.

## What an objection is

A claim, with a failure scenario, resting on cited evidence.

- **Claim** — the thing that does not work. Stated flatly.
- **Failure scenario** — concrete inputs or state, and the wrong outcome. Not
  "this could be risky".
- **Evidence** — the path and line, or the query result. An objection with no
  evidence is a hunch; label it `INFERENCE:` or do not raise it.

Three checks before an objection goes out. Fail any one and it does not go out.

1. **Absence needs more than one search.** "There is no cache", "nothing calls
   this", "the field does not exist" — one grep that found nothing is not
   evidence. Search a second way (another name, the config, the dependency
   list) or read the file where it would live. Still only absence? Label it
   `INFERENCE:`.
2. **Re-read what they wrote.** Read the section again, and the sections it
   points to. If their prose already covers the case, the objection is dead
   before it is raised.
3. **One plain sentence.** State the claim in one sentence a reader new to the
   design would follow. No coined terms, no shorthand you made up this turn
   ("goes blind", "the stamp gap"). If it cannot be said plainly, you do not
   understand it well enough to raise it.

Not an objection:
- "Have you considered..." — that is a gap. Put it in gaps.
- "I would have done it differently." Design taste is theirs. Only object when
  something breaks, costs materially more, or contradicts a fact.
- A menu. Never "you could do A or B". If you have a position, state the
  objection. If you do not, it is not one.

**A turn never ends in "A or B?"** — not in a gap, not in an objection, not in
the prompt. "Does it live on `properties`, `propertiesPage`, or a new query?"
is a menu dressed as a question. Name the fork as a gap — "nothing says which
query carries location search" — give the facts that bear on it, and stop.
When they ask for suggestions, state the facts that constrain the choice; the
choice stays theirs unless they hand you the section.
- Restating a risk they already wrote in the failure-mode section.

## Objection discipline

**Two per section, hard cap.** Pick the two that would change what they build. A
long list reads as noise and they will start skipping the section.

**When they refute the premise, the objection is dead.** Drop it. Do not
downgrade it to a caveat, do not carry it into the phase files, do not raise it
again at a later section. Say "dropped" and move on.

**When they overrule a live objection** — the premise holds, they want it anyway
— that is their call. Record it in the section as a one-line note of the
decision and the tradeoff they accepted, and never raise it again. That note is
the most valuable line in the document six months later.

**Do not soften.** "This might possibly be an issue" wastes the turn. If you
believe it breaks, say it breaks.

## Bare sections

A `TODO` that survived stage 2 gets one turn of its own:

```
── §5/7 · Rollout ── (empty)

Skipped, or not decided yet? If it is genuinely not applicable, say so
and I will cut the section.

> _
```

Ask once. Take "skip it" at face value and cut the section from `design.md`.

## Drafted sections

When they hand you a section by name ("you draft §8"), draft it in that turn
and mark it `<!-- agent-drafted -->` under the heading. The next turn is a
loop turn on it: they read it and change what they want, and you do not
object to your own draft. Their edits replace
yours; the marker stays so a later reader knows where the text came from. Only
the named section — the one beside it is still theirs.

## Writing it down

Before advancing the cursor:

1. Replace the section's `TODO`, or extend the prose they wrote, with the
   settled answer in **their** words, not a paraphrase into yours.
2. Add any decision-with-a-tradeoff as a one-line note under it.
3. If the answer changes something an earlier section settled, say so in one
   line — "§6 changes §2: the stamp now happens at presign" — and fix §2 now,
   in their words. Confirm the rewrite in the same turn. An earlier section
   left stale is the most common thing stage 4 finds, and the cheapest to fix
   here.
4. Append the raw exchange to `.shape/transcript.md`.
5. Bump `cursor` in `state.json`.

They can jump — "go back to section 3", "skip to phases". Follow it; move the
cursor.

## Ending

After the last section, one paragraph: what changed during the loop, and any
objection still live that they overruled. Then go straight to stage 4 —
`references/reconcile.md` — and read the whole document once. Do not ask
whether to carve phases yet; that question belongs at the end of reconcile.
