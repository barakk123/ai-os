# Profile: client-embedded

> A client / embedded repo over a shared upstream monorepo (e.g. a frontend over a shared framework).
> All the good practices, none of the heavy machinery.

**For:** working inside someone else's codebase whose truth is an upstream monorepo, where you want
the collaboration OS but not a parallel spec or a full test program.

## Modules
Core (all): `shared-language` · `mutual-push` · `handoff` · `sync` · `memory` · `state-docs` ·
`tracker` · `doc-hygiene` · `platform-notes` · `gates` · `git-dev-push` *(deferred - not yet built; the commit step falls back to a plain commit)*
Heavy: `source-of-truth` **(external-monorepo)** · `agent-fleet` **(lite: review gates only)**

<!-- ai-os:profile
core: [shared-language, mutual-push, handoff, sync, memory, state-docs, tracker, doc-hygiene, platform-notes, gates]
heavy:
  - source-of-truth: external-monorepo
  - agent-fleet: lite
deferred: [git-dev-push]
excluded: [test-program]
-->

## Notes
- source-of-truth external: distill a thin `docs/house-rules.md` + point at the upstream
  (light by default). No parallel spec.
- agent-fleet lite: spec-guardian + code-reviewer as a checklist; no 7-role ceremony.
- gates: universal only (the test-program-conditional gates stay dormant).

## Excludes (add a-la-carte if wanted)
- `test-program` (even lite) - add it if the repo warrants a test charter.
- `agent-fleet` **full** - add it only if the repo is big enough to warrant generated specialists.
