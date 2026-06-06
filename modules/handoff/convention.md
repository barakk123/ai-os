# handoff - convention

The handoff is the **continuity / review-seam** of the AI-collaboration loop.
It is the one artifact that lets a *fresh chat with zero prior context* either
(a) **resume** a half-finished task, or (b) run an **independent adversarial review**,
without re-discovering everything by hand and without silently dropping work.

A handoff is NOT a summary. A summary tells the next chat what happened; a handoff tells it
**exactly where to stand, what to read first, what to NOT trust, and what to do next** -
and it is built so the next chat re-derives the work instead of inheriting this chat's
possibly-wrong conclusions.

It lives **untracked at the repo root** (e.g. `HANDOFF.md`). The next chat pastes its contents
(or a generated copy-paste prompt) as message #1, works, then deletes the file. It is never committed -
it is scaffolding for a transition, not project state. (Durable state goes to `state-docs`, durable
lessons to `memory`, deferrals/findings to the `tracker`.)

---

## 1. When a handoff fires (triggers)

**Owner-initiated.** Any phrasing that means "we're about to swap chats / the window is full":
"pack up", "let's prep a new chat", "let's do a handoff", "the window is full" (and the
project-language equivalents). Phrase the trigger list in the **project's chosen language**; keep an
English template - never hardcode one locale's words as the only triggers.

