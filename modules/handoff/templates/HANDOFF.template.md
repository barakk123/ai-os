<!--
  HANDOFF.template.md - the warm-start brief for a fresh chat.
  ONE skeleton, TWO modes: a CONTINUATION handoff and an INDEPENDENT-REVIEW handoff.
  Pick a mode at section 1; delete the unused mode's lines where they diverge (marked [CONT] / [REVIEW]).

  USAGE: copy to the repo root as `HANDOFF.md`, fill the <...> placeholders, leave it UNTRACKED.
  The next chat pastes it (or a generated copy-paste prompt that points at it) as message #1,
  works, then deletes the file. NEVER commit this file - it is transition scaffolding, not state.

  Obey doc-hygiene: semantic line breaks (~100 chars, clause boundaries), no block-trigger as a
  wrapped line's first char, tables/fences stay one line.
-->

# HANDOFF - <CONT: continue \<task\> from the exact point reached  |  REVIEW: independent review of \<artifact\>>

> **Your role in THIS chat:**
> [CONT] continue `<task>` from where it stands - read this brief + the binding-docs map + the memory
> index, re-derive the remaining work from the source-of-truth, then build.
> [REVIEW] an honest, adversarial, **INDEPENDENT** review of `<artifact>`. Do NOT rubber-stamp. Find what
> is missing / wrong / weak / mis-classified / invented. **READ-ONLY** - report findings to a new doc
> `<review-doc-path>`; do NOT edit the artifact or write code (corrections happen back in the origin chat).

---

## 1. ANTI-SCOPE FRAMING (non-negotiable - read before anything else)

The file-map, scenario seeds, prior conclusions, and work-list in this brief are a **PARTIAL warm-start -
what the previous chat happened to notice - NOT the scope.** Re-derive the full work list **from scratch**
from the source-of-truth (§7) **as if this brief did not exist**, then cross-check against the seeds here.
The seeds catch *your* misses; they do not bound the work and do not replace reading the sources. You WILL
find files, sections, functions, edges, states, settings, and findings that are not listed here, and you
are expected to.

Frame it as **two independent derivations** (blind derivation from the sources, then a cross-check against
the seeds), and **present the delta for approval before acting** (or, for review, before finalizing).

**Tripwire:** if your final work-list / findings look like a copy of the seeds in this brief, you
under-investigated.

[REVIEW] **Do NOT trust this brief or the artifact as truth.** Every count / "what's done" / oracle value
here is a *claim by the chat that wrote it*. Re-derive expected behavior from the **source-of-truth, never
from the code or from this brief**; read the code only to check status, never to decide what *should*
happen. (The external oracle: a green check proves "the code does X", not "X is the intended Y".)

## 2. THE TWO COMPLETENESS GATES (run BOTH, visibly, report each result)

- [ ] **Gate I - second completeness pass** (BEFORE writing / finalizing): re-derive again from a different
  angle - walk every *sentence* of `<spec section>` + every settings value and confirm each maps to >=1
  work item. Report "found X / nothing missed" and expand the list.
- [ ] **Gate II - backward retro sweep** (AFTER the forward pass): re-examine the already-covered / done
  areas for interaction edges that later work created in them but were never back-filled. Bidirectional.
  Close cheap gaps in-run; log the rest to the tracker. Report the result even when it is "no back-fill needed".

These are **separate, visible, reported** steps - never merged into the up-front derivation, never
assumed-clean. Saying a gate ran when it did not is the failure.

---

## 3. Where we are (the arc, newest last)

1. <the broader context / why this work exists>
2. <the prior milestone>
3. <the most recent landing>
4. <the exact point reached - and explicitly: what is NOT done yet>

## 3a. CHECKPOINT, not stage boundary (do NOT declare the domain done / do NOT scale)

[CONT] **The per-batch commits in this run are CHECKPOINTS, not stopping points.** A green suite + a pushed
commit at a batch boundary means "this slice is saved", NOT "the domain is done". **The review gate has NOT
fired** - <the named closure step(s), e.g. step (e) remediations + (f) the coverage bar + the full-closure
independent review> still remain. Do **NOT** declare `<domain>` complete, do **NOT** start the next domain,
and do **NOT** scale the pattern to other domains until the **full-closure path is complete AND a fresh-chat
independent review has run** (the Per-Stage / Per-Domain Independent-Review gate fires at the *domain
boundary*, not at a checkpoint). "Done" is NEVER a built-count or a green run - it is the closure path
finished + the genre/work ledger dispositioned + the coverage bar met + the external review passed.

(If this handoff IS at a real stage/domain boundary, say so and switch to the REVIEW mode - the independent
review fires now, unasked. If it is mid-arc, this guard keeps the next chat from mistaking a checkpoint for a
finish line.)

