# test-program

> Build a test program that proves the artifact does what the truth-docs INTEND, not merely
> what its own checks assert. Two tiers: a cheap `lite` floor every project ships, and a `full`
> program (genre taxonomy, mutation/branch bars, master matrix, prod-safety, owner-acceptance)
> for high-stakes / AI-authored codebases.

**Status:** since v0.1  ·  **Tier:** `lite` (floor) | `full` (opt-in for high-stakes / AI-authored code)

**Dependencies:**
- `standalone` — the lite tier needs nothing but a test runner.
- Soft: `source-of-truth` — the oracle derives every expected value from the precedence ladder, never
  from the code. Without a precedence-ordered spec the oracle has nothing trustworthy to derive from.
- Soft: `gates` — installing this module ACTIVATES the otherwise-dormant Regression-Coverage gate and
  arms the Per-Stage Independent-Review gate (the external oracle that this module was empirically proven by).
- Soft: `tracker` — the master matrix, the findings register, and the "done" ledger all feed the
  TRACKER, the owner of `T-XXXX` + no-finding-dissolves.
- Soft: `handoff` — this module triggers and consumes the per-stage independent-review brief.

<!-- ai-os:manifest
version: 0.1.0        # bumps (semver) when this module's content changes; absence is fine (older installs have none)
tier: heavy           # core | heavy
deps: { hard: [], soft: [source-of-truth, gates, tracker, handoff] }
tiers: [lite, full]   # [single]  OR  [lite, full]  OR named e.g. [authored-spec, external, hybrid]
-->

---

## What it gives you

The single thesis (the spine of the whole OS): **the source-of-truth doc is the oracle; the code is
never the oracle.** A green check proves "the artifact does what the check asserts," not "the artifact
does what intent requires." The gate-passing, dangerous failure is **failure-mode-B** — a check that
mirrors the code instead of the truth-doc. The only reliable catch is an **external oracle**: a reviewer
who re-derives expected behavior from the spec WITHOUT reading the implementation. This module turns that
thesis into a runnable program.

**lite** (the floor — worth it almost everywhere):
- The external-oracle rule + spec-independent expected values.
- **Spec-derived scenario-list-first** — enumerate every rule / edge / limit / toggle / gate as a
  reviewable row BEFORE writing a single test.
- **Negative-testing with specific asserts** — the things that MUST fail, asserting the EXACT failure
  (status + machine code / SQLSTATE), never "an error occurred".
- The **prod-safety invariant** — tests can never touch production (a tested, fail-closed guarantee).
- The **per-domain run process** (incl. the second completeness pass + backward retro sweep).

**full** (lite plus, for an owned / AI-authored product):
- The complete **genre taxonomy** (~19 lite types → the ~149-genre universe in categories A–K) +
  a per-domain **genre-applicability ledger** (every genre dispositioned: covered / partial / scoped-out
  with a written reason — silence is a gap).
- A **master matrix**: every (spec rule × applicable genre) → a row with a status and an oracle.
- **Mutation tooling** — the empirical "did we actually test it" bar (DB-free + DB-backed scopes, merged).
- **Reference-domain-first** — drive ONE domain to full closure + an independent review as the proven
  template BEFORE scaling N×.
- The **3-phase delivery model** + ratified completeness **bars** ("done" is never a test count).
- **no-finding-dissolves** wired to the TRACKER.

## Install

Paste into the project's `CLAUDE.md` (pick the tier; `full` for high-stakes / AI-authored code):

> ### Test Charter (tier: lite | full)
> The source-of-truth doc is the oracle, never the code: every expected value is derived from the spec,
> never snapshotted off current output (that ratifies the code as the oracle). Before writing tests for
> a domain, first enumerate a spec-derived scenario list for review. Every must-fail path asserts the
> SPECIFIC failure (status + machine code / SQLSTATE). Tests can NEVER touch production (a tested,
> fail-closed guarantee).
> (full) Cover every applicable genre × scenario (a written genre-applicability ledger — silence is a
> gap); meet the mutation + branch bars with zero un-triaged survivors; drive ONE reference domain to
> closure with an independent review before scaling; "done" is the ledger + the bars, NEVER a test count.

## Generates in target

- A "Test Charter" section in `CLAUDE.md`.
- (lite) a `scenario-list.template.md` per-domain checklist.
- (full) a `genre-taxonomy.md` reference, a master-matrix doc, an interaction-matrix doc, mutation-tooling
  config pointers, and a per-domain genre-applicability ledger.
- Activates the Regression-Coverage gate (marked dormant until the harness lands).

## Files it scaffolds

- `templates/genre-taxonomy.md` — the full genre universe (categories A–K, what each catches).
- `templates/scenario-list.template.md` — the per-domain spec-derived scenario list.
- `templates/interaction-matrix.template.md` — the cross-domain runtime edges (+ the scope taxonomy).
- `templates/mutation-tooling.md` — how to wire DB-free + DB-backed mutation runs and merge them.
- `templates/owner-acceptance.template.md` — the owner-legible, signable acceptance checklist / Gherkin
  form that closes oracle 2 (truth-docs ↔ owner intent).

## Why

On the proven project, ~1600 tests were written and green — then found to have been verified
SELF-REFERENTIALLY (the same agent read the docs, derived the scenarios, wrote the tests, AND reviewed
them: no external oracle). A single independent review instantly surfaced a green test that asserted the
OPPOSITE of a sealed decision (failure-mode-B), plus whole genres missing (mutation / property / invariant
/ adversarial-security / perf / owner-acceptance) — and the hardest logic (the core engine) was the LEAST
covered. Retrofitting trust into ~1600 green tests is expensive. Designing the charter in on day one —
the oracle rule, scenario-list-first, the genre taxonomy, a mutation backstop, reference-domain-first,
and the external-oracle review gate — is the cheap version, and it is what closes the artifact↔truth-doc
oracle that CI alone never can.
