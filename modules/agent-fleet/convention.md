# agent-fleet - convention

The execution layer of the OS. This is the runtime that turns the external-oracle principle into a
working loop: single-responsibility roles, a router that plans the gates, and the hard rule that
**no single agent both interprets intent and judges its own output.**

> This module REFERENCES, does not redefine, the shared vocabulary. The external oracle and
> "green != correct-to-intent" are owned by the OS thesis; the precedence ladder is owned by
> `source-of-truth`; the standing gates and "Required User Action" are owned by `gates`; the
> anti-scope framing + two completeness gates are owned by `handoff`; `no-finding-dissolves` + the
> TRACKER are owned by `state-docs`. This module owns the **Output Contract** (the universal inter-role
> interface), the **gated workflow** itself, and the **generation pattern**.

---

## 1. The reliability thesis (why a fleet at all)

A single agent that writes code AND decides it is correct is its own oracle - and a self-referential
oracle is exactly the gate-passing failure the OS exists to prevent. So the work is split across roles
that each have ONE job, and the two judgement roles (`spec-guardian`, `code-reviewer`) are structurally
downstream of and distinct from the producing roles. Even in the lite tier - where there is no
orchestrator and no separate implementer - you still split **interpret** from **judge**: the review
pass is run as a deliberate role-switch, not as the author nodding at their own diff.

Two oracle relationships the fleet helps close:
1. **artifact <-> truth-docs** - the `spec-guardian` (intent) + `code-reviewer` (correctness) gates, and
   ultimately the independent fresh-chat review the per-stage gate triggers.
2. **truth-docs <-> owner intent** - only the human closes this; the fleet surfaces it via the
   `implementer`'s **Required User Actions** field and the orchestrator's escalation rule.

---

## 2. The role set

### 2.1 The five generics (shipped as templates; stack-independent; copied as-is)

| Role | One job | Oracle relationship |
|---|---|---|
| `orchestrator` | route + classify + select roles + plan the gates + emit the final approve/return | the hub; owns no judgement of its own |
| `spec-guardian` | judge proposed/implemented work against the specs (the **intent** oracle) | artifact <-> truth-docs |
| `code-reviewer` | judge changed code for correctness/regression/edge/verification (the **correctness** oracle) | artifact <-> truth-docs |
| `current-state-specialist` | record what is now built/pending/blocked after approved work (the reality recorder) | substrate |
| `git-dev-push` | summarize-first terminal commit/push | (its own deferred `git-dev-push` module; the orchestrator hands off to it - a plain commit until it ships) |

