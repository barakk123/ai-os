# gates - convention

A gate is a **named, always-on discipline** with five parts. Every gate below is written in this
shape so the agent can actually *perform* it, not just nod at it:

- **Trigger** - the exact boundary where it fires (a commit, a milestone, a verify step, a stage end).
- **Procedure** - the literal, ordered steps to walk. Walk them; do not run them from memory of the gist.
- **Dormancy** - when the gate depends on infra not yet built, it is explicitly asleep until then
  (so an early-stage project isn't blocked by a gate it can't yet satisfy).
- **Rationale** - "established because X cost Y." The stamp is not decoration: a tired agent skips a
  rule with no story and runs a rule that earned its place. Keep it.
- **Reporting** - every gate ends by reporting findings, with an explicit **"nothing outstanding"**
  when clean. Silence is not a pass; an explicit clean signal is.

Two cross-cutting rules bind every gate (defined elsewhere, referenced here, never re-argued):

- **No-finding-dissolves** (owned by `tracker`): anything a gate catches that needs later work
  becomes a durable `T-XXXX` row. Nothing lives only in chat. See "Routing findings" at the end.
- **The-spec-is-the-oracle-never-the-code** (the OS thesis): every step that says "re-derive" or
  "don't trust the summaries" appeals to the `source-of-truth` precedence ladder and to the
  external-oracle principle. A green check proves "the artifact does what the check asserts," not
  "the artifact does what intent requires." The gates exist to close that gap.

---

## Universal gates (every project, day one)

These five need no test harness, no fleet, no managed platform. They are the floor.

---

### Gate 1 - Pre-Commit Alignment Audit

> **Reconciles:** prompts-since-last-commit  ↔  what actually landed in the files.
> **Scope:** the commit delta. (Gate 2 covers whole-item scope - keep them distinct.)

**Trigger.** Immediately before *any* commit or push on meaningful work - every run, sub-task, or item.

**Procedure.**
1. Identify the last commit (`git log -1`). Gather every turn - user *and* assistant - since then.
2. Extract each discrete decision, agreement, deferral, correction, and "we'll also do X" from
   those turns into a short checklist. Include the things that were *changed mid-conversation*
   (an early plan the owner later revised is a classic drop).
3. For each checklist item, **READ the actual file** it should have landed in. Do not trust your
   recollection of what you wrote - the failure mode is precisely that you *think* you wrote it.
   For a large file, read the specific region (offset/limit), not just its head - a head-only read
   does not let you assert the tail is correct.
4. Fix or add every gap. If a fix is *new* design (not just landing an agreed decision), honor
   **Gate 3 (Consult-Before-Edit)** first.
5. Run `git status --short` and confirm **every** file you intended is actually staged. Stage an
   explicit file list - never blind `git add -A` (it sweeps in stray temp files and would stage
   junk). A silently-no-op'd edit becomes a "ghost commit" that ships without the file.
6. Report: the fixes applied, plus an explicit **"nothing else outstanding"** when clean. Then commit.

**Operational hazard (carry this).** Keep file **edits in their own tool call**, separate from
shell verification. If you batch an Edit alongside a fragile command (a count, an arithmetic, a
`cd`) and that command exits non-zero, the harness can cancel the whole batch - the edits silently
never apply while the run continues as if they did. Edit in isolation; verify in a separate step;
re-read the anchor before a status-doc edit (long headers get rewritten every run, so a remembered
`old_string` is usually stale).

**Rationale.** Decisions reached in conversation routinely fail to land in files, or land
inaccurately. This audit caught real gaps the first week it existed: an agreed granularity decision
that was never written, and a doc that claimed "all 10 categories covered" when one was absent. A
disciplined re-read against the actual diff catches drift *before* it ships - which is the only
time it's cheap.

**Worked example.**
Over a session the owner agreed three things: (a) use the existing JSONB feature-flag mechanism for
a new toggle, (b) add an example to the API spec, (c) defer the migration to next run. Before
committing, the audit checklist is `[a][b][c]`. Reading the files: (a) landed; (b) the example was
*discussed* but the spec file still has the old text - **gap**; (c) correctly deferred - but there
is no `tracker` row for it yet - **gap**. Fixes: write the example (consult-before-edit, since it's
new prose), open `T-<n> (deferral)` for the migration. `git status` shows the spec file staged and
no stray temps. Report: "Fixed: API-spec example landed; opened T-<n> for the deferred migration.
Nothing else outstanding." Commit.

---

### Gate 2 - Implementation / Plan-Completeness Audit

