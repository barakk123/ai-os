# ai-os - Catalog

The single readable map of everything in ai-os: every module, profile, and skill, with what each
gives you, its tier, and its dependencies. This is the knowledge base the `/ai-os-helper` concierge
reads, and a quick reference for humans.

> The machine-readable source of truth for tiers + dependencies is each module's `<!-- ai-os:manifest -->`
> block (and each profile's `<!-- ai-os:profile -->` block) - the installers read those. This catalog
> mirrors them in readable form; if they ever disagree, the manifest wins (regenerate this).

---

## Modules

`hard` deps are pulled in automatically by `/os-install`; `soft` deps are optional (the module works
degraded without them, richer with them). All modules currently declare `hard: []`.

### Core (every project)

| Module | What it gives you | Tier | Soft-deps |
|---|---|---|---|
| [shared-language](modules/shared-language/README.md) | a sealed glossary + the coin-it mechanism (one token = a whole process) | core | mutual-push, tracker |
| [mutual-push](modules/mutual-push/README.md) | the proactive two-way relationship + the coin-it reflex | core | shared-language |
| [handoff](modules/handoff/README.md) | the `תארוז` pack-for-a-fresh-chat protocol + HANDOFF templates | core | state-docs, memory, tracker, gates, sync, mutual-push, source-of-truth, doc-hygiene |
| [sync](modules/sync/README.md) | the `סנכרן` refresh-the-living-docs mode (a handoff minus the chat-swap) | core | state-docs, memory, tracker, handoff |
| [memory](modules/memory/README.md) | native auto-memory discipline: one-line index, format, budget, write/recall hygiene | core | state-docs, tracker, shared-language, doc-hygiene |
| [state-docs](modules/state-docs/README.md) | owner-facing + AI-facing where-are-we docs, lean + rolling-archive | core | memory, tracker, handoff, doc-hygiene |
| [tracker](modules/tracker/README.md) | the `T-XXXX` deferrals + findings ledger; no-finding-dissolves | core | state-docs, memory |
| [gates](modules/gates/README.md) | the standing review gates (each with procedure + rationale) | core | tracker, handoff, agent-fleet, source-of-truth, test-program |
| [doc-hygiene](modules/doc-hygiene/README.md) | semantic line breaks + no-em-dash + lean docs | core | - |
| [platform-notes](modules/platform-notes/README.md) | environment/tooling gotchas + verified fixes (TLS, shell, CI) | core | - |
| git-dev-push | terminal summarize-first commit/push convention | core | *deferred - not yet built* |

### Heavy (opt-in)

| Module | What it gives you | Tiers | Soft-deps |
|---|---|---|---|
| [source-of-truth](modules/source-of-truth/README.md) | precedence ladder + 3 source modes + the Phase-0 score rubric (+ the `spec-developer` skill) | authored-spec / external-monorepo / hybrid-greenfield | state-docs, agent-fleet, tracker |
| [test-program](modules/test-program/README.md) | the test charter: external-oracle, genre taxonomy, mutation tooling, prod-safety | lite / full | source-of-truth, gates, tracker, handoff |
| [agent-fleet](modules/agent-fleet/README.md) | the role set + gated workflow + per-project specialist generation | lite / full | source-of-truth, gates, state-docs, handoff |

---

## Profiles

| Profile | For | Modules |
|---|---|---|
| [product-full](profiles/product-full.md) | an owned product, reliability is the point | all core + source-of-truth (authored-spec) + test-program (full) + agent-fleet (full) |
| [client-embedded](profiles/client-embedded.md) | a client/embedded repo over an upstream monorepo | all core + source-of-truth (external-monorepo) + agent-fleet (lite); no test-program |
| [solo-lite](profiles/solo-lite.md) | a small/personal project | core only |

(`git-dev-push` is listed in every profile but deferred - the commit step falls back to a plain commit
until it ships.)

---

## Skills

| Skill | What it does |
|---|---|
| [`/ai-os-helper`](skills/ai-os-helper/SKILL.md) | **the concierge - start here.** Knows this whole catalog; explains at any level, recommends with reasoning, installs/verifies on request, routes to health. |
| [`/ai-os-bootstrap`](skills/bootstrap/SKILL.md) | install a whole profile end-to-end (mode + profile -> evaluate -> install -> generate fleet -> assemble CLAUDE.md + settings -> seed docs -> record + self-verify -> commit). |
| [`/os-install`](skills/os-install/SKILL.md) | install individual modules a-la-carte; dependency-aware, idempotent, self-verifying. |
| [`/os-doctor`](skills/os-doctor/SKILL.md) | health-check an install (every module present, wired, deps satisfied) -> GREEN/ISSUES + fixes. |
| [`/spec-developer`](skills/spec-developer/SKILL.md) | mature a rough spec to "derivable" across stateful interview rounds. |

---

> Full how-to: [docs/USAGE.md](docs/USAGE.md).