The generics never hardcode file paths. They discover the Source Of Truth docs dynamically by reading
the project's `CLAUDE.md` "Source Of Truth" section. That is what lets the same template serve any
project. (Earlier project-specific versions hardcoded paths like `.cursor/current-state.md`, `docs/project-status.md`, etc. -
that is project-payload noise; the generalized templates say "the AI-facing state doc," "the
human-facing status doc," "the master/source doc.")

### 2.2 The three generated role TYPES (full tier; emitted per project by the bootstrap)

| Type | Generated... | Interprets product meaning? | Writes code? | Does manual platform ops? |
|---|---|---|---|---|
| `<layer>-implementer` | one per tech layer found (backend / web / mobile / db / ...) | no - consults specialists | yes | no |
| `<domain>-specialist` | one per significant domain (its own entities + workflows + edges) | yes - this is its whole job | no - advises only | no |
| `<platform>-platform-specialist` | one per managed platform found | no | no | yes - guides the human |

These are NOT shipped pre-built. They are derived from the source-of-truth analysis, using the FORMATs
in `templates/*.format.md`. A project with no managed platform gets no platform specialist; a project
with one domain gets one specialist. (See §7, the generation pattern.)

---

## 3. The universal skill-body shape

Every role - generic or generated - uses the same five-part shape. This uniformity is what lets the
orchestrator route to any role and know what it gets back, and what lets one role's output be the next
role's input without translation.

```
## Use When            - the trigger list (when the orchestrator selects this role)
## Required Sources     - ordered by the precedence ladder (source-of-truth owns the order)
<body>                  - the role's actual work (responsibilities / review focus / procedures)
## Do Not               - lane discipline: >=1 bullet that names what this role must NOT do
## Output Contract      - the structured handoff (see §4)
```

The `## Do Not` block is not decoration - it is the **lane discipline** that keeps roles from
re-converging into "one agent does everything." Every role must have at least one bullet that names the
adjacent role's job as off-limits (implementer: "do not interpret product meaning - consult the
specialist"; specialist: "do not write code"; reviewer: "do not flag things the spec intentionally
simplified"; etc.).

---

## 4. The Output Contract (the universal inter-role interface)

**This module owns this definition.** Every role returns a structured Output Contract; downstream roles
and the human consume it. The fields are a superset; each role fills the ones that apply:

- **Sources used** - which truth-docs (by precedence) this role actually read. Makes the
  "re-derived from the spec, not the summary" claim auditable.
- **Assumptions made** - anything the role decided that was not explicit in the sources.
- **Summary** - what was interpreted / written / found.
- **Cross-domain risks / implications** - the effects on OTHER domains the producing role may not have
  considered. (Specialists name this proactively via the "Surface effects on:" radar; see §6.2.)
- **Required User Actions** - the human-in-the-loop seam: anything not complete by editing code alone
  (deploy, config, secrets, a manual migration, a verification only the human can do). Each one:
  say-so + why + steps + what-to-verify. This is where "GREEN-in-repo != DONE-in-prod" gets flagged.
  (The class itself is owned by `gates` / the "Required User Action" convention.)
- **Escalated ambiguities** - anything that depends on an interpretation the sources do not clearly
  support. ALWAYS present as a field (even if empty) on specialists and guardians, so "none" is an
  explicit assertion, not a silent omission.

The seam contracts (which role's Output Contract is which role's input) are listed in §5.4.

---

## 5. The gated workflow

### 5.1 Tier: lite (the floor)

On meaningful work, run two review gates as a checklist before declaring done:

1. **`spec-guardian`** - spec alignment, cross-doc consistency, invented-logic risk, derived-vs-stored
   truth, unresolved ambiguity. Re-derive expectations from the Source Of Truth, NOT from the summaries.
2. **`code-reviewer`** - correctness, regressions, edge cases, missing/weak verification.

Run them as a deliberate role-switch: the agent that produced the change adopts the reviewer lane and
reads the actual files fresh. That single split - interpret vs judge - is the minimum that keeps
reviewer != author. No orchestrator, no separate implementer, no state role required at this tier.

### 5.2 Tier: full (the gated state-machine)

```
UserTask
  -> ORCHESTRATOR        classify (domains/layers) · select role(s) · source (controlling docs) · gate-plan
  -> domain SPECIALIST   interpret the rules; Output Contract = rules + cross-domain implications + ambiguities
       [its Output Contract IS the implementer's INPUT]
  -> IMPLEMENTER         write code; Output Contract = affected area + assumptions + cross-domain risks
                         + REQUIRED USER ACTIONS
       [its Output Contract IS the reviewer's + human's INPUT]
  -> GATE 1: SPEC-GUARDIAN   intent oracle (contradiction / invented logic / derived-vs-stored /
                             cross-domain / unresolved ambiguity)
  -> GATE 2: CODE-REVIEWER   correctness oracle (only if code changed)
  -> ORCHESTRATOR        decision: approve | return-for-fix -> loop back to specialist/implementer
  -> CURRENT-STATE-SPECIALIST   if state changed materially
```

Do not bypass the gates on meaningful work. The standing gates from the `gates` module wrap this loop
at its boundaries (pre-commit alignment audit, completeness audit, negative-testing, regression sweep,
and - the keystone - the per-stage independent-review handoff gate).

### 5.3 What "meaningful work" means (the gate trigger)

Changes code, schema, API, or domain behavior. Pure formatting, a typo fix in a comment, or reading is
NOT meaningful and does not require the ceremony. The orchestrator (full) or the author (lite) makes
this call up front; when in doubt, treat it as meaningful.

### 5.4 The hand-off seams (each role's output is the named input of the next)

1. **specialist -> implementer** - contract = the specialist's Output Contract (rules + cross-domain
   implications + ambiguities). The implementer consumes it and NEVER re-interprets product meaning.