> **Reconciles:** the implemented artifacts  ↔  the FULL plan + sealed decisions + existing-codebase
> mechanisms.
> **Scope:** the whole item. (Gate 1 is the commit delta - a diff can't see a cross-cutting omission.)

**Trigger.** At each item / milestone completion - and at major step/run boundaries for large items.

**Procedure.**
1. Reconstruct the *full* plan for the item: every planned element, every sealed decision, every
   conclusion - from the roadmap notes, the `tracker`, the source-of-truth, and the whole
   conversation (not just the last commit).
2. Build a checklist of "what this item was supposed to produce" and cross-check each row against
   the **actual** artifacts (READ them - spec text, derived docs, code, migrations, tests).
3. Three explicit checks per row:
   - **(a) landed** - the planned element is actually present, not just intended.
   - **(b) not weakened** - nothing was quietly omitted, narrowed, or watered down from the sealed
     decision.
   - **(c) consistent** - it reuses the codebase's *established* mechanism rather than inventing a
     divergent one (same schema patterns, same helpers, same naming, same gating, same error-code
     convention). Inventing a parallel pattern is a defect even when it "works."
4. Fix omissions, restore weakened decisions, refactor divergent patterns onto the existing
   mechanism, delete stale "awaiting step N" scaffolding text left behind after sign-off.
5. Report fixes + "nothing else outstanding."

**Rationale.** This is the second standing audit, distinct from Gate 1: pre-commit reconciles
prompts↔files commit-to-commit; this reconciles artifacts↔full-plan at *whole-item* scope and
catches the cross-cutting issues a commit diff structurally cannot see. It earned its place
immediately - it caught a spec that introduced a brand-new boolean column when the codebase already
had a JSONB feature-flag + accessor for exactly that purpose, and a stale "awaiting step 3"
blockquote left dangling after the step had shipped.

**Worked example.**
An item adds a new "auto-optimizer" capability gated behind a plan tier. At completion, the audit
reconstructs the plan: schema column, gating helper, two endpoints, spec section, tracker
follow-ups. Check (a): all four present. Check (b): the spec sealed "owner can disable per-business"
but the implementation only exposes a global toggle - **weakened** → restore the per-business path.
Check (c): the new column is a bespoke `optimizer_enabled boolean`, but the codebase already gates
every tier feature through a `plans.features` JSONB key + a `has_feature()` accessor - **divergent
pattern** → drop the column, add the feature key, route the helper through `has_feature()`. Report
the two fixes; confirm nothing else weakened or divergent.

---

### Gate 3 - Consult-Before-Edit

> Confirm the *exact* change before writing it - even when the direction is already agreed.

**Trigger.** Before any Write/Edit to a spec, design doc, governance file, or any owner-facing
normative text (specs, the source-of-truth doc, CLAUDE.md, status docs).

**Procedure.**
1. Before calling the edit tool, output a concise summary of **exactly** what you're about to add
   or change - the wording, the location, the scope. Quote the new text if it's prose.
2. Wait for an explicit green light.
3. Only then call Write/Edit.

Scope note: this gate is about *normative / design* edits. Routine mechanical edits the owner has
already greenlit (landing an agreed code change, a typo, a formatting reflow) don't need a fresh
confirmation each time - the judgment is "would the owner want to see the wording before it's
written?" For specs and design, the answer is yes.

**Rationale.** The owner felt blindsided when spec files were edited before the planned content was
shown - even though the *direction* had been agreed in conversation, the exact wording and scope
had not. Agreement on direction is not agreement on text.

**Worked example.**
The owner says "tighten the cancellation policy section." Direction agreed - but instead of editing
straight away, output: "Planned change to the cancellation spec, §X: replace 'customers may cancel
freely' with 'customers may cancel up to the configured cutoff (default 24h); inside the cutoff the
slot is offered to the waitlist.' Adds one sentence, no other section touched. Confirm?" On the
green light, edit. (If the owner instead wanted the cutoff configurable per-service, you just saved
a wrong edit.)

---

### Gate 4 - Don't-Declare-Working-Before-Confirmed

> A local green is *evidence*, not *proof*. Proof is green in the real target env, owner-confirmed.

**Trigger.** Any time you're about to characterize a CI run, build, deploy, migration, or any
externally-verified thing as "fixed" / "works" / "resolved."

**Procedure.**
1. Distinguish the two environments explicitly: "passes **locally**" vs "green in **<the real
   target>** (CI / the deployed env / the actual database)."
2. Until the real target is green **and the owner has confirmed it**, report status faithfully:
   *"passes locally; awaiting CI / your confirmation"* - never "fixed" / "works."
3. Do **not** flip a `tracker` / status entry to `resolved` on a local-only signal. Keep it
   `in-progress` until the real env is green and confirmed.
4. Be *more* skeptical, not less, about anything with a history of never having worked - that's
   exactly where each iteration fixes one small thing and prematurely declares victory.

**Rationale.** A dependency was added to fix a CI typecheck error; `tsc` passed locally, so "CI
typecheck fixed" was written and the tracker row flipped to resolved and committed. CI was still
red - the local fix cleared error #1 and revealed error #2, which passed locally but failed in CI
(version / fresh-install / env differences). The CI had in fact *never* gone green, yet each pass
declared the latest small fix "working." Local-green ≠ target-green is a recurring, expensive split.

**Worked example.**
A flaky test is patched; `vitest` is green locally. Wrong report: "Fixed the flaky test, CI is
good now." Right report: "The test passes locally on 50 runs. I've pushed; **awaiting the CI run** -
I'll confirm once it's green in Actions. T-<n> stays `in-progress` until then." When Actions goes
green and the owner says "yep, green," only then: resolve T-<n>, and "confirmed green in CI."

---

### Gate 5 - Negative-Testing (assert the specific failure)

> Test the things that SHOULD fail. A correctly-coded error response IS the passing test.

**Trigger.** Whenever you author or review tests - and conceptually whenever you reason about
"does this reject what it must reject."

**Procedure.**
1. For **every** path that MUST be rejected, write a dedicated test. The standard rejection set:
   bad / missing / oversized / wrong-type input; missing / expired / forged auth; wrong role;
   cross-tenant access; an illegal state-machine transition; a DB-constraint violation; a lost race.
2. Assert the **specific** failure - never a weak "an error occurred":
   - API / e2e: the exact HTTP status **and** the machine error `code` (e.g. `res.json().error.code`).
   - service layer: the typed error's `{ statusCode, code }` (e.g. `rejects.toMatchObject(...)`).
   - DB constraint / row-security: the exact SQLSTATE (`23514` CHECK, `23505` unique, `42501`
     security-deny, `42P17` recursion, …).
3. Treat negative-path coverage as **equal** to happy-path coverage when judging whether a domain
   is "covered." Half a domain's behavior is its refusals. This is not a per-PR checklist item you
   tick once - it is enforced structurally by `test-program`'s **genre-applicability ledger**, where
   the rejection genres (bad-input, authz-deny, illegal-transition, constraint-violation, lost-race)
   are first-class rows that must each be dispositioned `covered` / `partial` / `scoped-out-with-reason`.
   A negative genre simply *absent* from a domain's ledger is an un-triaged hole, not an implicit
   pass. (See the `test-program` module §2.2.)

**Rationale.** "Returning the expected failure IS the test passing." A weak assertion like
`expect(error).not.toBeNull()` passes for *any* error - a connectivity blip, the wrong constraint,
the wrong code - so it lets a wrong-*reason* failure pass falsely and hides real bugs. Asserting
specifics is also what *surfaces* genuine bugs: pinning the exact code/SQLSTATE is what exposed a
stale-report-after-revoke defect and a security-policy recursion defect that a vague assertion
would have green-washed.

**Worked example.**
Endpoint: "only an owner may delete a service." Weak (forbidden):
`expect(res.statusCode).not.toBe(200)`. Strong (required): a test where a non-owner calls delete and
asserts `res.statusCode === 403` **and** `res.json().error.code === 'forbidden_role'`; plus a
cross-tenant test asserting `404` (not `403`, to avoid leaking existence); plus a DB-level test that
a direct delete under the wrong tenant raises SQLSTATE `42501`. Three refusals, three specific
assertions - each one is a green test that would have caught a real regression.

---

## Conditional gates (active once `test-program`'s harness lands)

These two depend on a test program existing. **Dormancy clause:** until the harness is installed,
they are explicitly asleep - documented, agreed, not yet enforced. Wake them the moment the suite
+ CI exist. (See the `test-program` module.)

---

### Gate 6 - Regression-Coverage Gate (bidirectional)

> New work that could touch a *closed* domain re-runs its suite and back-fills new interaction
> edges - in **both** directions.

**Trigger.** When new or changed work could affect a domain whose tests are already "closed"
(covered to its scenario list / catalog rows).

**Procedure.**
1. **Forward:** re-run the affected domain's relevant suites (per-section, or all if the change is
   broad). A red run blocks merge.
