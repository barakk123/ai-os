# source-of-truth - convention

This module owns four things: the **precedence ladder** (with stop-on-conflict), the **three source
modes**, the **source-agnostic Phase-0 evaluation** (the 8 dimensions + question bank + decision table),
and the **extraction rules** (sub-specs + api-spec / db-schema / ux-flows). The multi-session interview
that matures a rough spec to derivable lives in the companion `spec-developer` skill; this file defines
the rubric that skill scores against, so the two share ONE scorer.

---

## 1. The precedence ladder (the spine)

List the truth sources highest-first. This is the single ordering the whole OS reads top-down; every
role's `Required Sources` is a slice of it, and every gate that re-derives "from source-of-truth, not
summaries" appeals to it. The canonical ladder (authored-spec mode):

1. **Master spec** - the single self-contained product truth.
2. **Derived per-domain docs** - one per domain; add technical precision but NEVER new business truth.
3. **Architecture / AI-architecture doc** - repo shape, role boundaries, routing, runtime model.
4. **Skills + CLAUDE.md files** - how the AI layer operates.
5. **TRACKER** - the deferrals + findings registry.
6. **State files** - owner-facing status + AI-facing current-state; describe reality, never override truth.

**The two rules that make the ladder load-bearing:**

- **Stop-on-conflict.** If two sources disagree, do NOT guess and do NOT pick the more convenient one.
  STOP, surface the contradiction explicitly, and let the **highest-precedence source win**. A
  lower-rung doc that contradicts a higher rung is a bug in the lower doc (or a real spec gap) - either
  way it is surfaced, never silently reconciled.
- **Lower layers explain HOW, not WHAT.** A skill or a state file can tell you how to do the work or
  what is currently built; it can NEVER establish a product rule. Never treat a summary or a state file
  as stronger truth than the source it derives from. This is the rule that prevents "the status doc says
  X so X must be true" drift.

> **Why it matters (proven):** on a large project this precedence rule is what prevented doc drift -
> when a derived doc and the master spec diverged, the master won and the derived doc was corrected,
> rather than two docs quietly contradicting each other for months.

---

## 2. The three source modes

The mode determines what sits at the top of the ladder and which Phase-0 you run.

### 2.1 authored-spec
The project authors a master spec as the top rung. This is the default for an owned product.
- `spec-developer` matures a rough spec to **derivable**; THEN the derived per-domain docs are extracted.
- Precedence = master spec > derived domain docs > architecture doc > skills + CLAUDE.md > TRACKER > state.
- Phase-0 = score the spec on the 8 dimensions (§4) and develop the gaps before building.

### 2.2 external-monorepo
The truth is an upstream framework / monorepo (e.g. a client codebase you are embedded in). Do NOT
author a parallel spec - it would immediately drift from the upstream and create two contradicting
truths. See §6 for the full Phase-0 variant.
- Distill a THIN `docs/house-rules.md` (light by default; deepen only on request) that captures the few
  conventions that matter locally, and POINT at the upstream as the real truth.
- Precedence = upstream source + house-rules > local conventions > state.
- Phase-0 = score **codebase legibility** instead of a spec, using the legibility rubric in §4.9 (the same
  1-5 scale, per-dimension floor >= 4): conventions-doc presence, pattern discoverability, upstream
  documentation, example+test coverage, naming consistency. Fill the sub-4 dimensions (by writing the
  house-rules doc) before building.

### 2.3 hybrid / greenfield
No spec yet, or a spec that will grow with the project. See §8 for the full Phase-0 variant + the
promote-before-build mechanism.
- Start lean (a one-page vision + entity sketch is enough to begin) and grow the spec as the project
  does. Each meaningful feature promotes its rules INTO the spec before it is built (§8.2).
- Precedence = whatever spec exists > conventions > state; the ladder fills in as the spec matures.
- Phase-0 = a fast, SCOPED 8-dimension score (§8.1) over only the dimensions the first feature touches -
  to decide whether to begin or to run a short spec-developer round on the one or two dimensions that
  block that feature.

---

## 3. The derivable bar (the unit of "done" for a spec)

