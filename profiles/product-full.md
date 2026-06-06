# Profile: product-full

> An owned product built to the highest bar (like the project this OS was distilled from).
> Everything, at full rigor.

**For:** a product you own and will grow for a long time, where reliability is the point.

## Modules (all)
Core: `shared-language` · `mutual-push` · `handoff` · `sync` · `memory` · `state-docs` · `tracker` ·
`doc-hygiene` · `platform-notes` · `gates` · `git-dev-push` *(deferred - not yet built; the commit step falls back to a plain commit)*
Heavy: `source-of-truth` **(authored-spec)** · `test-program` **(full)** · `agent-fleet` **(full)**

<!-- ai-os:profile
core: [shared-language, mutual-push, handoff, sync, memory, state-docs, tracker, doc-hygiene, platform-notes, gates]
heavy:
  - source-of-truth: authored-spec
  - test-program: full
  - agent-fleet: full
deferred: [git-dev-push]
-->

## Notes
- Phase-0: develop the master spec to "derivable" first (the spec-developer step).
- gates: all universal + the conditional ones (test-program is installed -> they activate).
- This is the gold-standard answer to "give me everything."

## Excludes
Nothing. To go lighter, drop `test-program` -> not installed, `agent-fleet` -> lite.