## 4. [CONT] What just landed (VERIFY, don't trust)  |  [REVIEW] What to review

[CONT]
- <what landed and HOW it was verified - the command + the result, not just "done">
- <any gated / sensitive change + who approved it>
- <any new finding raised this run -> its T-XXXX>

[REVIEW] Categorize your findings by these:
1. **Completeness (missing items):** every spec clause / edge / policy branch / boundary / settings combo /
   error code -> an item in the artifact? List what is missing.
2. **Oracle correctness (wrong expected value):** is each item's expected behavior what the **spec** says
   (not what the code does)? A wrong oracle is worse than a missing item.
3. **Status accuracy:** spot-check any "current-status" claims against the ACTUAL code/tests - mis-labels?
4. **Genre / aspect coverage:** are all applicable kinds represented (per `<taxonomy doc>`)? Any thin/absent?
5. **Structural / bar-readiness:** are items fine-grained enough to actually verify? Any too vague? Any
   spec ambiguity that blocks deriving an oracle (flag as an owner-decision item)?
6. **Invented / unanchored:** any item asserting behavior with no anchor in the spec.

## 5. [CONT] What remains - next step(s)  |  [REVIEW] Your deliverable + verdict

[CONT]
- <the immediate next action(s), ordered>
- <what is BLOCKED on (infra / a decision) vs buildable now>

[REVIEW]
- Write `<review-doc-path>`: findings categorized (missing / wrong-oracle / wrong-status / missing-genre /
  structural / invented / spec-ambiguity), each with **severity** (blocker/high/med/low) + **evidence**
  (the ref + what-the-spec-says) + a **recommended correction**.
- End with a **verdict**: is `<artifact>` a sound, complete, oracle-correct foundation - or what must be
  fixed first? Consider parallel independent sub-reviews from different angles (completeness vs
  oracle-correctness vs genre-coverage) so one blind spot is covered.

## 5a. ⚠️ STALE ARTIFACTS - RECONCILE AT START (regenerate; do NOT rebuild done rows)

The previous chat left these LIVING artifacts mid-reconcile or at a stale count. They are *tracking* docs,
not work to redo - reconcile them against their source FIRST, before deriving or building anything, so each
becomes a reliable remaining-work list. **The danger is rebuilding already-done rows because a checkbox/count
lags behind reality.**

