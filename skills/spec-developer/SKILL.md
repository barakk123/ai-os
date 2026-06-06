---
name: spec-developer
description: Develop an incomplete master spec into a fully derivable project specification. Works in iterative, stateful rounds across multiple sessions, tracking progress in a log file. Use BEFORE bootstrap when the master spec is rough, partial, or missing key areas. Invoke with /spec-developer path/to/spec.md
allowed-tools: Read, Write, Edit
---

# Spec Developer

## Purpose

One job: take a weak or incomplete master spec and bring it to **derivable** - the level where a role can
do its whole job from the spec alone, no asking and no inventing (operationally: every one of the 8
dimensions scores >= 4). It does NOT bootstrap the project; when it finishes it hands off to the bootstrap
skill.

It scores against the SAME 8-dimension rubric the `source-of-truth` module defines (`convention.md` §4) -
one shared scorer. This skill owns the INTERVIEW mechanics: the rounds, the stateful log, the question
structure, the completion criterion.

---

## Usage

```
/spec-developer path/to/master-spec.md
```

Run as many times as needed. Each invocation continues from where the previous one left off - the log
file is the memory between sessions.

---

## Session start - always do this first

1. Read the spec file at the path provided in `$ARGUMENTS`.
2. Check for `spec-development-log.md` in the same directory as the spec.
   - **If it exists** -> read it fully. Continue from the current round. Do NOT re-ask any question
     already answered in the log. Resume from the recorded scores + open questions.
   - **If it does not** -> this is Session 1. Start fresh.
