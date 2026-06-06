# sync

> The sync update mode (fired by the project's sealed **sync trigger**; ai-os dogfoods `סנכרן`):
> refresh state-docs + memory + tracker to match reality, without packing for a fresh chat.

**Status:** since v0.1
**Dependencies:** `standalone`.
Soft-deps: `state-docs`, `memory`, `tracker` (it refreshes them); pairs with `handoff`.

<!-- ai-os:manifest
tier: core            # core | heavy
deps: { hard: [], soft: [state-docs, memory, tracker, handoff] }
tiers: [single]       # [single]  OR  [lite, full]  OR named e.g. [authored-spec, external, hybrid]
-->

---

## What it gives you

- A cheap, explicit "make the docs honest again" routine - a handoff minus the chat-swap.
- A clear boundary vs `handoff`, so the two are never confused.

## Install

Paste into the project's `CLAUDE.md`, substituting your project's sealed **sync trigger** for the
example token below (ai-os uses `סנכרן`):

> ### Sync (the project's sealed sync trigger; ai-os: `סנכרן`)
> On the sync trigger (or when the assistant notices the docs have drifted from reality): re-read the
> work since the last sync, then refresh the state-docs + memory + tracker to match what is
> actually true now. No warm-start brief, no chat-swap, no pack commit - just realign the
> living docs, and report what changed.

## Generates in target

- Nothing new; it updates the existing state-docs / memory / tracker.

## Files it scaffolds

- None (behavioral; relies on the Project-Knowledge modules).

## Why

State drifts from reality mid-work. A lightweight, named refresh keeps the docs trustworthy
between handoffs, so a later pack-handoff (ai-os's sealed `תארוז`) or a fresh chat starts from an
honest baseline instead of reconstructing it.
