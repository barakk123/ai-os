# test-program — convention

The test program exists to close one of the OS's two oracle relationships empirically:
**artifact ↔ truth-docs**. Everything below is in service of that. Where this module says "the oracle,"
it means the source-of-truth precedence ladder (owned by the `source-of-truth` module); where it says
"the external oracle" or "independent review," it means the gate owned by the `gates` module (Gate 7).
This module defines its own terms once — failure-mode-A/-B, the scenario list, the genre taxonomy,
the genre-applicability ledger, mutation score, reference-domain-first, the prod-safety invariant — and
references the rest.

---

## 0. The thesis this whole module is built on

**The source-of-truth doc is the oracle; the code is never the oracle.**
A green check proves "the artifact does what the check asserts," not "the artifact does what intent
requires." There are two failure modes, and conflating them is what makes test programs feel safe while
being unsafe:

- **failure-mode-A — coverage ABSENCE.** No test exists for path X. Easy to detect (coverage tools find
  it). NOT the scary one.
- **failure-mode-B — wrong EXPECTATION (the dangerous, gate-passing one).** A test EXISTS, is green, is
  well-categorized, "covers" the path — but it asserts **what the code currently does**, not **what the
  truth-doc says it should do**. Its expected value was read off the CODE (X), not derived from the SPEC
  (Y), so green only means "code does X" — it says nothing about whether X == Y. The test has become a
  mirror of the implementation. It passes every automated gate.

The decisive framing: a test answers "does the code do X?"; the truth-doc says "the code SHOULD do Y."
A test whose oracle came from the code can never tell you X == Y. Only re-derivation from the spec
(ideally by a reviewer who did NOT read the code) can.

**Why this is not paranoia:** on the proven project, the entire ~1600-test suite had been verified
self-referentially. The first external review found a proper, green, well-categorized test asserting the
EXACT OPPOSITE of a sealed product decision. That is the existence proof that failure-mode-B is real and
gate-invisible. Internalize: **a test is only as good as the independence and correctness of its oracle.**

Two oracles, both must be closed:
1. **tests ↔ truth-docs** — auditable by an independent technical review (writer ≠ reviewer).
2. **truth-docs ↔ owner intent** — only the human owner can close this, via an owner-legible acceptance
   layer (full-tier genre H). A test faithful to a drifted spec inherits the drift.

---

## 1. Tier: lite (cheap; worth it almost everywhere)

The lite floor is five disciplines. Each is cheap, each survives chat-swaps, and together they make a
suite **resistant** to failure-mode-B even before the heavy machinery.

### 1.1 The source-of-truth is the oracle, never the code
Every expected value is hand-derived from the spec FIRST. You may read the code ONLY to (a) set a row's
current status, or (b) flag a divergence — NEVER to derive what the answer should be. The anti-pattern is
the inline snapshot / approval test taken off current output: it silently ratifies the implementation as
the definition of correct. If you must use a golden, **hand-compute** it from the spec and assert explicit
per-field values.

### 1.2 Spec-derived scenario-list-first
Before writing ANY test for a domain, enumerate every rule / edge / limit / toggle / gate / state
transition / error code from the spec into a reviewable checklist (one row each — see
`templates/scenario-list.template.md`). Get the LIST reviewed for breadth first; only then write tests.
This makes intended breadth visible before code exists, and it separates "what should we test" (a spec
question, ownable by the human) from "did we test it" (a mechanical question).

### 1.3 Negative-testing with specific asserts
The negative paths are first-class. Every operation that MUST be rejected gets its own test, and that test
asserts the **specific** failure, not "some error happened":
- e2e: exact HTTP status + the machine error code (`res.json().error.code`).
- service: the typed error `{ statusCode, code }`.
- DB constraint / RLS: the exact SQLSTATE (CHECK / unique / RLS-deny / recursion / …).

