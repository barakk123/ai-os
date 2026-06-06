# source-of-truth

> Establishes WHERE the project's truth comes from, in WHAT precedence, and HOW that truth is
> matured to the point where downstream work can be DERIVED from it instead of invented.

**Status:** since v0.1  ·  **Tier:** floor (the precedence ladder + Phase-0 score + sub-spec extraction
ship in every project) / heavy opt-in (the multi-session `spec-developer` interview + the adversarial audit round)
**Dependencies:** `standalone`.
Soft-deps: `state-docs` (the state docs sit at the BOTTOM of the precedence ladder this module defines);
`agent-fleet` (every role's `Required Sources` points UP this ladder);
`tracker` (gaps surfaced during spec evaluation become `T-XXXX` rows);
consumed by the `bootstrap` skill (its Phase-0 IS this module's evaluation, and its Phase-2 IS this
module's sub-spec extraction).
Owns the `spec-developer` skill (the interview that matures a rough spec to **derivable**).

<!-- ai-os:manifest
tier: heavy            # core | heavy
deps: { hard: [], soft: [state-docs, agent-fleet, tracker] }
tiers: [authored-spec, external-monorepo, hybrid-greenfield]
-->

---

## The thesis this module serves

The source-of-truth doc is the **oracle**; the code is never the oracle. A green check proves "the
artifact does what the check asserts," not "the artifact does what intent requires." Every other module
in the OS appeals to a precedence-ordered truth ladder when it re-derives expected behavior. This module
is what BUILDS that ladder and matures its top rung (the spec) to the point where re-derivation is
actually possible. A weak spec poisons every gate downstream: the spec-guardian cannot catch invented
logic if the approved rules were never made explicit, and an independent reviewer cannot re-derive
behavior from a spec that does not contain it.

---

## What it gives you

- **The precedence ladder + stop-on-conflict rule** - the single ordering every module reads top-down.
  Define it ONCE here; every role's `Required Sources` references it rather than inventing its own order.
- **Three source modes** - authored-spec / external-monorepo / hybrid-greenfield - so the same discipline
  serves a client repo whose truth is an upstream monorepo, not just a project with an authored spec.
- **The source-agnostic Phase-0 evaluation** - the full 8-dimension rubric on a 1-5 scale, per-dimension
  floor >= 4, /40 total, WITH a per-dimension question bank and a score-decision table. (For
  external-monorepo mode, the same shape scores codebase legibility instead of a spec.)
- **Sub-spec extraction criteria** - when a domain earns its own derived doc, and how api-spec / db-schema
  / ux-flows are extracted.
- **The `spec-developer` skill** - the multi-session interview that takes a rough spec to derivable,
  with a stateful log (resume-from-state, decision register, contradiction register).

## Install

Paste into the project's `CLAUDE.md` (fill the precedence list + the chosen mode). The full block lives
in `templates/source-of-truth.section.md`:

> ### Source Of Truth
> Truth comes from `<mode>`. Precedence (highest first): `<master spec>` > `<derived domain docs>` >
> `<architecture doc>` > `<skills + CLAUDE.md>` > `<TRACKER>` > `<state files>`.
> If two sources disagree, STOP and surface the contradiction - the highest-precedence source wins.
> Lower layers explain HOW to work, not WHAT is true. Never treat a summary or a state file as stronger
> truth than the source it derives from.

## Generates in target

- A "Source Of Truth" section in `CLAUDE.md` (the precedence ladder + stop-on-conflict rule).
- mode authored-spec: a versioned master spec (top rung) + derived per-domain docs under `docs/`
  (each carrying the load-bearing precedence header "Derived from <master>. Master wins on conflict.").
- mode external-monorepo: a thin `docs/house-rules.md` distilled from the upstream (light by default),
  scored with the §4.9 codebase-legibility rubric.
- mode hybrid/greenfield: a lean seed spec (from `spec-seed.template.md`) that is the top rung from day one
  and grows into the master spec, each feature promoting its rules in before it is built.
- A `spec-development-log.md` alongside the spec (only while the spec is being matured).

## Files it scaffolds

- `templates/source-of-truth.section.md` - the CLAUDE.md precedence block.
- `templates/domain-spec.template.md` - the derived-doc skeleton with the precedence header (authored-spec
  mode).
- `templates/house-rules.template.md` - the thin upstream-truth layer for external-monorepo mode (carries
  the §4.9 codebase-legibility score table).
- `templates/spec-seed.template.md` - the lean seed spec that grows into the master, for hybrid/greenfield
  mode (carries the version row, the scoped first-feature Phase-0, and the growth log).
- `skills/spec-developer/SKILL.md` - the spec-maturing interview (this module owns it).

## Why

A single precedence ladder with stop-on-conflict is what prevents doc drift across a large project: when
two docs disagree you do not guess, you STOP and the higher rung wins. Making it source-agnostic lets the
same discipline serve a client repo whose truth is an upstream monorepo. And maturing the spec to
**derivable** BEFORE generating anything is what lets every downstream role do its whole job from the spec
alone - no asking, no inventing - which is the precondition for the external oracle to work at all.

## Sibling cross-references

- **bootstrap** consumes this module twice: its Phase-0 IS the evaluation defined here (one shared 8-dim
  scorer), and its Phase-2 IS the sub-spec extraction defined here. Bootstrap's Phase-0 is the lightweight
  twin - if the spec is rough it routes to `spec-developer` first; if partial it patches inline.
- **agent-fleet**: every generated role's `Required Sources` is an ordered slice of THIS ladder.
- **gates**: every gate that says "re-derive from source-of-truth, not summaries" appeals to this ladder
  and the stop-on-conflict rule; it does not re-argue the external oracle.
- **tracker**: spec gaps and contradictions found during evaluation become `T-XXXX` rows.
- **state-docs**: the state files sit at the BOTTOM of this ladder (they describe reality, never override
  product truth).