**Proactive - OFFER.** At a *closed + verified + committed* stopping point, the assistant should
**offer** to pack ("we're at a clean stopping point - want me to prepare a handoff before we continue
in a fresh chat?"). Offer, do not auto-run, when the context window still has comfortable headroom -
the owner may want to keep going in the same chat.

**Proactive - AUTO.** Run the pack **automatically** (then report) when the context window is
**nearly full**, because at that point the cost of *not* packing is a hard, silent context loss. Auto is
the safety net; the offer is the courtesy.

**Standing - the Per-Stage / Per-Domain Independent-Review gate (keystone).** At the completion of
EVERY stage AND EVERY domain of meaningful work, the assistant - **proactively, unasked** - prepares
the **independent-review** variant of the handoff and explicitly notifies the owner: *"the infrastructure
is ready for a fresh-chat full honest review of <stage/domain>."* This is integral and continuous, never
skipped. The external oracle (reviewer != author) is the core reliability mechanism of the whole program -
it catches the self-referential failures a green check cannot. This trigger is owned by `gates` (Gate 7);
the handoff module supplies the brief it produces.

### Offer vs auto - the policy in one table

| Situation | Action |
|---|---|
| Owner says a trigger phrase | Pack now (full 6 steps). |
| Closed + verified + committed point, window has headroom | **Offer** to pack; wait for a yes. |
| Context window nearly full | **Auto-pack**, then report what you did. |
| Stage / domain boundary reached | **Auto-prepare the independent-review variant + notify the owner** (Gate 7). |
| Mid-task, nothing closed, window fine | Do nothing (a handoff mid-thought just freezes a half-baked state). |

---

## 2. The 6-step pack protocol

Run these in order. Each step has a concrete "how", not just a label.

### Step 1 - Pre-commit alignment audit (reconcile prompts ↔ files)
Re-read **every** prompt (owner + assistant) since the last commit. Reconcile what was decided,
discussed, or deferred against what was **actually written to the files** - and verify by **reading the
files**, never by trusting your recollection of what you wrote. Fix or add anything missed *before*
packing. This is the `gates` Pre-Commit Alignment Audit; it is mandatory here because a handoff that
packs an un-reconciled state hands the next chat a lie. Report findings (fixes applied) and an explicit
"nothing else outstanding" when clean.

### Step 2 - Refresh the living docs (state-docs + tracker + memory)
Update the standing docs so the present they describe is actually the present:
- **state-docs** - the owner-facing status and the AI-facing current-state (what's done / pending /
  blocked / next). Point them at the next task.
- **tracker** - every open finding / deferral has a `T-XXXX` row with a current `last_updated`; any new
  finding from this run is recorded (no-finding-dissolves - nothing lives only in chat).
- **memory** - if a durable lesson or domain fact emerged this run, add it (one-line index entry + topic
  file). If a `[[wikilink]]`'d memory's guidance changed, update its topic file (never silently overwrite -
  append a dated lapse/reinforcement block).

This is exactly what `sync` does; the handoff just always does the *full* version of it before packing.

### Step 3 - Compress / re-index memory if it grew
`MEMORY.md` is the only auto-loaded memory file and it has a hard token budget - bloat means a silent
partial load, which is silent memory loss. If the index grew this run, enforce **one line per memory**
(`[Title](file.md)` + a one-sentence hook); push depth down into the topic files; if a topic file is over
budget, snapshot-and-archive it per the `memory` / `doc-hygiene` convention. A bloated index quietly breaks
the next chat's recall - keep it lean here.

### Step 4 - Write the warm-start (brief and/or copy-paste prompt)
Produce the artifact(s) the next chat will consume. Two flavors from one skeleton (see §5):
- **`HANDOFF.md` brief** - the full warm-start document at the repo root. Use for mid-task continuation
  where the next chat needs the whole picture, the binding-docs map, and the verify-the-baseline block.
- **`handoff-prompt.md` copy-paste prompt** - the same content compressed into a ready opening **message**.
  Use for a clean next-step where the next chat can act immediately.

**Decide which fits and recommend it** (see §6). Often you write the `HANDOFF.md` brief and a short
copy-paste prompt that *points at it* ("paste this, which tells you to open `HANDOFF.md` and re-derive").

Whichever you write, it MUST carry, prominently and near the top (not buried): the **anti-scope framing**
(§3) and the **two completeness gates** (§4) - and repeat the anti-scope caveat at the section headers
(file-map, seeds, prior conclusions) so it is reinforced at the point of consumption.

### Step 5 - Commit the pack
A proper pack commit using the project's commit convention (the deferred `git-dev-push` module; a plain commit until it ships): summarize
the changes since the last push, stage, commit, push, confirm CI is green where it applies.
**Do NOT commit the `HANDOFF.md` / prompt file** - it is untracked transition scaffolding. Commit the
*state-docs / tracker / memory* refresh from Steps 1-3. Commit and push only when the owner has asked, or
per the project's commit policy.

### Step 6 - Notify the owner
Tell the owner the handoff is ready, in one line, with the headline: which variant
(continuation / independent-review), the file name, and - for the review variant - the explicit
*"ready for a fresh-chat full honest review of <stage/domain>"* phrasing the keystone gate requires.

---

## 3. The anti-scope framing (mandatory in every brief)

This is the single most important thing a handoff does, and the failure it prevents is subtle:
a thorough-looking handoff makes the next chat *feel* that "everything is already prepared, there's
nothing left to investigate." That feeling is the trap. **Detail is not completeness.**

State explicitly, near the top AND at every seed-bearing section header:

> The file-map / scenario seeds / prior conclusions / row-list in this brief are a **PARTIAL warm-start -
> what the previous chat happened to notice - NOT the scope.** Re-derive the full work list **from
> scratch** from the source-of-truth (the spec, the derived docs, the schema, the code itself) **as if
> this brief did not exist.** Only THEN cross-check against the seeds here - the seeds catch *your* misses;
> they do not bound the work and do not replace reading the sources. You WILL find files, sections,
> functions, edges, states, settings, and findings that are not listed here, and you are expected to.

Frame it as **two independent derivations** (a blind derivation from the sources, then a cross-check
against the seeds) and require a **delta presented for approval before acting**.

**The tripwire, stated in the brief:** *"if your final work-list looks like a copy of the seeds in this
brief, you under-investigated."* Put the tripwire in the brief itself so the next chat self-checks.

For the **independent-review** variant there is one more clause: the brief must **NOT hand the author's
conclusions over as truth.** The matrix / counts / "what's done" / oracle values in the brief are
*claims by the chat that wrote it*. The reviewer re-derives the expected behavior **from the
source-of-truth, never from the code or from this brief**, and reads the code only to check status, never
to decide what *should* happen. (This is the external oracle - see `gates` Gate 7 and the OS thesis:
green proves "the code does X", not "X is the intended Y"; only re-derivation by a non-author catches the
gate-passing failure.)

---

## 4. The two completeness gates (mandatory, never conflated)

These are two **separate, visible, reported** steps. They are not the same as the up-front derivation, and
they must never be merged into it or assumed-clean. (A real lapse: a run once labeled "two derivations" as
if it had run these gates - it had not. Saying a step ran when it did not is the failure. Name both gates
explicitly and report each one's result, even when the result is "nothing missed / no back-fill needed.")

**Gate I - second completeness pass (BEFORE writing / acting).**
After the first derivation, re-derive **again from a different angle** - e.g. walk every *sentence* of the
relevant spec section and every settings value and confirm each maps to at least one item in the work-list.
Report it explicitly: "second pass found X / nothing missed", and expand the list with whatever it catches.

**Gate II - backward retro sweep (AFTER the forward pass).**
Re-examine the **already-covered / already-done** areas for interaction edges that *later* work created in
them but were never back-filled. The check is **bidirectional**: covering a new area also means
re-examining the priors for now-reachable cases. The first such sweep carries the whole accumulated
backlog (largest); afterwards each run only reconciles the priors against what was added since, so it stays
small. Close cheap gaps in-run; log the rest to the tracker. Report the result even when it is "no back-fill
needed."

Both gates appear as explicit checkboxes in the handoff template so the next chat cannot skip them silently.
They are defined here once and **referenced verbatim** by `test-program` (the per-domain run process) and by
`gates` (Gate 7) - do not fork or re-define them elsewhere.

---

## 5. The handoff skeleton (one shape, two modes)

Both variants share one skeleton; the **mode** changes the role line, the deliverable, and whether the next
chat may edit. Build from `templates/HANDOFF.template.md`. The skeleton (core sections numbered; the lettered
sections are the **continuity-machinery** inserts of §5a, present when the run state needs them):

1. **Title + role line** - what this chat IS (continue task X / independently review Y) in one line.
2. **The anti-scope box** (§3) - prominent, near the top.
3. **The two-completeness-gates box** (§4) - as visible checkboxes.
4. **Where we are** - the arc (newest last): the position reached, in 3-6 numbered lines.
   - **3a. Checkpoint != boundary** (§5a) - the per-batch commits are checkpoints; the review gate has NOT fired.
5. **What just landed** (continuation) / **What to review** (review) - with "VERIFY, don't trust".
6. **What remains / next step(s)** - the immediate work, ordered.
   - **5a. Stale artifacts - reconcile at start** (§5a) - the living docs left mid-reconcile; regenerate them.
7. **Source of truth - re-derive from these, not from this brief** - pointers into the *one* precedence
   ladder (owned by `source-of-truth`), never a private ordering. The oracle is the spec, never the code.
8. **Open decisions / sealed owner rulings (do NOT re-ask)** - the decisions already made, recorded so the
   next chat does not reopen them (an owner-decision is recorded, not re-asked).
   - **7a. Owner-approved deferrals** (§5a) - reason-bound, settled-later, do NOT re-ask; distinct from findings.
9. **Open findings (tracker; none may dissolve)** - the live `T-XXXX` rows for this area.
10. **File / artifact map** - service / routes / schema / migrations / tests / docs - labeled PARTIAL.
11. **Domain character** - which test types / aspects are heavy vs N/A here (review/test runs).
12. **Seeds** - the non-exhaustive scenario / row seeds, **labeled "non-exhaustive, NOT scope"** at the header.
13. **Binding-docs / read-first map** - the exact files + memory entries to read before acting, in order.
14. **Verify the baseline yourself (don't trust this brief)** - the exact commands to re-run (suite / tsc /
    CI / mutation), with expected results, so the next chat confirms the starting state first-hand.
15. **Commit + PARTIALLY-DONE state** (§5a) + standing gates - which half sits at which HEAD, what is loose
    in the working tree (owner commits when ready), the clean/known-tree note, which gates apply, push policy.

### 5a. Continuity machinery (the four mid-task-state inserts)

A handoff that fires *mid-arc* (not at a clean closed boundary) must carry four extra pieces, or the next
chat silently corrupts the state. These are the difference between a real continuation brief and a tidy
summary; include each one whenever the run state calls for it (omit, with a one-line "n/a", when it does not):

- **Stale-artifacts / reconcile-at-start** (template §5a). A run often leaves *living* tracking docs mid-update:
  a per-row ledger/matrix whose `- [ ]` boxes lag because earlier batches tracked completion in a progress
  narrative instead of flipping boxes, or a test-index/count doc at a stale number. Name each, give its current
  stale value, and say **"regenerate from `<source>` (e.g. the test enumerator); do NOT rebuild already-done
  rows."** The danger this kills: the next chat re-builds finished work because a checkbox lagged reality.
- **Checkpoint != stage boundary** (template §3a). Per-batch commits are *checkpoints* - a green suite + a
  pushed commit means "saved", not "done". State plainly that the **review gate has NOT fired** and the next
  chat must NOT declare the domain done or scale the pattern until the full-closure path + a fresh-chat
  independent review are complete (the Per-Stage gate fires at the *domain boundary*, never at a checkpoint).
  "Done" is never a built-count or a green run.
- **Partially-done state model** (template §14). When part of the run is committed + pushed + CI-green at one
  HEAD while a *later* sub-batch is built-but-uncommitted in the working tree, the brief must spell out
  **exactly which half sits at which HEAD**: list the modified (M) and new (NEW) files of the loose half, note
  whether it is verified-green and whether it touches gated/source code (so the next chat knows which bars
  still hold), and say **"the owner commits when ready"**. The next chat must NOT assume a clean tree and must
  NOT rebuild the loose half.
- **Owner-approved-deferrals slot** (template §7a). Kept **distinct from open findings**: an open finding still
  needs work; an approved deferral is *settled work, scheduled later*, bound to a reason ("inherently
  red-against-current-code and entangled with a later fix"; "the feature is not built yet"; "needs a param the
  code lacks until step e"). Record item -> target step (reason) + date. The next chat must **not** re-ask it
  and must **not** re-flag it as "missing"; if it believes one should move sooner it *raises* it, never
  silently re-adds it to the work-list. Conflating the two makes the next chat either reopen a closed question
  or drop a real gap.

### Mode differences

| | Continuation handoff | Independent-review handoff |
|---|---|---|
| Role line (§1) | "continue task X from the exact point reached" | "honest, adversarial, INDEPENDENT review of Y - do NOT rubber-stamp" |
| Edit rights | edits + writes code | **READ-ONLY**: report findings to a new review doc; corrections happen back in the origin chat |
| Author's conclusions | trusted as the prior state (still re-derive forward) | **explicitly NOT trusted** - re-derive everything from source-of-truth |
| Deliverable (§6 of skeleton) | continue building | a categorized findings doc + a **verdict** (sound foundation? or what must be fixed first) |
| Who consumes the output | the same loop, next batch | findings flow back to the TRACKER (no-finding-dissolves), then fix/defer in the origin chat |

The review variant additionally specifies **finding categories** (e.g. missing / wrong-oracle /
wrong-status / missing-genre / structural / invented / spec-ambiguity), each with **severity**
(blocker / high / med / low) + **evidence** (the ref + what-the-spec-says) + a **recommended correction**,
and ends with an explicit **verdict**. Recommend running parallel independent sub-reviews from different
angles (completeness vs oracle-correctness vs genre-coverage) so one reviewer's blind spot is covered.

---

## 6. Brief vs copy-paste prompt - which to write

Both carry the same anti-scope + two-gates content; they differ in **shape and entry cost**.

**Write the `HANDOFF.md` brief when** the next chat needs the whole picture before it can act safely:
mid-task continuation, a large file/docs map, a binding-docs map, a verify-the-baseline block, many open
findings. The brief is a *document the next chat reads*.

**Write the `handoff-prompt.md` copy-paste prompt when** the next step is clean and self-contained and the
next chat can start acting from a single pasted message: a discrete next task, an independent review with a
short source-of-truth list. The prompt is a *message the next chat is given as its first turn*.

**The common, recommended pattern:** write the **`HANDOFF.md` brief** (the full document) **and** a short
**copy-paste prompt that points at it** - the prompt is what the owner pastes; it tells the next chat to
open `HANDOFF.md`, re-derive from the source-of-truth, run the two gates, and confirm the baseline before
acting. This gives the owner a one-paste hand-off while keeping the depth in the file.

Whichever you produce, **recommend which fits** in the Step-6 notification so the owner is not left to guess.

---

## 7. Relationship to `sync`

`sync` ("synchronize") runs the *refresh* subset only - Steps 1-3 (audit + refresh state-docs / memory /
tracker) - **without** writing the brief, doing the pack commit, or implying a chat-swap. Use `sync` often
(it is cheap, keeps the docs honest). Use the full handoff at a real stopping point or a chat-swap. The
boundary is owned by `sync`'s convention; it is referenced here so the two never blur:

| | sync | handoff |
|---|---|---|
| refresh state-docs / memory / tracker | yes | yes (Step 2-3) |
| pre-commit alignment audit | light | full (Step 1) |
| warm-start brief / copy-paste prompt | no | yes (Step 4) |
| pack commit + notify | no | yes (Steps 5-6) |
| intended for a chat-swap / fresh-chat review | no | yes |

---

## 8. Cross-references (reference, never redefine)

- **The external oracle / "green != correct-to-intent"** - the OS thesis; owned by `gates` (Gate 7).
  This module supplies the brief that gate produces; it does not re-argue the principle.
- **The precedence ladder** ("re-derive from the source-of-truth, not summaries") - owned by
  `source-of-truth`. Every brief's "Source of truth" section points into that one ladder.
- **The two completeness gates** - defined here (§4); referenced verbatim by `test-program`'s
  per-domain run and by `gates` Gate 7. Do not fork them.
- **no-finding-dissolves + `T-XXXX`** - owned by `tracker`; review findings route
  there, nothing lives only in chat.
- **Semantic line breaks + lean / rolling-archive + the one-line memory index** - owned by
  `doc-hygiene` / `state-docs` / `memory`; every doc this module writes obeys them.
- **The pack commit convention** - owned by the (deferred) `git-dev-push` module.