`expect(err).not.toBeNull()` passes for ANY error — connectivity, the wrong constraint, the wrong code —
so it can let a wrong-reason failure pass falsely and hide a real bug. A correctly-returned,
correctly-coded error response IS the assertion (the green test). Treat negative-path coverage as EQUAL
to happy-path coverage when deciding whether a domain is "covered". (Asserting specifics is also what
surfaces genuine bugs: on the proven project, tightening "an error occurred" to a specific code uncovered
a stale-report race and an RLS infinite-recursion that broke the entire authenticated read layer.)

### 1.4 The prod-safety invariant (a HARD, TESTED guarantee)
Tests can NEVER touch production. Implement it as **four fail-closed layers**, not a convention:
1. **Isolated test env vars** — the test config points only at a local/test target.
2. **No prod credentials in CI** — the prod secrets are simply not present in the test environment.
3. **A runtime fail-closed guard** — a helper that REFUSES to run (throws) if the target URL is not a
   recognized local/test host. This guard is itself unit-tested.
4. **Destructive ops (DB reset, truncation) run only against the local target** — wired behind the same
   guard.
Default to fail-closed: if the environment is ambiguous, refuse. This is the one place "better safe" beats
"better convenient" unconditionally.

### 1.5 The per-domain run process
Each domain is one run (or a few). The standing sequence:
1. **Read the spec AND the code** for the domain (the spec to derive the oracle; the code to set status /
   spot divergences).