2. Verify the domain's existing scenario rows still hold, and **add** any rows the new work
   introduces - the new interaction edges between the changed area and this domain.
3. Fix breakage before merge.
4. **Backward retro sweep** (the bidirectional half - the part that's easy to forget): re-examine
   the *already-closed* domains for interaction edges / scenarios / now-unblockable cases that
   *later-added* domains created but were never back-filled (because the retro check wasn't run
   after each of them). The **first** such sweep carries the whole accumulated backlog and is the
   largest; afterward each run only reconciles the priors against the domains added since, so it
   stays small.
5. Close cheap gaps in-run; **log the rest** as `tracker` rows + fold them into the interaction-matrix
   step. The mechanical half (re-running suites on every push) is enforced by CI; this gate is the
   judgment half (which rows to *add*).

**Rationale.** Coverage rots silently. When domain B is added, it creates new interaction edges with
already-closed domain A - but if nobody re-examines A, those edges go untested forever and the "A is
done" claim quietly becomes false. The sweep is bidirectional precisely because the forward-only
check misses every edge a *future* domain introduces into a *past* one.

**Worked example.**
A new "waitlist" domain ships. Forward: its suite is green, and it adds rows for "a freed slot
offers to the waitlist." Backward sweep over the already-closed "cancellation" domain: cancellation
was closed *before* waitlist existed, so no test ever checked "cancelling inside the cutoff triggers
a waitlist offer." That edge was never back-filled - add the row, write the test. Two other priors
(reschedule, swap) are checked: reschedule has an edge (add it, cheap, do it now); swap's edge is
larger → log `T-<n> (finding)` and fold into the interaction matrix. Report: forward green, one
backward row closed, one logged.

---

### Gate 7 - Per-Stage / Per-Domain Independent-Review Handoff Gate (the keystone)

> At every stage/domain boundary, **proactively and unasked**, ready the infra and prepare a
> fresh-chat external-oracle review - then notify the owner it's ready.

This is the gate the whole OS hangs from. The external oracle - **reviewer ≠ author**, re-deriving
expected behavior from the source-of-truth **without reading the implementation** - is the only
mechanism that catches the self-referential failure a green check cannot: a check that mirrors the
code instead of the intent (and therefore passes every automated gate). Build it into the cadence;
never leave it to be remembered ad-hoc.

**Trigger.** The completion of EVERY stage AND EVERY domain of meaningful work. The assistant raises
it *on its own* - the owner must never have to ask.

**Procedure.**
1. **Ready the infrastructure for an independent reviewer.** Clean/known repo + CI state; suite +
   typecheck green (or every delta explained); all artifacts committed/saved. A reviewer should be
   able to start from a clean checkout with nothing dangling.
2. **Write the review brief** (a `handoff`-style document) for a FRESH CHAT - an honest,
   adversarial, verify-don't-trust review that:
   - re-derives behavior from the **source-of-truth, NOT the summaries** (the review variant must
     *not* hand the author's own conclusions over as truth);
   - re-runs the suite / CI / typecheck;
   - surfaces EVERY miss / weakness / wrong-expectation / coverage gap, categorized, with severity
     and an explicit verdict.
   The brief MUST embed (referenced from `handoff`, not re-defined here):
   - the **anti-scope framing** - the seeds / file-maps / prior work are a PARTIAL warm-start, NOT
     the scope; derive the full scenario list from scratch as if the seeds don't exist, then
     cross-check the two derivations; detail ≠ completeness (tripwire: if the final list looks like
     a copy of the seeds, you under-investigated);
   - the **two completeness gates** as separate visible reported steps - the 2nd completeness pass
     (before writing) and the backward retro sweep (after the forward check).
3. **Notify the owner explicitly:** "the infrastructure is ready for a new-chat full honest review
   of \<stage/domain\>." This notification is the assistant's standing responsibility to remember.
4. The fresh review's findings flow back through **no-finding-dissolves**: every miss becomes a
   `T-XXXX`, gets dispositioned (fix now / defer), and the state docs are refreshed.

**Dormancy.** The *review* discipline applies as soon as there is meaningful work to review.
Its empirical teeth (re-running a suite, mutation/coverage checks) sharpen once `test-program` lands;
before then the review still re-derives from the spec and re-reads the artifacts.

**Rationale.** The build loop verifies itself self-referentially: a suite written by the author can
pass while testing the wrong thing (failure-mode-B - the check mirrors the code, not the truth-doc).
Green ≠ correct-to-intent. The only reliable catch is an outside oracle that re-derives intent
independently. The owner ruled this a standing, continuous gate so the review is never something the
owner has to remember to request - it's integral to closing every stage.

**Worked example.**
A domain's test program reaches its internal bars (suite green, mutation over the bar, coverage over
the bar). Rather than declaring it done, the gate fires: confirm the repo is clean and CI green;
write the review brief seeded with the spec sections, the scenario list, and the file map - **but
framed as "this is a partial warm-start, re-derive the full list yourself; here is what to verify,
not what to conclude."** Notify the owner: "infra ready for a fresh-chat honest review of the
\<domain\> test program." The fresh chat re-derives, finds three scenarios the author never wrote
and one assertion that mirrored a bug instead of the spec → four findings → four `T-XXXX` rows →
fixed → state docs refreshed. *That* is "done" - not the green suite that preceded the review.

---

## Retiring or superseding a gate (keep the rationale honest)

A gate is scar tissue, so it must not be *silently* deleted the day it feels inconvenient - a future
session that hits the same failure would have no record of why the gate existed and would re-learn it
the hard way. When a gate is genuinely outgrown (its triggering failure mode is now structurally
impossible, or a newer gate subsumes it), retire it the same way `memory` retires a rule:

1. **Never a silent overwrite.** Append a **dated** note - "superseded by \<gate\> on \<date\>" or
   "relaxed on \<date\> because \<the failure mode no longer applies\>" - to the gate's `memory`
   entry and to its line here. The history of *why it kept mattering* survives the change.
2. **Lapse vs reinforcement, both dated.** If a retired gate's failure recurs, the gate is
   *reinforced*, not re-invented: append a fresh dated note rather than starting over, so the record
   shows the rule lapsed and then earned its place again.
3. **Supersession is a pointer, not a delete.** A gate folded into a stronger one keeps a one-line
   stub pointing at its successor; a reader following an old reference still lands somewhere true.

This discipline is owned by `memory` (the dated-lapse / never-silent-overwrite rule for any durable
rule); the gates merely inherit it. The provenance stamp ("established because X cost Y") and its
retirement note are two halves of the same honesty: a gate must be able to say *why it is still here*
or *why it stopped being needed* - never just vanish.

---

## Routing findings to the tracker (no-finding-dissolves)

A gate that catches something **must not let it dissolve in chat**. The routing is uniform across
all seven gates (the mechanism is owned by `tracker`; this is how the gates feed it):

1. **Every** divergence / gap / weakness / ambiguity a gate surfaces → a `tracker` row
   (`T-XXXX`) or folded into an existing one. The row carries category
   (`deferral` · `finding` · `user-action`), severity, source (which gate / review found it), and
   status.
2. If it's cheap and in-scope, **fix it in-run** and resolve the row in the same pass. If not,
   leave it `open` / `in-progress` - it *will* be fixed eventually, but it is now durable.
3. **"Done" is never a count.** A domain/item is done when its rows are dispositioned and its gates
   are met - the genre/work ledger built, zero un-triaged findings, the bars met - **not** when a
   number of tests is hit. A finding mentioned-and-forgotten becomes a permanent bug; the row is
   what prevents that.
4. Surfaced `user-action` items (a deploy, a secret, a platform setup, a verification the owner must
   do) route to the same ledger as `user-action` rows, so the human handoff is tracked too -
   the seam where Gate 4's "GREEN-in-repo ≠ DONE-in-prod" gets a durable record.

---

## How the gates compose (where each fires in the loop)

The gates are not independent checklists - they fire at distinct boundaries of the one work loop and
hand off to each other:

- **Mid-task:** Gate 3 (consult) before each design edit; Gate 5 (negative-testing) and Gate 4
  (don't-declare) on every verify step.
- **At item/milestone completion:** Gate 2 (whole-item completeness) + Gate 6 (regression + backward
  sweep, when active).
- **At commit:** Gate 1 (commit-delta alignment) - then commit, then `git-dev-push`.
- **At every stage/domain boundary:** Gate 7 (the external-oracle handoff) - the seam between the
  build loop and the review loop.

Gates 1 and 2 are deliberately **paired and distinct** (commit-delta vs whole-item). Gates 6 and 7
both reuse the `handoff` module's **two completeness gates** (2nd pass + backward sweep) - the same
two, referenced from one definition, never forked. And every gate's "re-derive, don't trust the
summaries" clause points at the same root: the spec is the oracle, the code never is.
