# <Project Name> - Spec Seed

> Hybrid / greenfield mode (`source-of-truth` convention §2.3 + §8). This is the SEED of the master spec,
> not a one-off planning doc: it is the top rung of the precedence ladder from day one, and it GROWS into
> the full master spec as the project does. Every meaningful feature promotes its rules INTO this file
> BEFORE it is built (§8.2) - never the other way round. Keep it lean now; it earns sections as domains
> mature past the derivable bar (then they are extracted exactly as in authored-spec mode, §5).

## Version

| Version | Date | Notes |
|---------|------|-------|
| v0.1    | `<date>` | initial seed |

---

# 1. Vision

<One paragraph: what this system is, for whom, why it exists. This is Dimension 1 - aim for >= 4: problem
+ users + at least one measurable success criterion + the one thing that must NEVER go wrong.>

This system complements (does NOT replace):
- `<external tool / nothing yet>`

### What Must NEVER Go Wrong
- `<the single most important invariant - the thing the system must always protect>`

---

# 2. Entity Sketch

<Dimension 2, rough form. Just enough to start the first feature - name the main "things" the system
manages and their key attributes. States, relationships, and lifecycle get filled in per-entity as the
feature that needs them is built (promote-before-build, §8.2).>

- `<Entity A>`: `<one-line purpose>` - key attributes: `<a, b, c>`.
- `<Entity B>`: `<one-line purpose>` - key attributes: `<a, b, c>`.

---

# 3. Tech Direction

<Dimension 5. Name what is decided; mark the rest "to decide in bootstrap." Aim for >= 4 before the first
feature so no implementer has to guess the stack.>

- frontend: `<framework / TBD>`
- backend: `<language + framework / TBD>`
- database: `<type + platform / TBD>`
- auth: `<approach / TBD>`
- external services: `<named, or none yet>`

---

# 4. First Feature (the one being built next)

<The scoped Phase-0 (§8.1) runs over ONLY the dimensions THIS feature touches; develop any sub-4 touched
dimension here BEFORE building. Untouched dimensions stay blank on purpose - that is correct, not a gap.>

## Touched dimensions + scores

| Dimension (only the ones this feature needs) | Score (1-5, floor >= 4) | Gap to close before building |
|----------------------------------------------|-------------------------|------------------------------|
| `<e.g. Entities>`                            | `<n>`                   | `<gap>`                      |
| `<e.g. Workflows>`                           | `<n>`                   | `<gap>`                      |
| `<e.g. Business Rules>`                      | `<n>`                   | `<gap>`                      |

## Workflows

<Trigger -> steps -> decision points -> outcomes -> what the user sees, for this feature's flows.>

## Business Rules (four buckets - promote BEFORE building)

### Constraint Rules (must NEVER happen)
- A `<entity>` cannot `<action>` if `<condition>`.

### Calculation Rules (how values are derived)
- `<field>` = `<formula>`; calculated from `<sources>`, never stored independently.

### Trigger Rules (what change causes what)
- When `<event>` -> `<consequence>`.

### Lifecycle Rules (state transitions)
- `<entity>` moves from `<A>` to `<B>` when `<condition>`.

---

# 5. Growth Log (what has been promoted into the spec so far)

<Append-only. One line per feature whose rules were promoted BEFORE it was built - the running record that
the spec led the code, never trailed it. When a domain here matures past the derivable bar, extract it to
its own `docs/<domain>-spec.md` (§5) and note that here.>

- `<date>` - `<feature>`: promoted entities/workflows/rules; touched dims re-scored to >= 4; then built.