A spec section is **derivable** when a role can do its WHOLE job from the spec alone - no asking, no
inventing. Concretely:

- A **DB implementer** can design the schema from the Entities section alone.
- A **frontend implementer** can build the screens and a **backend implementer** the handlers from the
  Workflows section.
- A **spec-guardian** can catch invented logic because the approved Business Rules are explicit enough to
  compare against.
- An **API implementer** can write the auth guards from the Roles & Permissions section without asking.
- A **code reviewer** can check that edge cases are handled without inventing what they should be.

"Derivable" is the single acceptance criterion the 8-dimension rubric operationalizes. The bar for a
whole spec being ready is **every dimension >= 4** (not the total - a single 2 blocks even if the total
looks fine, because that one weak dimension is where invention will happen).

---

## 4. Phase-0: source-agnostic evaluation

Read the entire source first. Then score, decide, and (if needed) develop. This is the SAME rubric the
`spec-developer` skill scores against - one scorer, imported by both. Use a **1-5 scale per dimension**;
the bar is **every dimension >= 4** (max 40). Do NOT use a 1-3 scale or a total-only threshold - they hide
the one weak dimension that causes invention.

### 4.1 The 8 dimensions

For each dimension, score 1-5 by the rubric below, then draw from its question bank to develop the gaps.
Each dimension states its **derivable test** - the concrete "a role can do X from this alone" bar that
score 4 must satisfy.

---

#### Dimension 1 - Product Vision

| Score | What it means |
|-------|---------------|
| 1 | Not described at all |
| 2 | One vague sentence ("a system to manage X") |
| 3 | Problem and users described, but no success criteria |
| 4 | Problem + users + success criteria + what must NEVER go wrong |
| 5 | Also: explicit scope boundaries (what is OUT), key constraints |

**Derivable test:** an implementer can answer "why does this feature exist and what makes it successful?"

**Question bank:**
- What specific problem does this solve, and for whom exactly?
- What is the single most important thing the system must do correctly (the NEVER-break invariant)?
- What does success look like 6 months after launch (a measurable criterion, not a feeling)?
- What is explicitly OUT of scope - what will this system NOT do, and what external tool covers it?

---

#### Dimension 2 - Entities

| Score | What it means |
|-------|---------------|
| 1 | Not listed |
| 2 | Named but no attributes |
| 3 | Main attributes listed, no states or relationships |
| 4 | Each entity: name, key attributes + types, possible states, relationships, who creates/owns/modifies it |
| 5 | Also: lifecycle transitions, derived-vs-stored fields, deletion/archival behavior |

**Derivable test:** a DB implementer can design the schema from this section alone.

**Question bank:**
- List the main "things" the system manages (e.g. clients, orders, documents, bookings).
- For each: key attributes + their types? possible states? who creates / owns / modifies it?
- How do they relate (one-to-many? ownership? cardinality)?
- Which fields are STORED vs always DERIVED from other fields? (this prevents the derived-vs-stored bug.)
- What happens on deletion - hard delete, soft delete, archive, or anonymize?

---

#### Dimension 3 - Workflows

| Score | What it means |
|-------|---------------|
| 1 | Not described |
| 2 | Actions mentioned but not stepped out |
| 3 | Steps described but no decision points or error states |
| 4 | Each major workflow: trigger -> steps -> decision points -> outcomes -> what the user sees at each step |
| 5 | Also: concurrent-access scenarios, interruption/timeout behavior |

**Derivable test:** a frontend implementer can build the screens and a backend implementer the handlers.

**Question bank:**
- Walk me through the 3 most important user actions step by step.
- What triggers each workflow? What happens at each step? What ends it?
- Where are the decision points / approval / confirmation steps?
- What does the user SEE at each step (so the screens can be built)?
- What happens if two users act on the same thing at once (concurrent access)?

---

#### Dimension 4 - Business Rules

| Score | What it means |
|-------|---------------|
| 1 | Not stated |
| 2 | Mentioned loosely ("documents have deadlines") |
| 3 | Some rules explicit, some still implied |
| 4 | Every constraint stated precisely: "X cannot happen if Y", "Z = A + B", "when P changes, Q updates" |
| 5 | Also: priority rules when constraints conflict, audit/history requirements |