3. Print a one-line status (in the project's chosen language), e.g.:
   `Session <N> - current score: <X>/40 - round <R>`

---

## The derivable bar

The spec is **ready** when **every one of the 8 dimensions scores >= 4** (max 40). The bar is the
PER-DIMENSION floor, not the total: a single dimension at 2 blocks readiness even if the total looks
healthy, because that one weak dimension is exactly where a downstream role will be forced to invent.

The 8 dimensions, their 1-5 rubrics, their per-dimension "derivable test," and their question banks live
in the `source-of-truth` module's `convention.md` §4. Use them verbatim - do not fork the rubric. In
brief, the dimensions are:

1. **Product Vision** - why it exists + success criteria + what must NEVER go wrong.
2. **Entities** - each entity's attributes, states, relationships, ownership, stored-vs-derived.
3. **Workflows** - trigger -> steps -> decisions -> outcomes -> what the user sees.
4. **Business Rules** - every constraint precise, in the four buckets (Constraint / Calculation / Trigger
   / Lifecycle).
5. **Tech Stack** - frontend / backend / database / platform / auth / each external service named.
6. **Roles & Permissions** - what each role sees / can do / cannot do / whether it reaches others' data.
7. **Edge Cases & Errors** - invalid input, unauthorized, missing data, concurrency, integration failure;
   each with its specific failure.
8. **Integration Points** - each integration's purpose, data flow, and failure behavior.

---

## Round structure

Each round follows this exact sequence.

### Step 1 - Score all 8 dimensions
Read the current spec + the log's prior answers. Score each dimension 1-5 per the §4 rubric. Print the
table clearly (in the project's language):

```
| Dimension          | Score | Short gap note            |
|--------------------|-------|---------------------------|
| Product Vision     |  3    | missing success criteria  |
| Entities           |  2    | no states on entities     |
| ...                |       |                           |
| Total              | 18/40 |                           |
```

### Step 2 - Identify focus areas
Choose the **2-3 lowest-scoring dimensions** (or the dimension blocking the most downstream work). Depth
on a few areas beats breadth on all - do NOT try to fix everything in one round.

### Step 3 - Ask focused questions
For each chosen dimension, ask **3-5 targeted questions** drawn from that dimension's question bank in
§4. Question-quality rules:
- **Specific, not open-ended:** "What states can a `Document` be in?" not "Tell me about documents."
- **Binary or enumerable when possible:** "Can a client have multiple advisors, or only one?"
- **Build on what is already written** - never repeat a question already in the log.
- **Ask in the project's chosen language.**
- **Inline, with background** - present each question with enough context that the owner can answer
  without a round-trip; do not use a popup/multiple-choice widget.

### Step 4 - Write improved spec sections
After the owner answers, rewrite or expand the relevant spec sections. Rules:
- **Keep everything the owner already wrote** - only add and improve, never delete.
- Add clearly, under the correct `## Section` header, using terminology consistent with the rest of the
  spec.
- **If an answer contradicts something already in the spec, STOP and flag it explicitly before writing.**
  Resolve it WITH the owner; record the resolution in the log's Contradictions register. Do not silently
  overwrite the older text.
- Consult-before-edit: summarize the planned change and confirm before calling Write/Edit on the spec.

### Step 5 - Update the log
Record this round in `spec-development-log.md`: new scores, questions asked, sections improved, score
change, any new open questions, any contradiction found + how it was resolved.

### Step 6 - Check completion
If every dimension >= 4 -> the spec is forward-complete, but do NOT declare ready yet: run the
**pre-completion adversarial audit gate** below first. Otherwise briefly state what remains and invite the
next session. **Never declare ready while any dimension is below 4**, regardless of how many rounds have
passed - the score is the gate, not the round count.

### Step 7 - Pre-completion adversarial audit gate (required before declaring ready)
Every dimension at >= 4 means every section is *locally* complete on the forward pass; it does NOT mean the
spec has no systemic miss. Before the Completion Declaration, run ONE deliberate adversarial pass per the
`source-of-truth` module's `convention.md` §4.3: re-read the whole spec as a hostile reviewer trying to
break it, and produce a **numbered gap register** (every missing edge, unstated rule, ambiguous
transition). This routinely surfaces a whole missing feature the forward pass accepted because each
section looked fine in isolation. Disposition each gap - fix now, defer to a `T-XXXX`, or accept-with-reason
- and record the register + dispositions in the log. If a fix drops a dimension back below 4, return to the
round loop. Only when the register is fully dispositioned do you proceed to the Completion Declaration.
This is the spec-side analogue of the test-program's backward retro sweep.

---

## Log file format

Create and maintain `spec-development-log.md` in the same directory as the spec. This file is the
state-of-the-interview - it is what lets a later session resume without re-asking. It carries five
registers: the score table, the session history, the open questions, the contradictions, and the
adversarial-audit gap register (Step 7).

```markdown
# Spec Development Log - <Project Name>

## Current Quality Scores

| Dimension          | Score | Last Updated | Notes (the open gap) |
|--------------------|-------|--------------|----------------------|
| Product Vision     | X     | Round N      | <gap>                |
| Entities           | X     | Round N      | <gap>                |
| Workflows          | X     | Round N      | <gap>                |
| Business Rules     | X     | Round N      | <gap>                |
| Tech Stack         | X     | Round N      | <gap>                |
| Roles & Permissions| X     | Round N      | <gap>                |
| Edge Cases         | X     | Round N      | <gap>                |
| Integrations       | X     | Round N      | <gap>                |
| **Total**          | X/40  |              |                      |

## Completion Status
- [ ] Not ready - remaining gaps: <list the dimensions below 4>
- [ ] Ready - declared in Round N

## Session History

### Round N - <date>
- **Focus dimensions:** <list>
- **Questions asked:** <Q1 / Q2 / ...>
- **Sections improved:** <list>
- **Score change:** <before> -> <after>

## Open Questions (asked, not yet answered)
- [ ] <question carried to the next session>

## Contradictions Register
- <a contradiction found between an answer and the existing spec, and how it was resolved (or that it is
  still open). Never silently overwrite - record the decision here.>

## Adversarial Audit Gap Register (Step 7 / convention §4.3)
- [ ] <numbered gap found in the hostile re-read> -> disposition: <fixed now / deferred T-XXXX /
  accepted because ...>. Run once all dims >= 4, before the Completion Declaration; the spec is not ready
  until every row here is dispositioned.
```

---

## Completion declaration

When every dimension reaches >= 4 AND the pre-completion adversarial audit gate (Step 7 / `convention.md`
§4.3) has run with its gap register fully dispositioned, declare ready (in the project's chosen language):

```
The spec has reached a sufficient level.

Final scores:
<the score table>

Total: <X>/40

What we now cover:
- <key decisions now documented>
- <key entities now defined>
- <key workflows now described>

Open points that remain (NOT blockers - resolvable during implementation):
- <minor open questions>

The spec is ready.
Next: run the bootstrap skill on this spec.
```

---

## The four binding rules

- **Never invent.** If the owner has not answered, do not fill the answer in yourself. Mark it open in the
  log. An invented answer poisons every downstream role that derives from it.
- **Never delete.** Only add and improve. If something the owner wrote seems wrong, FLAG it - do not
  silently overwrite.
- **Never rush.** Do not declare "ready" while any dimension is below 4. The per-dimension floor is the
  gate, not the round count or the total.
- **Always flag contradictions.** If an answer contradicts the spec, say so explicitly before writing, and
  resolve it with the owner. Record the resolution in the Contradictions register.

Plus continuity: start every session by reading the log; never re-ask a question it already answers.

---

## Document structure & format

When writing or expanding the spec, use this proven structure and style - it is what makes a master spec
derivable into sub-specs, skills, and implementation docs. (The illustrative module names below -
"documents," "orders," "sessions" - are placeholders; use the project's own domain names.)

### Overall document structure

```
# <Project Name> - Master System Specification

## Version
| Version | Date | Notes |
|---------|------|-------|
| vN      | ...  | <one-line description of this version's change> |

> This is the only source of truth. Sub-specs and skills are derived from this document; update this
> file and bump the version row on every change.

# 1. Vision
# 2. System Boundaries (IN / OUT of scope + technology direction)
# 3. User Roles
# 4. <Main Entity / Core Module 1>
# 5. <Core Module 2>
# ...
# N-2. Data Model Overview + Entity Relationships
# N-1. Business Rules Matrix + Trigger Matrix
# N.   Derived Docs Readiness Pass
```

### Section 1 - Vision

```markdown
# 1. Vision
<One paragraph: what system this is, for whom, why it exists.>

This system complements (does NOT replace):
- <external tool 1>
- <external tool 2>

### Success Criteria
A deployment is successful when:
- <measurable criterion 1>
- <measurable criterion 2>

### What Must NEVER Go Wrong
- <invariant 1 - and where it is enforced (DB constraint AND service level).>
```

### Section 2 - System Boundaries

```markdown
# 2. System Boundaries

## IN SCOPE
- <feature>

## OUT OF SCOPE
- <what will NOT be built; what an external tool handles>

## 2.1 Official Technology Direction
- frontend / backend / database / auth / file storage / server-side logic - each named or marked
  "to be decided in bootstrap."
- Architectural constraints (what should NOT be assumed by default) + the product-level reason for the
  direction.

## 2.2 Product-Wide Non-Functional Principles
- <principle - e.g. low-friction workflows / explainable calculations / responsive support>. These are
  part of the product identity, not secondary niceties - they influence UX, schema, API, and architecture
  together.
```

### Section 3 - User Roles

```markdown
# 3. User Roles
- <role 1> / <role 2>

## 3.x Role Model Deep Dive
- Each role's typical responsibilities; what stays broadly available; what is role-weighted; what truly
  requires one role or extra confirmation. State the permissions philosophy (simple vs complex, and why).
```

### Section pattern - each core module / domain

Every domain section follows this pattern. Keep the main section light; move detail into Deep-Dive
subsections.

```markdown
# N. <Module Name>

## Overview
<2-3 sentences: what this module is, its core purpose.>

## Fields
- `field_name` (`type`): <meaning, possible values, stored vs derived>.

## States (if stateful)
- `state_name`: <meaning; what transitions into / out of it>.

## <Module Name> Deep Dive

### <Sub-topic - e.g. Creation Flow>
#### Core Principle / Rules
<what must be true>
#### Steps / Behavior
<step by step, or rule by rule>
#### Product Meaning
<WHY this rule exists - one paragraph, after any non-obvious decision.>

### <Sub-topic - e.g. Lifecycle / Status Semantics>
### <Sub-topic - e.g. Edge Cases / Operational Rules>

## <This Module> and <Other Module> Relationship
<How they connect, what flows between them, what triggers what.>

## <Module Name> UX Summary
<What the user sees and interacts with for this module.>
```

### Section pattern - business rules (the four buckets)

State rules as explicit, unambiguous constraints - never implied. Every rule is one of four buckets:

```markdown
## Business Rules

### Constraint Rules (must NEVER happen)
- A <entity> cannot <action> if <condition>.
- <entity> status cannot change from <X> to <Y> directly - must go through <Z>.

### Calculation Rules (how values are derived)
- <field> = <formula>; calculated from <sources>, never stored independently.

### Trigger Rules (what change causes what)
- When <event> -> <consequence 1>, <consequence 2>.

### Lifecycle Rules (state transitions)
- <entity> moves from <A> to <B> when <condition>.
- <entity> cannot be deleted if <condition> - use <archive/close> instead.
```

The contrast that defines "explicit" - the good/bad pair is the test, because the bad version is what a
downstream role is forced to invent against:
- Constraint: "A session cannot be confirmed if its duration is zero." (good) vs "Sessions should have
  valid durations." (bad - a guardian cannot enforce a "should.")
- Scope: "OUT: this system does not send invoices; an external accounting tool does." (good) vs "Focuses
  on scheduling." (bad - an unstated boundary is where scope-creep invents a whole feature.) Define what
  is OUT as clearly as what is IN.
- Deep-Dive: main section stays light - overview + fields + states - and the creation flow / lifecycle /
  edge cases live in `## <Module> Deep Dive` subsections. (good) vs dumping every rule into the main
  section so the one load-bearing constraint is buried in a wall of prose. (bad - what cannot be found
  cannot be derived against.)

### Section - Data Model Overview (near end)

```markdown
# N. Data Model Overview + Entity Relationships
## Core Entities         <all entities + one-line purpose each>
## Entity Relationships  <each related pair: relationship type, cardinality, ownership>
## Source-of-Truth Boundaries  <which entity owns which data>
## Stored vs Calculated Fields <which are stored directly, which are always derived>
```

### Section - Derived Docs Readiness Pass (always last)

This section evaluates whether the spec is mature enough to be split into derived docs. Always include it.

```markdown
# N. Derived Docs Readiness Pass

## Goal
Evaluate whether this master spec is mature enough to be split into derived implementation docs.

## Product Principle
Derived docs are created only when the master spec is a stable parent source. Downstream docs may add
technical precision but NOT new business truth.

## Readiness Scale
- Ready: enough stable product truth; only technical elaboration needed.
- Mostly Ready: core stable but some rules/fields need technical interpretation.
- Partially Ready: meaningful gaps remain that would force premature invention.

## Readiness Matrix
### <derived-doc-name>.md - Status: <Ready / Mostly Ready / Partially Ready>
- Why ready: <reasons>
- Still needs: <gaps>
- Generation intent: <what this derived doc should do with the master content>
```

### Writing-style rules (carry these into every section)

- **"Product Meaning" blocks** - after any non-obvious rule or design decision, one paragraph on WHY it
  exists (not what it is).
- **Constraints explicit, never implied** - the good/bad contrast above is the test.
- **Scope stated both ways** - IN and OUT, each named (see the Scope good/bad contrast above).
- **Deep Dives** - main section light (overview + fields + states); detail in subsections (see the
  Deep-Dive good/bad contrast above).
- **Cross-domain relationships** - every module that touches another gets an explicit relationship
  section (this is what the regression gate and the domain specialists read to find interaction edges).
