# House Rules - <project>

> The truth is the upstream `<monorepo / framework>`. This is a THIN local layer, NOT a parallel spec.
> Light by default; deepen only on explicit request. A parallel spec would drift from the upstream and
> create two contradicting truths - which is exactly the failure this mode exists to avoid.

## Upstream source of truth

- **Primary:** `<link / path to the upstream repo + its docs>`
- **What it governs:** `<which decisions defer entirely to upstream - framework conventions, data model,
  build, release, etc.>`
- **How to read it:** `<where the upstream's own conventions/architecture docs live, so future work
  re-derives from THEM, not from this file>`

## Precedence here

1. The upstream source + its docs (above).
2. This `house-rules.md` (local distillation + deviations).
3. Local conventions / state files.

If this file and the upstream disagree, the upstream wins and this file is corrected (same stop-on-conflict
rule as authored-spec mode, one rung down).

## How this repo works (distilled patterns)

<Only the few conventions that actually matter locally - structure, data flow, where to add a new
feature, the naming/lint/test conventions this repo enforces. Keep it short; depth lives upstream.>

- `<pattern 1 - e.g. where new endpoints go and what they must register>`
- `<pattern 2 - e.g. how state flows from the data layer to the UI>`
- `<pattern 3 - e.g. the test + CI expectation before a PR>`

## Local deviations / additions

<Anything this repo does DIFFERENTLY from upstream defaults, or adds on top. This is the part future work
most needs - it is the surprise surface.>

- `<deviation 1 - what, and why>`

## Legibility gaps (from Phase-0)

<The codebase-legibility Phase-0 score lives here while gaps are open. Score the five dimensions of the
legibility rubric (`source-of-truth` convention §4.9) on the 1-5 scale, per-dimension floor >= 4:
conventions-doc presence, pattern discoverability, upstream documentation, example+test coverage, naming
consistency. Each sub-4 dimension is a gap; each open gap -> a T-XXXX. Close the blocking ones (by filling
this file - point at upstream docs, capture the discoverable patterns) before building.>

| Legibility dimension      | Score (1-5) | Gap note |
|---------------------------|-------------|----------|
| Conventions-doc presence  | `<n>`       | `<gap>`  |
| Pattern discoverability   | `<n>`       | `<gap>`  |
| Upstream documentation    | `<n>`       | `<gap>`  |
| Example + test coverage   | `<n>`       | `<gap>`  |
| Naming consistency        | `<n>`       | `<gap>`  |

- [ ] `<sub-4 dimension / gap>` -> `<T-XXXX>`