**Derivable test:** a spec-guardian can catch invented logic because the approved rules are explicit
enough to compare against. **This is the dimension that most directly enables the external oracle** - a
vague rule is a rule the guardian cannot enforce.

Capture rules in the **four-bucket taxonomy** (every business rule is one of these):
- **Constraint** (things that must NEVER happen): "A `<entity>` cannot `<action>` if `<condition>`."
- **Calculation** (how values are derived): "`<field>` = `<formula>`; calculated from `<sources>`,
  never stored independently."
- **Trigger** (what change causes what): "When `<event>` -> `<consequence 1>`, `<consequence 2>`."
- **Lifecycle** (state transitions): "`<entity>` moves from `<A>` to `<B>` when `<condition>`; cannot be
  deleted if `<condition>` - use `<archive/close>` instead."

**Question bank:**
- What are the hard constraints - the things that must NEVER happen, however the user tries?
- How are key values calculated (formulas, aggregations)? Which are stored vs always recomputed?
- What changes AUTOMATICALLY when something else changes (the trigger cascade)?
- What are the legal state transitions, and which are forbidden (must go through an intermediate state)?
- When two rules conflict, which wins? Is there an audit/history requirement?

---

#### Dimension 5 - Tech Stack

| Score | What it means |
|-------|---------------|
| 1 | Not mentioned |
| 2 | One technology mentioned ("we'll use React") |
| 3 | Frontend + backend named, database vague |
| 4 | Frontend framework, backend language/framework, database, hosting, auth approach, each external service named |
| 5 | Also: version constraints, package manager, monorepo-vs-separate-repos, CI/CD approach |

**Derivable test:** an implementer knows exactly which language, framework, and platform to use without
guessing.

**Question bank:**
- Frontend: web / mobile / both? Which framework(s)?
- Backend: which language + framework? Which database?
- Hosting / managed platform? Auth approach? Each external service it depends on?
- Package manager, monorepo vs separate repos, CI/CD - any constraints decided already?

---

#### Dimension 6 - Roles & Permissions

| Score | What it means |
|-------|---------------|
| 1 | Not defined |
| 2 | Roles named but no permissions |
| 3 | What each role can do described, but gaps remain |
| 4 | Each role: what they see, what they can do, what they cannot do, whether they reach other users' data |
| 5 | Also: permission inheritance, role-assignment rules, what happens when a role changes |

**Derivable test:** an API implementer can write auth guards from this section without asking.

**Question bank:**
- Who are the user types?
- What can each role SEE and DO that others cannot? What is each role explicitly forbidden?
- Can a role reach another user's / another tenant's data? (the multi-tenant isolation invariant.)
- How is a role assigned, and what happens when a user's role changes mid-flight?

---

#### Dimension 7 - Edge Cases & Errors

| Score | What it means |
|-------|---------------|
| 1 | Not addressed |
| 2 | "Handle errors properly" level |
| 3 | Some cases mentioned, most implied |
| 4 | For each major workflow: invalid input, unauthorized action, missing data, concurrent access, external-service failure |
| 5 | Also: data-migration edge cases, what legacy/old data looks like, recovery procedures |

**Derivable test:** a code reviewer can check that edge cases are handled without inventing what they
should be. **Couple this with the negative-testing gate** - every must-reject case should name its
specific failure (status + machine code), not "an error occurs."

**Question bank:**
- For each major workflow: what happens on invalid input? unauthorized action? missing data?
- What is the most dangerous thing that could go wrong, and what must the system protect against?
- What does old / legacy / migrated data look like, and how is it handled?
- When a must-reject case is rejected, WHAT is the specific error (status + machine code)?

---

#### Dimension 8 - Integration Points

| Score | What it means |
|-------|---------------|
| 1 | Not mentioned |
| 2 | "We might integrate with email" level |
| 3 | Services named but not scoped |
| 4 | Each integration: purpose, when called, what data flows in/out, what happens if it fails/unavailable |
| 5 | Also: rate limits, retry behavior, fallback behavior |