- **<per-row ledger / matrix>** is STALE: <why - e.g. earlier batches tracked completion in the progress
  narrative, NOT by flipping its `- [ ]` boxes, so most rows it shows "open" are ALREADY BUILT>.
  **Reconcile:** cross-ref <the progress doc's "built so far" list / the SE-* IDs per sub-batch> + `<the
  authoritative enumerator, e.g. vitest list>` against the ledger, flip the boxes for what exists, build only
  the genuinely-absent rows. **Regenerate from `<source>`; do NOT rebuild already-done rows.**
- **<test index / count doc>** is at `<stale number>` (pre-`<last batch>`): regenerate it from `<the
  generator, e.g. vitest list>` so the global per-test index is current - <+N this/last arc>.
- <any other living doc the prior chat left mid-update: a coverage table, a survivor-triage list, a status
  anchor> - state the stale value + the source to regenerate from.

(If nothing is stale, say so explicitly: "no stale artifacts - all living docs current as of `<HEAD>`.")

---

## 6. Source of truth - re-derive from THESE, not from this brief

In precedence order (per the project's one precedence ladder - the lower layers explain HOW, not WHAT):
- <master spec + the exact sections that govern this work>
- <the relevant derived domain doc(s)>
- <schema / api-spec / error-code / status-machine sections>
- <the code, ONLY to check status - never to decide what SHOULD happen>

The oracle is the spec, never the code. If two sources disagree, STOP and surface it (the master spec wins).

## 7. Open decisions / sealed owner rulings (do NOT re-ask)

- **OD-<n>** <the decision> - <ruled on date>. <one-line rationale>.
- <... record every sealed ruling so the next chat does not reopen it>

## 7a. Owner-APPROVED deferrals (reason-bound, tracked - do NOT re-ask, do NOT re-derive)

Distinct from open findings (§8): these are items the owner has **explicitly approved deferring** to a named
later step, each bound to a reason. They are NOT gaps to re-flag and NOT decisions to reopen - the next chat
must treat them as settled and NOT re-surface them as "missing". Each carries: the item, the target step, the
**reason** it was deferred, and the date approved.

- **<item / row id> -> <target step>** (owner-approved <date>): <the reason - e.g. it is inherently
  red-against-current-code and entangled with a later fix, so building a throwaway pin then ripping it out is
  wasteful; or the feature it tests is not built yet; or it needs a param the code lacks until step e>.
- <... every reason-bound deferral; if a later chat thinks one should move sooner, it RAISES it, it does not
  silently re-add it to the work-list>.

(Keep this separate from §8. An open finding still needs work; an approved deferral is *settled work,
scheduled later* - conflating them makes the next chat either re-ask a closed question or drop a real gap.)

## 8. Open findings (tracker; none may dissolve)

- **T-<xxxx>** (status) <one line> - <where it is tracked / what it blocks>.
- <... the live findings for this area; each must map to a tracker row, never live only here>

## 9. File / artifact map (PARTIAL - you WILL find more)

- Source: `<service / module file(s)>`
- Routes / API: `<route file(s)>`
- Schema / migrations: `<schema + migration file(s)>`
- Tests: `<existing test file(s)>`
- Docs under work/review: `<doc(s)>`

(Labeled PARTIAL on purpose - this is what the prior chat happened to touch, not a full inventory.)

## 10. Domain character (what is heavy vs N/A here)

- Heavy: <the aspects / test kinds that dominate this area>
- Light / N/A: <what does not apply, and why>

## 11. Seeds (NON-EXHAUSTIVE - NOT scope; cross-check after your own derivation)

- <scenario / row / edge seed>
- <... a quick-scan starter list; if your final list looks like a copy of this, you under-investigated>

## 12. Binding-docs / read-first map (in order)

- <the memory index + the relevant memory entries>
- <CLAUDE.md + the relevant path-scoped CLAUDE.md>
- <the program-context docs (the why / the taxonomy / the plan)>
- <the per-row work-list / progress / matrix docs, if any>
- <the spec sections from §6>

## 13. Verify the baseline yourself (don't trust this brief)

Re-run and confirm the starting state first-hand BEFORE building/reviewing:
- `<test command>` -> expect `<result>`.
- `<typecheck command>` -> expect clean.
- `<CI check command>` -> confirm green on `<HEAD>`.
- `<mutation / coverage command, if applicable>` -> expect `<bar>`.
- <re-derive any count yourself; do not trust a number printed in this brief>.

## 14. Commit + PARTIALLY-DONE state (exactly which half sits at which HEAD)

State the split precisely so the next chat knows what is saved vs what is loose in the working tree, and at
which commit each half sits. **This run may be partially done: one half is committed + pushed + CI-green at a
HEAD; a later sub-batch is built-but-uncommitted in the working tree** - the next chat must NOT assume the
tree is clean, and must NOT re-build the loose half.
(If the tree IS clean at this handoff - everything committed - say so in one line here and skip the split below.)

- **Committed + pushed + CI-GREEN** at **HEAD = `<hashX>`** on `<branch>` (CI run `<id>`): <what landed there -
  the first sub-batch + any docs-refresh commit on top>.
- **Built-but-UNCOMMITTED in the working tree** (the later sub-batch - **the owner commits when ready**):
  - Modified (M): `<exact/file/a>`, `<exact/file/b>`, `<exact/file/c>`.
  - New (NEW / untracked, part of this work): `<exact/new/file>`.
  - This loose half is **`<tests-only / no engine source change>`** so `<the relevant bar, e.g. the mutation
    bar, holds>`; it is **`<verified green - the command + result>`** but **not yet committed**.
- **Unrelated untracked, do NOT commit:** `<scratch notes / unrelated working files - leave them>`.
- When committing the loose half: <branch policy - e.g. commits direct-to-`main`, CI runs on push>; run the
  pre-commit alignment + implementation audits; push, then confirm CI <phases> GREEN before declaring done.

## 15. Standing gates (do NOT skip)

- **Anti-scope (§1) + the two completeness gates (§2)** apply to this whole run.
- **Checkpoint != boundary (§3a):** do not declare the domain done or scale at a per-batch checkpoint.
- **No-finding-dissolves:** every finding -> a tracker `T-XXXX` + the relevant ledger; nothing lives only in
  chat. Keep §7a (owner-approved deferrals) distinct from §8 (open findings).
- The project's standing gates apply: pre-commit alignment audit, implementation/plan-completeness audit,
  regression-coverage (where the harness exists), don't-declare-working-before-confirmed, negative-testing
  (assert the specific failure code), consult-before-edit, No-Invented-Business-Logic.
- [REVIEW] This is READ-ONLY: report, do not fix. Reviewer != author - findings return to the origin chat.
- Commit/push only when the owner asks (or per project policy); confirm CI green before declaring done;
  any sensitive/gated change needs its named approver(s).
