# tracker

> The deferrals + findings ledger: every "later" and every finding gets a durable `T-XXXX` id,
> scanned before new deferrals, never silently dropped.

**Status:** since v0.1
**Dependencies:** `standalone`.
Soft-deps: `state-docs`, `memory` (the Project-Knowledge family).

<!-- ai-os:manifest
version: 0.1.0        # bumps (semver) when this module's content changes; absence is fine (older installs have none)
tier: core            # core | heavy
deps: { hard: [], soft: [state-docs, memory] }
tiers: [single]       # [single]  OR  [lite, full]  OR named e.g. [authored-spec, external, hybrid]
-->

---

## What it gives you

- `docs/TRACKER.md`: `T-XXXX` rows with category / severity / source / status.
- The no-finding-dissolves rule and the scan-before-deferring rule.
- A separate Resolved section so the live ledger stays short.

## Install

Paste into the project's `CLAUDE.md`:

> ### Tracker
> Every deferral AND every finding gets a durable `T-XXXX` row in `docs/TRACKER.md`
> (category: deferral | finding | user-action; + severity, source, status). Nothing lives only in
> chat (no-finding-dissolves). Scan the relevant open rows BEFORE proposing a new deferral. Move
> resolved rows to the Resolved section; "done" is never a count - it is rows dispositioned.

## Generates in target

- `docs/TRACKER.md` (from the template).

## Files it scaffolds

- `templates/TRACKER.template.md`.

## Why

Work gets deferred and findings get raised constantly; without a durable ledger they dissolve into
chat history and silently disappear. A scanned, id'd registry is what makes "we'll do it later"
honest, and what makes "done" mean dispositioned rather than forgotten.