2. **implementer -> reviewer + human** - contract = the implementer's Output Contract; its **Required
   User Actions** field is the seam where prod-gap flags reach the human.
3. **orchestrator -> gates -> state-docs** - the orchestrator's gate decisions invoke guardian/reviewer;
   post-approval material state change invokes `current-state-specialist`; standing gates fire at
   commit/milestone/stage boundaries and write findings to the TRACKER (`state-docs`).
4. **per-stage gate -> handoff -> fresh review** - at every stage/domain boundary the keystone gate
   (owned by `gates`) produces a `handoff` brief a fresh chat consumes; its findings flow back to the
   TRACKER via `no-finding-dissolves`. This is the build-loop -> review-loop seam.

---

## 6. Role boundaries (the lane discipline, in detail)

The boundaries are the whole point - they are what keep the fleet from collapsing into one do-everything
agent. State them in the AI-architecture doc and enforce them in each role's `## Do Not`.

### 6.1 Implementers
- Write code. Consult specialists for rule interpretation; do NOT decide product meaning.
- Never invent business logic, lifecycle meanings, or field semantics not in the specs (the
  `No Invented Business Logic` rule, owned by `gates`/CLAUDE.md).
- Surface, in the Output Contract, the cross-domain risks and the Required User Actions.
- **Parity gate (paired peer implementers).** When two implementers ship the same surfaces on different
  clients (e.g. web + mobile), each carries a symmetric **parity block**: before shipping a feature on
  one client, confirm the equivalent path exists or is planned on the other; ban "X-only / simplified-Y"
  framing; a true platform limitation must be filed as a tracked entry, not silently diverged. (This is
  the generalized form of a real project's cross-client parity rule - the *pattern* travels; the
  project-specific section numbers and copy bans do not.)

### 6.2 Domain specialists
- Interpret rules from the approved specs. They do NOT write code; they advise.
- Carry the **"Surface effects on:" radar** - a standing list of the adjacent domains this domain
  touches, so cross-domain implications are named proactively in every Output Contract (not discovered
  later by the reviewer).
- Carry an always-present **ambiguities-to-escalate** field - "none" must be an explicit assertion.
- Hold custody of any normative user-facing copy / labels for their domain (so wording does not drift
  per-implementer).
- Do NOT resolve cross-domain ambiguity unilaterally - escalate it.

### 6.3 Platform specialists
- Do NOT invent product logic and do NOT write code - that is the implementer/specialist lane.
- Translate code/spec requirements into safe, verifiable manual platform actions, using the six-field
  **Guidance Format** (§6.5).
- Maintain a `Project Context` stub - the current environment facts (project ref, what is set up, which
  secrets live where). This is the ONE place project-environment specifics legitimately live; keep it
  thin and current.
- Never recommend exposing secrets / service keys to clients; never run destructive ops without explicit
  user confirmation.

### 6.4 Orchestrator
- Routes a task to the right specialist + implementer, plans which gates are required, and emits the
  final approve / return-for-fix framing. It owns NO judgement of the content itself - it manages the
  flow and defers the verdicts to the gates.

### 6.5 The platform-specialist Guidance Format (six fields)
When a human must take a manual platform action, the platform specialist returns:
- **Goal** - what we are achieving.
- **Where** - the full navigation path in the platform console/dashboard.
- **What to click / configure** - precise.
- **What to enter** - exact values where applicable.
- **Verify** - a query or visible state that confirms success.
- **Risks / side effects** - especially anything destructive or hard to reverse.

