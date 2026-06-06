# <Domain> Spec

> Derived from `<master-spec>` § N. **Master spec takes precedence on any conflict.**
> This document adds technical precision to `<domain>` behavior; it introduces NO new business truth.
> If a rule is needed that the master spec does not contain, that is a spec gap to fix in the master
> (and log a `T-XXXX`), never to invent here.

## Overview

<2-3 sentences: what this domain covers, its core purpose, and why it exists as its own domain
(it has its own entities + workflows/rules + edge cases - the three-part test for earning a sub-spec).>

## Entities

<For each entity owned by this domain. A DB implementer should be able to design the schema from this
table alone. Mark snapshot / derived fields explicitly - the derived-vs-stored boundary is where bugs
hide.>

| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID PK | |
| `<field>` | `<type>` | `<meaning, possible values, stored vs derived>` |

### States (if stateful)

<The state machine: each state + what it means + which transitions are legal. Forbidden transitions are
as important as allowed ones - name them.>

```
<state_a>   <what it means; what transitions in / out>
<state_b>   <...>
```

## Workflows

<Each major workflow for this domain: trigger -> steps -> decision points -> outcomes -> what the user
sees at each step. A frontend implementer builds the screens from this; a backend implementer builds the
handlers. Include concurrent-access behavior where two actors can touch the same row.>

### <Workflow name>
- **Trigger:** <what starts it>
- **Steps:** <step by step>
- **Decision points:** <branches / approval / confirmation gates>
- **Outcomes:** <terminal states + what the user sees>

## Business Rules

<Every rule stated as an explicit, unambiguous constraint - never implied. A spec-guardian compares
proposed work against these; a vague rule is one the guardian cannot enforce. Sort each rule into the
four buckets below.>

### Constraint Rules (must NEVER happen)
- A `<entity>` cannot `<action>` if `<condition>`.

### Calculation Rules (how values are derived)
- `<field>` = `<formula>`; calculated from `<sources>`, never stored independently.

### Trigger Rules (what change causes what)
- When `<event>` -> `<consequence 1>`, `<consequence 2>`.

### Lifecycle Rules (state transitions)
- `<entity>` moves from `<state A>` to `<state B>` when `<condition>`.
- `<entity>` cannot be deleted if `<condition>` - use `<archive/close/anonymize>` instead.

## Edge Cases

<For each workflow: invalid input, unauthorized action, missing data, concurrent access, external-service
failure. For each must-reject case, name the SPECIFIC failure (HTTP status + machine error code /
SQLSTATE), not "an error occurs" - this is what makes the case testable and reviewable.>

| Case | Expected behavior | Specific failure (status + code) |
|------|-------------------|----------------------------------|
| `<invalid input>` | rejected | `<4xx + machine_code>` |
| `<unauthorized>` | rejected | `<403 + machine_code>` |
| `<concurrent edit>` | <last-write-wins / optimistic-lock / reject> | `<...>` |

## Cross-Domain Dependencies

<Every other domain this one touches gets an explicit entry: what flows between them, what triggers what,
which side owns the data. This is the radar a domain specialist surfaces ("effects on: ...") and the
input the regression gate uses to find interaction edges.>

### <This domain> and <other domain>
- **What flows:** <data / events>
- **What triggers what:** <change here -> consequence there>
- **Ownership boundary:** <which side is the authoritative source>
