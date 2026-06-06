# state-docs - convention

The standing "where are we now" layer. Two living docs, each with **ONE distinct job written into its own
header**, kept lean by an overwrite + rolling-archive lifecycle. They describe the **present**; they point at
their siblings (memory / tracker / handoff) for everything that is not "the present state of the project."

They sit at the **bottom of the source-of-truth precedence ladder**: a state doc describes reality, it never
overrides product truth. If a state doc disagrees with the spec, the spec wins - the state doc is simply the
thing that is stale, and the fix is to correct the state doc.

---

## 1. The two docs (distinct audiences - never merge them)

| Doc | Audience | Style | Job (what it owns) |
|---|---|---|---|
| `PROJECT_STATUS.md` (root) | the owner | concise, the project's chosen language | the new-chat bootstrap: **now / next / blockers**, plus the "read these in this order" map |
| `docs/ai/current-state.md` | the AI | detailed, English | **what is built / pending / decided / blocked**, in depth, with the locked architectural decisions |

Two audiences means two registers. The owner doc answers "where are we and what is next" in a couple of
screens; the AI doc answers "what exactly exists, what was decided and why, what is the next concrete step"
with the depth an implementer needs. The moment they start saying the same thing, one of them is wrong - the
role-boundary header (below) is what prevents that drift.

### Why not one doc, and why not two *generic* status docs

One doc cannot serve both readers: the owner-concise version is too thin for the AI, and the AI-detailed
version is unreadable for the owner. But the failure mode in the other direction is just as real: shipping
**two generic status files** that slowly converge into duplicate content. The proven project did exactly that
(a Hebrew owner status + an English "project-status.md"), discovered the redundancy, and **deleted the
duplicate** - folding its unique content into the AI-facing current-state. The lesson is baked in here: the
two docs are split by **audience and job**, not by language; never seed two interchangeable status files.

---

## 2. The role-boundary header (the anti-convergence mechanism)

Each file opens with a header that (a) states its own job in one line, (b) names its companion doc, and
(c) points at the siblings that own everything else. This is the single most load-bearing convention in the
module: a reader who lands in either doc immediately knows what this doc is for and where to go for the rest.

### Owner-facing header (`PROJECT_STATUS.md`)

The proven owner header opens with a "read this first" instruction and a map of the reading order, then a
note that this doc is kept lean with a rolling archive. Generalized (language-parameterized):

```markdown
# Project Status - <project>

> Read this file FIRST in every new chat / session. The concise source of truth for what is done
> and what remains.
> Live detail lives in: HANDOFF.md (warm-start for the current step) + docs/ai/current-state.md
> (AI-facing, detailed) + docs/TRACKER.md (deferrals/findings registry) + MEMORY.md (memory index).
> Full prior history (snapshot <date>): docs/archive/PROJECT_STATUS-<date>.md - the previous full
> narrative. This file was trimmed on <date>; keep it lean - when it grows, roll a new dated snapshot
> to the archive and leave only the live + forward-looking content here.

**Last updated:** <date>
```

### AI-facing header (`docs/ai/current-state.md`)

The proven AI header declares its audience, names every companion, and records the archive pointer plus what
was preserved verbatim there. Generalized:

```markdown
# Current State - <project>

> AI-facing detailed state. Companion to PROJECT_STATUS.md (owner-facing, concise) + HANDOFF.md
> (live warm-start) + docs/TRACKER.md (deferrals/findings) + MEMORY.md (index).
> Rewritten lean <date>. Full prior history (the accreted run-by-run header, the sealed outcomes, the
> earlier "what is done" narratives) is preserved verbatim in docs/archive/current-state-<date>.md.
> Keep this file lean; roll a dated snapshot to archive when it grows.

**Last updated:** <date> - <one-line "what changed this session">
```

The `Last updated:` line is **mandatory** on the AI doc and carries a one-line "what changed this session"
clause, so a returning chat sees at a glance whether the doc reflects the latest work or predates it.

---

## 3. Section shapes

These are the section skeletons the templates scaffold. Keep the headings stable across the project so a
returning chat finds the same anchors every time.

### `PROJECT_STATUS.md` (owner-facing)