**Derivable test:** an implementer knows which integrations to build and what to do when they fail.

**Question bank:**
- Does the system connect to external services (payments, email/SMS, auth providers, calendars)?
- For each: purpose, when it is called, what data flows in/out?
- What happens when it fails or is unavailable - retry, fall back, queue, surface to the user?
- Are there rate limits or quotas to design around?

### 4.2 The score-decision table

Decide on the **per-dimension floor**, not just the total. The three rows are keyed on the floor (the
lowest-scoring dimension, an integer on the 1-5 scale), so they are mutually exclusive and exhaustive -
every spec lands in exactly one:

| Condition (keyed on the floor) | Action |
|--------------------------------|--------|
| **All dimensions >= 4** | Proceed - the spec is derivable. (bootstrap goes to extraction.) |
| **One or more dimensions == 3, and none <= 2** | Light development: targeted questions for the sub-4 dimensions (use the question bank), patch the spec, re-score those dimensions. |
| **Any dimension <= 2** | Full development: route to `spec-developer` for multi-session convergence, THEN return. |

A single dimension at 2 blocks even when the total looks fine: that one weak dimension is exactly where a
downstream role will be forced to invent. The floor, not the sum, is the gate.

### 4.3 The optional adversarial audit round (high-stakes / full profile)

After the spec reads as "ready" (all dims >= 4), run ONE deliberate adversarial pass: re-read the whole
spec as a hostile reviewer trying to break it, and produce a **numbered gap register** (every missing
edge, unstated rule, ambiguous transition). This routinely surfaces whole missing features that the
forward pass accepted because each section looked locally complete. Disposition each gap (fix now / defer
to `T-XXXX` / accept-with-reason). This is the spec-side analogue of the test-program's backward retro
sweep: forward completeness is necessary but not sufficient; an adversarial second look is what catches
the systemic miss.

### 4.4 Asking in the project's language

All user-facing prompts (the question bank, the score table, the completion declaration) are phrased in
the **project's chosen language, parameterized** - never hardcoded to one human language. Keep an English
template internally; render to the owner's language at ask time.

### 4.9 The codebase-legibility rubric (external-monorepo mode)

When the truth is upstream (`§2.2`), there is no authored spec to score - so Phase-0 scores **how legible
the existing codebase is** instead. The shape is identical to §4: a small set of dimensions, the SAME
1-5 scale, the SAME per-dimension floor (>= 4) rather than a total. A low-scoring dimension is exactly
where future work will be forced to guess at a convention - so it is closed (by filling the
`house-rules.md`) before building, just as a sub-4 spec dimension is developed before building.

| Dimension | What it measures | Score 4 (the bar) |
|-----------|------------------|-------------------|
| **Conventions-doc presence** | Is there a written conventions / architecture / contributing doc? | A discoverable doc states structure, where-things-go, and the local rules. |
| **Pattern discoverability** | Can you find "how we do X here" by reading the code? | The dominant patterns (endpoint shape, data flow, error handling) are consistent enough to read off existing code without asking. |
| **Upstream documentation** | Is the upstream source-of-truth itself documented? | The upstream's own conventions/architecture docs exist and are linkable, so work re-derives from THEM. |
| **Example + test coverage** | Are there worked examples / tests to copy a new feature from? | At least one end-to-end example (and its test) exists for each kind of change you will make. |
| **Naming consistency** | Do names follow a predictable, inferable scheme? | Naming is consistent enough that a new name's correct form is inferable, not a coin-flip. |

**Scoring + decision** mirror §4.2, keyed on the floor: all dims >= 4 -> the codebase is legible, proceed;
one or more == 3 and none <= 2 -> fill the thin spots in `house-rules.md` (point at upstream docs, capture
the discoverable patterns) then re-score; any dim <= 2 -> the local layer is too thin to build safely -
distill the `house-rules.md` (and, where the gap is upstream, raise it upstream) before proceeding. Each
open gap becomes a `T-XXXX` row and is tracked in the house-rules "Legibility gaps" section until closed.