2. **Enumerate the scenario list** (§1.2) → get breadth reviewed.
3. **A mandatory SECOND completeness pass** — re-derive from scratch, deliberately hunting missed
   scenarios / edges / interactions the first pass skipped. Report it as its own step ("second pass added
   rows X, Y, Z"). It routinely finds real gaps.
4. **Write tests**, applying the negative-path rule and the oracle rule.
5. **Close cheap gaps in-run; log the rest** as tracked findings (no-finding-dissolves, §6).
6. **NEVER guess counts** — run the suite for the real number; keep a human-readable per-test index
   regenerated from the runner's `list` output.
7. **The backward retro sweep** (full-tier, §4.3) — when covering a new domain, also re-examine the
   ALREADY-covered domains for interaction edges the new domain created but that were never back-filled.

---

## 2. Tier: full — the genre universe + the master matrix

The full tier exists for owned products, and is non-optional for **AI-authored** codebases (where
inter-agent drift and hallucinated surfaces compound failure-mode-B). It adds breadth (every genre),
depth (a per-(rule × genre) matrix), and an empirical backstop (mutation).

### 2.1 The genre taxonomy
The lite tier covers ~10–19 functional/structural genres. The full tier considers the COMPLETE universe —
~149 deduped genres across categories A–K (functional · data/DB · security-privacy-compliance ·
concurrency/ops · perf · generative/property/formal · meta/measurement · acceptance/human ·
domain-specific · client/frontend · store-release). See `templates/genre-taxonomy.md` for the full list
and what each catches. Categories J (client) and K (store) ship **pre-registered but DORMANT** until a
frontend / mobile / release build exists. You do not run all 149 per domain — you DISPOSITION all of them
(next).

### 2.2 The genre-applicability ledger (per domain — the completeness gate)
For every domain, every genre in the taxonomy gets a written disposition in a ledger table:

| genre | disposition | reason |
|---|---|---|
| 2. Integration (service × real DB) | covered | <where: which test files / rows assert it> |
| 31. Time/timezone storage | partial | <what's covered, what residual is enumerated-but-thin> |
| 25. RLS / row-level authz | scoped-out | SCOPED-OUT: this domain owns no table → no own RLS policy; authz is at the API layer (covered as genre 37). EXPLICIT so table-owning domains do not inherit the silence. |

**Silence is a gap.** A genre is `covered`, `partial` (covered partly + the residual named), or
`scoped-out` WITH A WRITTEN REASON. A genre that is simply absent from the ledger is an un-triaged hole.
The scope-out reason must be domain-specific (e.g. "Type-D endpoint owns no table") and must explicitly
note that domains which DO own the surface must cover it — so a later domain copying this template does
not inherit a scope-out that does not apply to it.

**Dormant-category re-expansion trigger (J client / K store-release).** When a category is scoped out only
because its *surface does not exist yet* (no frontend → Category J; no store/app-release build → Category
K), the ledger row is `scoped-out` but carries an explicit **REACTIVATION TRIGGER**, not a permanent
dismissal — e.g. `scoped-out (DORMANT): no Expo/web build exists; REACTIVATE when a release build lands →
re-expand category K into its constituent genres and disposition each.` This is different from a genuine
per-domain scope-out (a Type-D endpoint owning no table is scoped out *forever* for that domain). The
trigger makes the dormant categories impossible to forget: when the gating build arrives, every dormant-with-
trigger row is re-opened and the category is itemized into its individual genres (store-listing metadata,
build/signing, deep-link, push, store-review-policy, …), each then dispositioned afresh. Log the
reactivation as its own tracked task (`tracker`) so it is scheduled, not merely remembered.

### 2.3 The master matrix (per (rule × genre) → a row)
The matrix is the artifact that makes "complete" measurable. Each row:
- a stable **row id** (e.g. `SE-CORE-026`, never reused, never deleted);
- the **scenario** (the spec rule / edge / transition / code);
- the applicable **genre(s)**;
- the **oracle** = the expected value, with its **spec citation** (the section that mandates it) — derived
  independently of the code;
- a **status**: `exists-correct` / `exists-weak` (asserted but not to bar) / `exists-wrong-expectation`
  (failure-mode-B — a green test asserting the wrong thing) / `absent` / `spec-ambiguity` (needs an owner
  decision — do NOT invent an oracle);
- a **priority** (default every essential-marked-genre row to essential unless a written reason downgrades);
- any **divergence pin** (when the code diverges from the spec, keep ONE row asserting the SPEC value [it
  will fail against current code — that failure IS the surfaced finding] and ONE clearly-labelled
  "per-code / known divergence" pin, never silently mirror the code).

**The status column is computed, not eyeballed.** Determine each row's status by globbing ALL test
locations (co-located unit `src/**/*.test.ts` + a dedicated `tests/**` dir + package `*/src/**/*.test.ts`)
and grepping the actual symbol / field the row asserts — NEVER judge "absent" by directory presence. (On
the proven project a false "~91% absent" headline came from scoping the glob to one directory and missing
a co-located unit suite; the correct multi-path glob flipped 66 rows from absent to weak/correct.) The
headline counts are a **computed roll-up**: a sum of per-group breakdowns, asserting `sum(groups) == total`
so the numbers cannot silently drift.

### 2.4 Mutation testing — the empirical "did we test it" bar
Coverage says a line RAN; mutation says a line is PROTECTED. A mutation tool injects bugs (mutants) into
the code and re-runs the suite; a mutant the suite kills = the tests catch that bug; a **surviving mutant**
= an assertion gap (or a documented equivalent). Mutation score is the single most direct instrument for
the owner's exact fear ("are my tests real?"). The bar: **covered-mutation score ≥ 90% (aspire 95%+) with
ZERO un-triaged survivors** — every survivor is either killed or written down as provably-equivalent.
Mutation, not line/branch coverage, is the real "did we test it" bar; branch ≥ 95% is a necessary-but-not-
sufficient floor.

Wire it as **two scopes that you merge** (see `templates/mutation-tooling.md` for the full how-to):
- a **DB-free** config — mutates the pure logic, runs only the no-DB unit suite (fast, no stack needed);
- a **DB-backed** config — mutates the query-arg / DB-interaction line ranges that only the
  integration/e2e suites reach, run against the live local stack;
- **merge:** a mutant is killed overall if killed by EITHER config. Report the combined covered-mutation
  score. Start report-only (`break = null`), then set `break = 90` per-domain as each domain reaches the
  bar (so a green suite is never broken by a gate it does not yet meet).

**Cheaper, EXECUTABLE backstops against failure-mode-B (partial automation of what Gate 7 does manually).**
Mutation is the strongest mechanical instrument, but it is slow. Two lighter static checks catch a real
slice of failure-mode-B in CI on every push, well before the manual external-oracle review (§4.2):
- **assertion-density / anti-tautology lint** — flag tests that run code but assert nothing observable, or
  whose only assertion is `not.toBeNull` / `toBeTruthy` / a bare `expect(fn).not.toThrow()` (§1.3). A test
  with no specific assertion cannot be asserting the right thing.
- **change-detector / snapshot lint** — flag inline snapshots and approval-goldens captured off current
  output (the §1.1 anti-pattern). A golden whose expected value was never hand-computed from the spec is a
  failure-mode-B candidate by construction; require a spec-citation comment beside each golden or block it.
These do NOT replace the manual Gate 7 (a lint cannot tell whether a SPECIFIC, well-formed assertion encodes
the SPEC value or merely the code's value — that judgment still needs a human re-deriving from the truth-doc).
They cut the volume the manual reviewer must inspect by killing the obviously-tautological tests automatically.
Treat them as a cheap CI pre-filter that feeds, not substitutes for, the external oracle.

### 2.5 Property / metamorphic / differential / invariant (generative genres)
Example-based tests with hand-computed expected values share ONE internal oracle. Generative genres attack
that limit:
- **property** — invariants the output must satisfy over generated inputs (no overlap; fits within range;
  ranking monotonic under a policy).
- **metamorphic** — relations the system must preserve under input transforms (translate the day → all
  slots translate; add an occupied interval → the candidate count cannot rise).
- **differential** — the real implementation vs an INDEPENDENT brute-force reference oracle, on random
  inputs (the cleanest external oracle there is — two independent derivations of the same answer).
- **invariant / data-integrity** — global properties over arbitrary random states ("no two live bookings
  overlap per provider" as a CHECKED property, not one example).

### 2.6 The ratified completeness BARS ("done" is never a count)
A domain is "done" ONLY when ALL hold (this is the definition written into its ledger header):
1. Every `buildable: now` matrix row is built + green (per-row checkbox).
2. Every blocked row is built once unblocked OR carries a live tracker / owner-decision ref — none silently
   dropped.
3. The genre-applicability ledger is fully dispositioned (covered / partial / scoped-out-with-reason).
4. Covered mutation ≥ 90% (ZERO un-triaged survivors) + branch ≥ 95%.
5. Every finding in the register is triaged — zero un-triaged.
6. 100% of error codes have a specific-assert negative test; 100% of state transitions (legal + illegal)
   tested; zero flaky, zero skips, order-independent.
7. Oracle-independence: expected values derived from the spec independently of the code, audited by a
   DIFFERENT agent than the author (writer ≠ reviewer).

**"Done" is NEVER a test count.** "1600 green" / "209 tests" proves nothing about correctness-to-intent.
And **"tracked" is NOT a terminal disposition** for a spec-mandated rule: a bare tracker entry with no
guarding test does not satisfy the gate. A terminal disposition is (a) pinned-by-a-test that presently
exercises the behavior, (b) blocked-on-an-owner-decision, or (c) an explicit owner-approved
deferred-with-reason marker.

### 2.7 T0 calibration — audit the inherited green suite BEFORE you build (measure, don't assume)
When you are layering this program onto an EXISTING green suite (the common case — most codebases already
have tests written self-referentially), do NOT assume the inherited suite is either fine or rotten. **Run a
calibrated sampling audit first** to estimate the failure-mode-B infection rate empirically. It is cheap and
fast, and its number decides the scope of everything after it (a targeted fix vs a full expectation-audit).
The protocol:

1. **Sample N high-stakes domains** (the deep-logic / settings-config / money / security ones — where a
   wrong expectation costs the most), not a random spread. ~4 domains and ~50 rules is enough to calibrate.
2. **Independent-oracle each sampled rule.** Derive the expected behavior from the truth-doc FIRST, BEFORE
   opening the existing test (writer ≠ reader where possible). Only then read the test that "covers" it.
3. **Classify each rule** against your independent oracle: `correct-expectation` · `wrong-expectation`
   (failure-mode-B) · `weak` (asserted, not to bar) · `absent` (failure-mode-A) · `spec-ambiguity`.
   **Adversarially refute** every claimed wrong-expectation from BOTH a spec lens and a test lens before
   recording it (so the audit itself does not manufacture false positives).
4. **Distinguish SILENT failure-mode-B from DISCLOSED.** A test that pins wrong behavior with NO note is the
   dangerous one; one that honestly discloses the divergence in a comment + a tracker id is already half-
   handled. Report the silent rate separately — it is the real exposure.
5. **Produce a per-domain infection estimate** + the concrete suspect rows (each → a `T-XXXX`). Compute the
   denominator honestly: the wrong-expectation/weak rates are fractions of the rules that HAVE a test, not of
   all sampled rules (absence is a separate axis).
6. **Glob ALL test locations before judging `absent`** (§2.3): co-located unit + dedicated `tests/**` +
   package suites, grepping the actual asserted symbol. A false "mostly absent" headline routinely comes from
   scoping the glob too narrowly — verify before you scope the remediation around it.

The output is a decision input, not a verdict: a contained, localized rate → targeted fixes; a systemic rate
→ a full independent expectation-audit + the genre-closure program. Either way you sized the risk by
measuring it instead of guessing. (Example calibration outcome from the proven project: ~12% wrong-
expectation and ~10% weak among tested rules — real and recurring, but not pervasive — with the silent
sub-rate concentrated in the settings/config domain, and the BIGGEST finding being failure-mode-A absence in
the hardest engine. That shaped the whole remediation order.)

---

## 3. The 3-phase delivery model + reference-domain-first

Do not test everything at once, and do not template an unproven approach 15×.

- **Phase 1 — the reference domain.** Pick ONE self-contained, finalized domain that exercises a broad
  slice of genres + a real security surface + a real interaction surface. Drive it to FULL closure (all
  bars), then put it through an INDEPENDENT review (writer ≠ reviewer; Gate 7). The reviewed reference
  domain is the proven TEMPLATE. **For an AI-authored codebase, pick the HARDEST core-logic domain as the
  reference** — coverage tends to be inversely correlated with complexity, so the engine that most needs
  the rigor is the one most likely to be under-tested. Do NOT declare the program ready after the
  reference domain alone.
- **Phase 2 — every existing finalized domain.** First INVENTORY what "all existing code" actually
  comprises (full domains / partials / shared helpers — examined in practice). Then cover EACH to the
  reference template, in a **dependency-aware order** (dependencies before dependents; close the corners
  that cause forced skips first), one (or a few) per run, each shipping its own genre-applicability ledger.
- **Phase 3 — new / future code.** Everything not yet written is tested **test-first / in parallel** with
  its build.

**Gating:** the test stage closes only when Phase 1 + Phase 2 are complete. Phase 3 begins after Phase 2.
(Spec-only work is not gated by test coverage and keeps its normal slot.)

**Reference-domain-first is load-bearing because of templating.** Whatever the reference domain does
(including any flaw) gets copied N×. The independent review of the reference domain is where you catch the
RECURRING PATTERNS before they multiply — e.g. sourcing an oracle from a code comment dressed as a spec
ref; "equivalence-by-domination" mis-classifying a killable mutant as equivalent; over-generalizing an
owner decision; accepting "tracked == tested"; over-crediting a genre as "covered" when it is ~8/51 built.
Fix the patterns in the template, then scale.

---

## 4. The standing gates this module activates

### 4.1 Regression-Coverage gate (dormant until the harness lands)
When new or changed work could affect a domain whose tests are already closed: (a) re-run that domain's
suites; (b) verify its matrix rows still hold and ADD any rows the new work introduces (new interaction
edges); (c) fix breakage before merge. CI enforces the mechanical part (every push re-runs the suites; a
red run blocks merge). This gate is owned by the `gates` module; this module supplies its substance and
marks it **dormant until the test harness exists**.

### 4.2 Per-Stage / Per-Domain Independent-Review gate (the keystone — the external oracle)
At the completion of EVERY domain and EVERY stage, PROACTIVELY (unasked) ready the infrastructure and
prepare an independent-review HANDOFF brief for a FRESH chat that re-derives expected behavior from the
source-of-truth (NOT the summaries, NOT the author's conclusions), re-runs the suite, and surfaces every
miss. This is the gate that closes the artifact↔truth-doc oracle CI cannot. It WORKS: on the proven
project it repeatedly caught blockers the author AND the author's own recheck both missed. (The brief
shape + the two non-negotiables it must embed are owned by the `handoff` module; this module is its
trigger and primary consumer.)

### 4.3 The backward retro sweep (the bidirectional half of the regression gate)
The regression check is **bidirectional**. Forward: a new domain re-runs the domains it touches. Backward:
when covering a new domain, re-examine the ALREADY-covered domains for interaction edges / now-unblockable
cases that LATER-added domains created but were never back-filled. The FIRST backward sweep carries the
whole accumulated backlog (it is the largest — run it as a dedicated pass); afterwards each run only
reconciles the priors against the domains added since, so it stays small. Close cheap gaps in-run; log the
rest. Report the second-completeness-pass and the backward-sweep as SEPARATE visible steps every run.

---

## 5. The interaction matrix (cross-domain runtime edges)

A normative matrix of runtime cross-domain interactions (e.g. cancellation × waitlist × frequency-limit ×
notification), each cell a scenario requiring a test. See `templates/interaction-matrix.template.md`. Every
deferred edge is classified by WHY it waits (the scope taxonomy):
- **(a) partner-built-not-yet-tested** — the partner domain's code exists; the edge rides that domain's run.
- **(b) partner-spec-only** — the partner is not yet implemented; the edge waits for its build.
- **(c) spec-decided-but-uncoded-in-the-covered-code** — the KEY class: a product decision that lives only
  in the spec and is NOT yet implemented. The sweep's primary value is making these VISIBLE (as tracked
  findings), not hiding them under "deferred".

The matrix is a living artifact: the stage closes only when every `now` edge is green or pinned-with-a-tracker.

---

## 6. no-finding-dissolves

Every finding from any review / run / sweep gets a DURABLE tracked record (a `T-XXXX` in the TRACKER +
a row in the findings register) — nothing lives only in chat. Every row needing work gets a per-row
ledger. "Done" = the ledgers dispositioned + mutation ≥ bar + zero un-triaged findings — NEVER a test
count. (The TRACKER, the `T-XXXX` id scheme, and the no-finding-dissolves rule are owned by the `tracker`
module; this module is one of its biggest writers. `state-docs` only holds the matrix / register DOCS as
part of the "where are we now" layer — the registry semantics live in `tracker`.) A finding may be closed
cheaply in-run, or logged with a severity and a tracker id, but it is never silently dropped — and
"tracked with no guarding test" does not count as closed for a spec-mandated rule (§2.6).

---

## 7. Tooling (full)

A test runner + the stack's in-process inject / e2e harness + a property library + a mutation tool
(DB-free + DB-backed). Pin versions (CI flake avoidance). Run the suites as projects so you can run
per-domain / several / all. Keep a local stack (full fidelity: the real DB engine + auth + row-level
security + triggers + stored functions) for the integration/e2e tier, behind the prod-safety guard.
Named vendors on the proven project (illustrative, not mandated): Vitest (runner), `fastify.inject`
(in-process e2e), fast-check (property), Stryker (mutation), a local Supabase stack (integration DB),
GitHub Actions (CI). Substitute the equivalents for your stack.