- **Now** - the current focus in a few lines (one short paragraph + an optional bullet or two).
- **Next** - the immediate next step(s).
- **Blockers** - "none" or what is blocking, with the owner action if there is one.
- *(optional, as the project grows)* **TL;DR table** (current stage / source-of-truth doc + version /
  major subsystem statuses), a **USER ACTIONS REQUIRED** list (mirrors the tracker's user-action rows for
  owner visibility), a one-line **roadmap** table, a **repo map**, and a **new-chat reading order**.
  These earn their place only when they save the owner a question; until then, keep it to Now/Next/Blockers.

### `docs/ai/current-state.md` (AI-facing)

- **What is built** - the present capability set (include the AI-layer artifacts the bootstrap produced).
- **What is pending** - concrete, with enough detail that an implementer could pick it up.
- **Decisions / constraints (locked)** - the architectural rulings that must not be re-litigated; each as a
  one-line "decision -> rationale or the field/mechanism it pins." This section is the AI doc's highest-value
  content: it is where "we already decided X, do not re-invent it" lives durably.
- **Blockers / required user actions** - what is blocked and any human-in-the-loop action (cross-linked to
  the tracker's user-action rows; the tracker is the registry, this is the at-a-glance pointer).

---

## 4. Lifecycle: overwrite + rolling archive

### 4.1 Overwrite, do not append

These are **living docs**: each is rewritten to reflect *now*, the way you would update a whiteboard - not
appended to like a changelog or a ledger. History does not accumulate inside the live doc. A run-by-run
"what we did" log inside `current-state.md` is the classic way it bloats to 100KB+; resist it. Anything that
is genuinely historical and worth keeping goes to one of three places:

- a durable lesson -> a **memory** topic file (referenced from the always-loaded index),
- a deferral or a review finding -> a **tracker** `T-XXXX` row,
- the full prior narrative at a size milestone -> a dated **archive snapshot** (below).

### 4.2 The rolling-archive mechanic (the concrete steps)

When a doc approaches its size budget, do exactly this:

1. **Snapshot.** Copy the full current file verbatim to `docs/archive/<name>-<YYYY-MM-DD>.md`. The snapshot
   keeps its own original header (it is a frozen point-in-time copy); it is never edited again. Examples from
   the proven project: `docs/archive/PROJECT_STATUS-2026-06-05.md`, `docs/archive/current-state-2026-06-05.md`.
2. **Rewrite lean.** Rewrite the live doc to contain only the **live + forward-looking** content: now, next,
   blockers, the still-relevant locked decisions, and pointers. Drop the historical narrative - it is now in
   the snapshot and lost nothing.
3. **Re-point the header.** Update the live doc's header line to reference the new snapshot by date and state
   in one phrase what it preserved ("full prior history (snapshot <date>): docs/archive/<name>-<date>.md").
4. **Roll, do not overwrite the archive.** Each budget-exceed produces a **new** dated snapshot; older
   snapshots are never deleted or overwritten. The archive is an append-only series; the live doc is the only
   thing that gets rewritten. (A reader who wants the whole story walks the dated series backwards.)

The archive folder + the first snapshot are created **lazily** - on the first budget-exceed, not on day one.
The *convention* is documented from day one (in this header), but you do not pre-create an empty archive.

### 4.3 Size budget (the trigger)

Budget by **what loads cleanly into a session**, not a hard byte law. The empirical lesson: when these docs
grew to **~94KB (owner status) and ~127KB (AI state)** they only **partially loaded** - which is worse than
missing, because the reader does not know what they did not see. The proven rewrites took them to **~9-13KB**
(roughly a 10x trim). Practical guidance:

- **Target:** keep each doc in the low tens of KB - small enough to always load whole and to read in a couple
  of screens (owner) or a few (AI).
- **Trigger:** roll a snapshot when a doc has roughly **doubled** from its last lean rewrite, or whenever a
  whole section has become historical narrative rather than present state - whichever comes first.
- **Bias:** archive **lazily** (only at the budget, not on every edit) so the dated series stays meaningful
  and you are not snapshotting noise. Over-archiving fragments the history; under-archiving bloats the load.

### 4.4 Worked example (the proven roll, generalized)

> A session closes a major milestone. The AI doc has accreted a run-by-run header that now spans dozens of
> KB of "what we did in run N" narrative.
> **Step 1:** copy the file to `docs/archive/current-state-<date>.md` (the full narrative, frozen).
> **Step 2:** rewrite the live `current-state.md` to: the current milestone in a few paragraphs, what is
> pending, the locked decisions still in force, blockers, and the user-action pointer - dropping every
> historical run narrative.
> **Step 3:** set the header to "Rewritten lean <date>. Full prior history preserved in
> docs/archive/current-state-<date>.md." Bump `Last updated:`.
> **Step 4:** do the same for `PROJECT_STATUS.md` against `docs/archive/PROJECT_STATUS-<date>.md`.
> Result in the proven project: 127KB -> ~12.5KB and 94KB -> ~9.4KB, nothing lost (the snapshots hold the
> full history), and both docs load whole into the next session.

---

## 5. Doc hygiene (obey the shared rule, do not redefine it)

All long-form content in these docs follows the **semantic-line-break** rule owned by `doc-hygiene`: wrap at
sentence/clause boundaries (~100 chars), never one giant physical line (a single newline renders as a space,
so the output is identical but the source stays diffable and tool-loadable); never start a wrapped line with a
block trigger (`-` `*` `+` `>` `|` `#` `N.`); tables and code fences stay on one line. This is what keeps the
audit loop able to load and diff these docs at all - a single-physical-line blob defeats both. See the
`doc-hygiene` module; do not re-spell the rule here.

---

## 6. Counts are computed, never hand-edited

Any headline number these docs carry (a suite count, a coverage figure, a "N of M done") is a **computed
roll-up** of an underlying breakdown - a sum you can re-derive, not a figure you type and hope stays true.
Re-derive it when you update the doc rather than trusting the last hand-edit; a stale hand-typed total is a
quiet lie a returning chat will believe. (This is the same `computed roll-up` discipline the test-program and
tracker use - see the shared vocabulary; reference it, do not re-argue it.)

And: "done" in these docs is never a raw count. A milestone is done when its rows/findings are
**dispositioned** (closed or explicitly deferred with a `T-XXXX`), not when some number is reached - the
no-finding-dissolves rule, owned by the tracker.

---

## 7. Boundaries with the rest of the Project-Knowledge family

state-docs describes **the present state of the project** and nothing else. Each sibling owns a different
slice; state-docs **points at** them and never duplicates their job:

| If the content is... | it belongs in... | not in state-docs because... |
|---|---|---|
| a durable lesson / workflow rule / domain fact | **memory** (a topic file + the one-line index) | state-docs is present-state, not accumulated wisdom |
| a deferral or a review finding | **tracker** (`docs/TRACKER.md`, a `T-XXXX` row) | findings need a stable id + a ledger, not prose that gets overwritten |
| a chat-swap warm-start for the current step | **handoff** (`HANDOFF.md`, episodic, deleted after use) | the handoff is a transient resume brief, not a standing doc |
| how to write the prose itself (line breaks, leanness) | **doc-hygiene** | that is a cross-cutting rule, not project state |

Two boundaries worth stating sharply:

- **state-docs vs handoff.** The current-state doc is the *standing* answer to "what is built / decided /
  pending"; `HANDOFF.md` is the *episodic* answer to "how do I, a fresh chat, safely resume THIS step right
  now." The handoff is richer and more procedural and is **deleted after the next chat consumes it**; the
  state doc persists and is kept lean. The handoff *reads from* the state docs (for its pointers); the state
  docs never depend on a handoff existing.
- **state-docs vs tracker.** state-docs may *mirror* a short list of user-action items for owner visibility,
  but the **tracker is the registry** - the `T-XXXX` rows, their severity, source, and status live there.
  state-docs points at them; it does not become a second copy of the ledger.

The integrity rule that ties them together: **nothing lives only in chat.** Every decision worth keeping
lands in exactly one of these homes (state / memory / tracker), and state-docs points at the others so a
returning chat can always find the rest.