This rubric is what `house-rules.template.md` references when it asks for a legibility score, and what
§2.2's "score codebase legibility instead of a spec" points at.

---

## 5. Sub-spec extraction (authored-spec mode)

Once the spec is derivable, split it into derived per-domain docs. **A domain earns its own sub-spec only
if it has all three:** its own distinct entities (not just shared ones), its own workflows and business
rules, AND its own edge cases and constraints. A domain that is just a thin view over shared entities does
NOT get a sub-spec - it stays a section of the master.

For each qualifying domain, write `docs/<domain>-spec.md` from `templates/domain-spec.template.md`, with
the **load-bearing precedence header** at the very top:

> Derived from `<master-spec>` § N. **Master spec takes precedence on any conflict.**

The fixed skeleton: Overview / Entities / Workflows / Business Rules / Edge Cases / Cross-Domain
Dependencies. The derived doc may add technical precision (exact field types, index notes, status-machine
tables) but NEVER new business truth - if it needs a rule the master does not contain, that is a spec gap
to fix in the master, not to invent in the derived doc.

### 5.1 The three cross-cutting extractions

Beyond per-domain docs, extract these when the material exists in the master:

- **`docs/db-schema.md`** - when the spec contains schema / data-modeling detail. Pulls the Entities +
  the Data-Model-Overview content into one place: tables, columns + types, relationships + cardinality,
  source-of-truth boundaries (which entity owns which data), stored-vs-calculated fields, deletion
  behavior. The DB implementer's primary doc.
- **`docs/api-spec.md`** - when the spec defines API contracts. Pulls endpoint definitions, request/
  response shapes, the error envelope + machine error-code catalog, pagination/limits policy, and the
  auth model. The API implementer's primary doc.
- **`docs/ux-flows.md`** - when the spec contains significant screen / UX flows. Pulls the Workflows
  content into screen-by-screen flows: what the user sees at each step, the wireframe-level layout, the
  state-to-UI mapping. The frontend implementer's primary doc.

Each of these is a derived doc and carries the same precedence header. They are not new truth - they are
the master's truth re-organized for one consumer.

---

## 6. external-monorepo mode: the house-rules layer

When the truth is upstream, do NOT extract sub-specs - distill `docs/house-rules.md` from
`templates/house-rules.template.md`. Keep it THIN: a pointer to the upstream source-of-truth, the few
distilled local patterns that matter (structure, data flow, where to add things), and any local
deviations from upstream defaults. Deepen only on explicit request. The whole point of this mode is to
avoid a parallel spec that drifts from the real truth.

### 6.1 The Phase-0 variant for this mode

There is no spec to score here, so Phase-0 runs the **codebase-legibility rubric (§4.9)** instead of the
8-dimension spec rubric. The variant is three steps:

1. **Run the legibility rubric.** Score the five dimensions (conventions-doc presence, pattern
   discoverability, upstream documentation, example+test coverage, naming consistency) on the 1-5 scale,
   floor >= 4 (§4.9). The decision is keyed on the floor exactly like §4.2: all >= 4 -> the codebase is
   legible, proceed; one or more == 3 (none <= 2) -> fill the thin spots in `house-rules.md` then re-score;
   any <= 2 -> the local layer is too thin to build safely - distill `house-rules.md` first (and where the
   gap is upstream, raise it upstream rather than paper over it locally).
2. **Distill house-rules from the gaps, not from scratch.** A sub-4 dimension tells you exactly what to
   write: a missing conventions doc -> write the "How this repo works" section; undiscoverable patterns ->
   capture the dominant pattern (endpoint shape, data flow, where new code goes) by reading existing code;
   undocumented upstream -> link the upstream's own docs so future work re-derives from THEM. Each open gap
   is a `T-XXXX` row tracked in the template's "Legibility gaps" table until closed.
