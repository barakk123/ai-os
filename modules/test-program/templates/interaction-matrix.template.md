# Interaction Matrix — cross-domain runtime edges

> A normative matrix of runtime cross-domain interactions — like a static conflict matrix, but for
> RUNTIME behavior. Each cell where two (or more) domains interact is a scenario that requires a test.
> Per-domain scenario lists catch a domain in isolation; this matrix catches what happens at the SEAMS,
> which is where the subtle, high-leverage bugs live. Full-tier (`convention.md` §5).
>
> This is a LIVING artifact: the stage closes only when every `now` edge is green OR pinned-with-a-tracker.

## How to build + maintain it

- **Forward (per domain):** when you cover domain D, enumerate every edge D × (already-covered domains)
  and add a row. Cover the edges that are buildable now; classify + defer the rest (scope taxonomy below).
- **Backward retro sweep (the bidirectional half of the regression gate — `convention.md` §4.3):** when
  covering D, ALSO re-examine the already-covered domains for edges that D (or a later domain) created but
  that were never back-filled. Run the FIRST backward sweep as a DEDICATED pass — it carries the whole
  accumulated backlog and is the largest; afterwards each run reconciles the priors against only the
  domains added since, so it stays small. Report it as its own visible step.
- **Most covered-domain edges are usually already green** if you tested in dependency order (dependencies
  before dependents). An honest "already-green" finding is a real signal — do NOT manufacture gaps to look
  thorough; record which edges were confirmed-already-green without re-writing them.

## The edge matrix

| id | models (× …) | scenario | asserts (spec-derived oracle) | genre(s) | status | finding |
|---|---|---|---|---|---|---|
| IM-01 | A × B | <what happens when A's event meets B's state> | <explicit expected outcome + side-effects> | interaction | absent | |
| IM-02 | A × B × C | <three-way edge> | <expected> | interaction | absent | |

**status:** `done` (coded + green) · `done (pin)` (current behavior pinned with an in-test divergence flag) ·
`absent` · `pinned-uncoded` (the decision exists in spec but is not implemented — a tracked gap) ·
`already-green` (confirmed covered by a prior domain's pass, not re-written).

## Scope taxonomy for deferred edges (WHY each waits — every deferred edge is classified)

- **(a) partner-built-not-yet-tested** — the partner domain's code EXISTS; the edge rides that domain's
  upcoming run. (Not a gap; a sequencing note.)
- **(b) partner-spec-only** — the partner domain is NOT yet implemented; the edge waits for its build.
  (Not a gap; a roadmap note.)
- **(c) spec-decided-but-uncoded-in-the-covered-code** — the KEY class: a product decision that lives
  ONLY in the spec and is NOT yet implemented in the covered code (e.g. a cascade that should emit a
  notification but the code emits none). These are REAL gaps. The sweep's primary value is making them
  VISIBLE as tracked findings — never hide a (c)-class edge under a bland "deferred".

## Deferred-edge register

| id | models | scope-class | tracker | unblocks-when |
|---|---|---|---|---|
| IM-… | A × B | (c) | T-XXXX | <when the emission surface / partner / decision lands> |

> NORMATIVE: a (c)-class row is a tracked gap to fix later (it does not block the current domain's
> closure, but it is NEVER silently dropped); (a)/(b) rows ride their partner's build. The stage as a
> whole closes only when every `now` edge is green or pinned with a live tracker.
