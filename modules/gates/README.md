# gates

> The standing review disciplines that fire on meaningful work - each shipped with its trigger,
> its step-by-step operational procedure, and the rationale that makes it stick.

**Status:** since v0.1
**Dependencies:** `standalone`.
Soft-deps:
`tracker` (every gate that catches something routes the finding there - no-finding-dissolves);
`agent-fleet` (the gated workflow runs the gates; gates assert on each role's Output Contract);
`source-of-truth` (every "re-derive, don't trust summaries" step appeals to the precedence ladder);
`handoff` (the independent-review gate produces a handoff brief and reuses its anti-scope framing +
two completeness gates - referenced, never re-defined);
`test-program` (enables the conditional gates: regression-coverage + the empirical "done" ledger).

<!-- ai-os:manifest
version: 0.1.0        # bumps (semver) when this module's content changes; absence is fine (older installs have none)
tier: core            # core | heavy
deps: { hard: [], soft: [tracker, agent-fleet, source-of-truth, handoff, test-program] }
tiers: [single]       # [single]  OR  [lite, full]  OR named e.g. [authored-spec, external, hybrid]
-->

---

## What it gives you

A small set of **always-on disciplines** woven into the work loop. A gate is not a vibe - it is a
named protocol with five parts: a **trigger boundary** (the exact moment it fires), an explicit
**procedure** (the literal steps to walk), a **dormancy clause** (if it depends on infra not yet
built), a **provenance stamp** (it was added because a real failure cost real time), and a
**reporting contract** ("report findings, with an explicit 'nothing outstanding' when clean").

**The five universal gates** (every project, day one - no extra infra needed):

1. **Pre-Commit Alignment Audit** - reconcile prompts-since-last-commit against what actually
   landed in the files, before every commit.
2. **Implementation / Plan-Completeness Audit** - at item/milestone completion, audit the whole
   item's artifacts against the full plan + sealed decisions + existing-mechanism consistency.
3. **Consult-Before-Edit** - summarize the planned spec/doc change and get a green light before
   Write/Edit.
4. **Don't-Declare-Working-Before-Confirmed** - never "fixed/works" for CI/build/deploy until GREEN
   in the real target env AND owner-confirmed.
5. **Negative-Testing** - test the things that should FAIL and assert the SPECIFIC failure.

**The two test-coupled gates** (their empirical teeth sharpen once `test-program`'s harness lands):

6. **Regression-Coverage Gate** - work touching a closed domain re-runs its suite + back-fills
   interaction edges, bidirectionally (the backward retro sweep). Fully **dormant** until the suite
   + CI exist - it has nothing to re-run before then.
7. **Per-Stage / Per-Domain Independent-Review Handoff Gate** - the keystone: at every
   stage/domain boundary, proactively (unasked) ready the infra + prepare a fresh-chat independent
   review (the external oracle: reviewer != author). The review **discipline is universal** - it
   fires on meaningful work from day one (re-derive from the spec, re-read the artifacts); only its
   **empirical teeth** (re-running a suite, mutation/coverage checks) are conditional on the harness.

See `convention.md` for each gate's full procedure, rationale, and a worked example.

## Install

Paste into the project's `CLAUDE.md` (the governance layer). Keep the inline rationale - the
"established because X cost Y" stamp is what makes a tired agent at 2am actually run the gate
instead of skipping it.

> ### Standing Gates
>
> Meaningful work = changes code, schema, API, or domain behavior. On meaningful work, these
> disciplines are always on. Each one: run the procedure, then report findings with an explicit
> "nothing outstanding" when clean. A finding that needs later work becomes a `tracker` row -
> nothing dissolves in chat.
>
> - **Pre-Commit Alignment Audit** (before every commit) - re-read every prompt since the last
>   commit; reconcile what was decided / deferred against what was actually written (READ the
>   files - do not trust recall); fix gaps first. *Added after a commit dropped an agreed decision
>   and another shipped an inaccurate "all categories" claim.*
> - **Implementation / Plan-Completeness Audit** (at item/milestone completion) - audit the whole
>   item's artifacts vs the FULL plan + sealed decisions; check (a) every planned element landed,
>   (b) nothing weakened, (c) it reuses existing mechanisms (don't invent a divergent pattern).
>   *Distinct from the pre-commit audit: whole-item scope, not commit-delta.*
> - **Consult-Before-Edit** - before any spec/doc Write/Edit, output "here is what I'm about to
>   change - confirm?" and wait. *Added after edits landed before the owner saw the wording.*
> - **Don't-Declare-Working-Before-Confirmed** - never write "fixed / works / resolved" for a
>   CI run / build / deploy until it is GREEN in the real target env AND owner-confirmed; a local
>   pass is evidence, not proof. *Added after "CI fixed" was declared on a local-only green while
>   CI stayed red.*
> - **Negative-Testing** - for every path that MUST be rejected, write a test asserting the
>   SPECIFIC failure (HTTP status + machine error code / SQLSTATE), never a weak "an error
>   occurred". A correctly-coded error response IS the passing test.
> - (with `test-program`) **Regression-Coverage** - work touching a closed domain re-runs its
>   suite + back-fills new interaction edges, bidirectionally (close cheap gaps in-run, log the
>   rest). *Fully dormant until the suite + CI exist.*
> - **Per-Stage / Per-Domain Independent-Review** - at every stage/domain boundary, proactively
>   ready the infra + prepare a fresh-chat external-oracle review, and notify the owner it is ready.
>   The review *discipline* is universal (fires on meaningful work from day one); only its empirical
>   teeth (re-running a suite, mutation/coverage checks) wait on `test-program`.

## Generates in target

- A "Standing Gates" section in the root `CLAUDE.md`.

## Files it scaffolds

- None. The gates are behavioral; their durable artifacts (findings, deferrals) live in
  `tracker`, their review briefs in `handoff`, their rationale in `memory`.

## Why

Every gate here is scar tissue. Each was added the day a real failure cost real time: a commit
that silently dropped a sealed decision; an implementation that forgot half its own plan and
invented a column the codebase already had a mechanism for; a "CI fixed" that was red in the real
runner; a test that asserted "something threw" and let a wrong-reason failure pass; a domain
declared "done" on a test count while whole scenario genres were missing. Shipping these on day
one - with the rationale inline so they read as lessons, not bureaucracy - is what stops a new
project from re-learning each one the hard way. The keystone (gate 7) is the external oracle: the
single mechanism that catches the self-referential failure a green check cannot (a check that
mirrors the code instead of the intent). The whole OS hangs from it.
