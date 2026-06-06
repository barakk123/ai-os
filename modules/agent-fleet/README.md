# agent-fleet

> The single-responsibility role set + the strict gated workflow that routes meaningful work,
> plus the pattern the bootstrap uses to GENERATE per-project specialists, implementers, and
> platform specialists - in two tiers.

**Status:** since v0.1  ·  **Tier:** lite floor (ships everywhere) / full ceiling (opt-in for high-stakes or AI-authored codebases)

**Dependencies:**
- `soft` -> `source-of-truth` (every role's "Required Sources" points into the ONE precedence ladder it owns; roles never invent a private ordering)
- `soft` -> `gates` (the standing disciplines wrap the loop; the orchestrator invokes the review gates)
- `soft` -> `state-docs` (the orchestrator hands off to `current-state-specialist`; the TRACKER is the shared registry)
- `soft` -> `handoff` (the per-stage review gate produces the brief a fresh chat consumes)
- `standalone` otherwise - the lite tier (two review roles run as a checklist) works with nothing else installed

<!-- ai-os:manifest
version: 0.1.0        # bumps (semver) when this module's content changes; absence is fine (older installs have none)
tier: heavy           # core | heavy
deps: { hard: [], soft: [source-of-truth, gates, state-docs, handoff] }
tiers: [lite, full]   # [single]  OR  [lite, full]  OR named e.g. [authored-spec, external, hybrid]
-->

---

## What it gives you

The execution layer of the OS: the reliability thesis is **no single agent both interprets intent and
judges its own output**. Work flows through roles that each have ONE job, and the reviewer is never the
author. This is the build-loop half of the external oracle (the review-loop half lives in `handoff`).

- **lite** (the floor every project ships): two review-gate roles run as a checklist on meaningful work -
  `spec-guardian` (the intent oracle: contradiction / invented logic / derived-vs-stored / unresolved
  ambiguity) and `code-reviewer` (the correctness oracle: bugs / regressions / edge cases / weak verification).
  That single split - interpret vs judge - is the minimum that keeps reviewer != author.

- **full** (opt-in ceiling): the whole gated workflow
  (`orchestrator` -> domain `specialist` -> `implementer` -> `spec-guardian` -> `code-reviewer` ->
  `current-state-specialist` -> `orchestrator` approves), plus the generation pattern the bootstrap uses
  to emit this project's own implementers, domain specialists, and platform specialists from the
  source-of-truth analysis. Ships the 5 stack-independent generics as templates; generates the rest.

## Install

Paste into the project's `CLAUDE.md` (pick the tier; the bootstrap fills this in automatically):

> ### Strict Gated Workflow (tier: lite | full)
>
> **Meaningful work** = changes code, schema, API, or domain behavior.
>
> **(lite)** On meaningful work, run two review gates as a checklist before declaring done:
> `spec-guardian` (spec alignment + invented-logic risk, re-derived from the Source Of Truth - not from
> the summaries) and `code-reviewer` (correctness, regressions, edge cases, missing tests). The agent
> that wrote the change does NOT also sign off on it: switch roles deliberately for the review pass.
>
> **(full)** Route meaningful work through the gated workflow and do not bypass the gates:
> `orchestrator` (classify + select + source + gate-plan) -> domain `specialist` (interpret rules) ->
> `implementer` (write code) -> `spec-guardian` -> `code-reviewer` (only if code changed) ->
> `current-state-specialist` (if state changed materially) -> `orchestrator` approves.
>
> The reviewer is never the author. The independent oracle is the point.

## Generates in target

- A "Strict Gated Workflow" + "Role Boundaries" section in the root `CLAUDE.md`.
- The 5 generic skills, copied from this module's templates into the project's skills dir (stack-independent;
  they discover the Source Of Truth paths dynamically from `CLAUDE.md` - never hardcoded).
- **(full)** project-specific skills generated from the `*.format.md` templates: one implementer per tech
  layer, one domain specialist per significant domain, one platform specialist per managed platform.
- A "Role Set" + "Runtime Model" + "Domain -> Specialist Routing" block in the AI-architecture doc
  (the fleet's binding artifact - see `state-docs` / the bootstrap's Phase 5).

## Files it scaffolds

The **lite-tier set** is just the two review roles - `spec-guardian` + `code-reviewer` - run as a
checklist (no orchestrator, no separate implementer, no state role). Everything else below is **full-tier
only**. The tier markers on each line say which set the file belongs to.

- `templates/spec-guardian.skill.md` - the intent oracle (generic; copied as-is). **[lite + full]**
- `templates/code-reviewer.skill.md` - the correctness oracle (generic; copied as-is). **[lite + full]**
- `templates/orchestrator.skill.md` - the router / gate-manager / hub (generic; copied as-is). **[full]**
- `templates/current-state-specialist.skill.md` - the reality recorder (generic; copied as-is). **[full]**
- `templates/implementer.format.md` - the project-agnostic FORMAT the bootstrap fills per tech layer. **[full]**
- `templates/specialist.format.md` - the FORMAT filled per significant domain. **[full]**
- `templates/platform-specialist.format.md` - the FORMAT filled per managed platform. **[full]**
- `templates/roles.md` - a one-screen index of the role set + boundaries (drop into the architecture doc). **[full]**

(The terminal push role - `git-dev-push` - is its own Core module (deferred / not yet built); the
orchestrator hands off to it at the commit boundary. Until it ships, that step is a plain commit.)

## Why

The external oracle - an independent reviewer who is NOT the author, who re-derives expected behavior
from the source-of-truth without reading the implementation - is the core reliability mechanism of the
whole OS. It catches the gate-passing failure (a check that mirrors the code instead of the truth-doc)
that green CI cannot. The fleet is how that principle becomes a runtime: roles with one job each, a
router that plans the gates, and a hard rule that the writer never signs off on their own work.

The full 7-role ceremony is heavy and was often run implicitly even on the project it came from.
Tiering it keeps the reliability win (interpret != judge) as a cheap floor, and reserves the full
state-machine for codebases where the stakes - or the AI-authored blast radius - justify the overhead.