---

## 7. Ambiguity escalation (define once; every role references this)

If a point is materially unclear and may affect product meaning or multiple domains:
1. **Stop** before implementing.
2. Summarize what IS known from the sources.
3. Explain the implications of each interpretation.
4. Ask the user a focused clarifying question - inline in chat, with full background per option and an
   explicit recommendation (not a terse popup).

This is the same protocol the AI-architecture doc states once and every role's body points back to.
It applies especially to: a setting whose default/boundary is unclear, a cross-domain interaction, and
anything not explicitly in the source-of-truth. Unresolved ambiguity is a HARD STOP - the `spec-guardian`
fails any work that depends on an unsupported interpretation, and it must go back to the user before
approval.

---

## 8. Review-gate rules (when each gate fires)

| Gate | Fires when | Verdict |
|---|---|---|
| `spec-guardian` | spec alignment, cross-doc consistency, invented-logic risk, cross-domain effect, any unresolved ambiguity | `pass` / `fail` + findings (each with a source reference) |
| `code-reviewer` | code / schema / API contract / UI logic changed | `pass` / `fail` + severity-ordered findings + residual risks |
| `current-state-specialist` | state changed materially (feature shipped, blocker resolved, decision sealed) | proposed updates to the state docs + remaining blockers |

The guardian's findings always carry a **source reference** (so "this violates the spec" is auditable,
not an assertion). The reviewer's findings are **severity-ordered** (Critical / High / Medium / Low) and
focus on risk, not style. Neither gate flags something the spec intentionally simplified.

Encode each cross-cutting policy (pagination discipline, parity, security-of-a-field, etc.) as a
**reviewable checklist item with a source reference** inside the relevant gate - so the policy is
mechanically checkable rather than living in someone's memory. (A real prior project did this for pagination and
parity inside `spec-guardian`; the *pattern* - "normative policy -> reviewable gate item with a ref" -
generalizes; the specific §-numbers do not.)

---

## 9. The generation pattern (full tier; how the bootstrap emits per-project roles)

The bootstrap (Phase 3) generates the project-specific fleet from the source analysis. Reference the
FORMATs in `templates/`:

- **3a - the 5 generics**: copy `orchestrator.skill.md`, `spec-guardian.skill.md`,
  `code-reviewer.skill.md`, `current-state-specialist.skill.md` as-is (plus `git-dev-push` from its own
  deferred module - a plain commit until it ships). They are stack-independent and discover the Source
  Of Truth dynamically.
- **3b - implementers** (one per tech layer found): fill `implementer.format.md`. The title names the
  exact stack. Add the **parity block** symmetrically to paired peer implementers.
- **3c - domain specialists** (one per significant domain - its own entities + workflows + edges): fill
  `specialist.format.md`. Include the **"Surface effects on:" radar** + the always-present
  **ambiguities-to-escalate** field + custody of normative copy.
- **3d - platform specialists** (one per managed platform found): fill `platform-specialist.format.md`.
  Include the six-field **Guidance Format** + a stubbed `Project Context`.

A domain earns a specialist only if it has its OWN distinct entities, workflows/rules, AND edge cases -
not just shared ones. A project with no managed platform gets no platform specialist. Generate exactly
what the analysis warrants; do not pre-ship roles a project does not need.

The fleet's binding artifact is the AI-architecture doc (bootstrap Phase 5): it carries the Runtime
Model state-machine, the Role Set (generics + this project's generated roles), the Role-Boundary
invariants, the single ambiguity-escalation protocol, the review-gate matrix, and the project-specific
**Domain -> Specialist routing table**. That doc is what makes the roles a *fleet* and not five isolated
prompts.

---

## 10. The non-negotiable

Whatever the tier, **the reviewer is never the author.** The independent oracle is the point. Everything
else in this module is machinery in service of that one rule.