3. **Know what "derivable" means when the truth is upstream.** It does NOT mean "the house-rules doc is
   complete" - the house-rules doc is deliberately thin. It means a role can do its whole job by reading
   the UPSTREAM source plus the thin local layer, without guessing at a local convention. The legibility
   floor is the proxy for that: if every legibility dimension is >= 4, a new piece of work can be derived
   from upstream + house-rules without inventing a local rule. The house-rules doc's job is only to make
   the upstream legible and to record the few local deviations - never to restate upstream truth (that
   would create the drifting parallel spec this mode exists to avoid).

---

## 7. The spec-developer step (owned here, run before bootstrap)

The multi-session interview that develops a rough spec to derivable lives in the companion
`spec-developer` skill (`skills/spec-developer/SKILL.md`). It scores against the SAME 8-dimension rubric
in §4, runs in stateful rounds (focus the 2-3 lowest dimensions per round, ask 3-5 targeted questions,
write back, log, re-score), and declares "derivable" only when every dimension reaches >= 4. Bootstrap's
Phase-0 is the lightweight twin: it re-scores, patches a merely-partial spec inline, and routes a rough
spec back to `spec-developer` first. This module owns the precedence ladder, the modes, the Phase-0
rubric, and the extraction rules; the skill owns the interview mechanics.

---

## 8. hybrid / greenfield mode: grow the spec with the project

This is the mode where no spec exists yet, or the spec is a stub that will mature as the project does
(§2.3). It is NOT the absence of a source-of-truth - it is a source-of-truth that starts small and is
grown deliberately, never skipped. The two failure modes it guards against are opposite: over-specifying
a product you do not yet understand (waterfall paralysis), and building features whose rules live only in
the code (the truth quietly migrates into the implementation, and the spec - hence every downstream gate -
goes stale).

### 8.1 The Phase-0 variant for this mode

You do not need a fully derivable spec to begin - you need enough to build the FIRST feature without
inventing. Phase-0 here is a **scoped, fast** version of §4:

1. **Write the seed.** A one-page seed is enough to start: the vision (Dimension 1), a rough entity sketch
   (Dimension 2), and the tech direction (Dimension 5). Two or three paragraphs, not a document. Start it
   from `templates/spec-seed.template.md` - the seed IS the top rung of the ladder and grows into the
   master spec, so it carries the version row and a growth log from day one.
2. **Score only the dimensions the first feature touches.** Run §4's 1-5 scale, floor >= 4, but ONLY over
   the slice of dimensions the first feature actually needs - not all eight. A booking feature needs
   Entities + Workflows + the Business Rules that govern it at >= 4; it does not need the analytics
   dimension scored at all yet.
3. **Decide on that scoped floor (§4.2 keyed on the floor).** All touched dims >= 4 -> begin the feature.
   One or more == 3 (none <= 2) -> a short, focused `spec-developer` pass on just those dimensions, then
   begin. Any touched dim <= 2 -> a full `spec-developer` round on that dimension before any code, because
   that is exactly where the first feature would otherwise invent.

Untouched dimensions stay deliberately blank - that is correct, not a gap. They are scored when a later
feature first reaches them.

### 8.2 The promote-before-build mechanism (the load-bearing rule of this mode)

The discipline that keeps this mode from degenerating into "the code is the spec": **every meaningful
feature promotes its rules INTO the spec BEFORE it is built**, never after. Concretely, for each new
feature:

1. Run the scoped Phase-0 (§8.1) over the dimensions it touches; develop any sub-4 touched dimension first.
2. Write the feature's entities, workflows, and business rules into the spec - in the same four-bucket
   form (§4.1 Dimension 4) used everywhere else - and let the touched dimensions re-score to >= 4.
3. ONLY THEN build. The spec is the input to the build, never a write-up produced after it.

This ordering is what keeps the spec the oracle as the project grows: because each rule is written down
**before** the code that implements it, the spec-guardian and any independent reviewer always have a truth
to re-derive against, and the spec never lags the codebase. The precedence ladder (§1) fills in top-down
as this repeats: whatever spec exists is the top rung, and it earns more rungs (derived per-domain docs,
extracted api-spec/db-schema) as domains mature past the derivable bar - at which point those domains are
extracted exactly as in authored-spec mode (§5). A greenfield project that survives long enough converges
on authored-spec mode; the difference is only the starting point and the cadence.
