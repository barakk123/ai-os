# handoff

> The warm-start brief that lets a fresh chat resume mid-task - or run an independent
> adversarial review - with no prior context, without silently dropping work.

**Status:** since v0.1
**Dependencies:** `standalone` (degrades gracefully if the siblings below are absent).
Soft-deps: `state-docs`, `memory`, `tracker` (it reads their pointers and is refreshed alongside
them), `gates` (Gate 7 - the Per-Stage Independent-Review gate - is what fires a handoff),
`sync` (the cheap subset that refreshes the same docs without packing), `mutual-push` /
`git-dev-push` (the commit convention used by the pack commit).

<!-- ai-os:manifest
version: 0.1.0        # bumps (semver) when this module's content changes; absence is fine (older installs have none)
tier: core            # core | heavy
deps: { hard: [], soft: [state-docs, memory, tracker, gates, sync, mutual-push, git-dev-push, source-of-truth, doc-hygiene] }
tiers: [single]       # [single]  OR  [lite, full]  OR named e.g. [authored-spec, external, hybrid]
-->

---

## What it gives you

- A defined trigger set + a **6-step pack protocol** (audit, refresh, compress, write, commit, notify).
- An **offer-vs-auto policy**: when the assistant proposes a pack and when it just runs one.
- Two artifact flavors built from one skeleton:
  - **`HANDOFF.md`** - a continuation OR independent-review brief (untracked, at the repo root).
  - **`handoff-prompt.md`** - the same content shaped as a ready **copy-paste opening message** for
    the next chat (paste-as-message-#1, then delete).
- The two things that make a handoff trustworthy, baked in and **repeated at each section header**:
  - the **anti-scope framing** (the brief is a partial warm-start, NOT the scope);
  - the **two completeness gates** (second completeness pass + backward retro sweep) as separate,
    visible, reported steps.

## Install

Paste into the project's `CLAUDE.md`:

> ### Handoff
> When a chat-swap is near (owner says "pack up" / "let's prep a new chat" / "the window is full",
> or proactively at a closed + verified + committed stopping point, or automatically when the context
> window is nearly full), produce a warm-start brief at the repo root (`HANDOFF.md`, untracked) and/or
> a ready copy-paste opening prompt. Run: (1) pre-commit alignment audit; (2) refresh state-docs +
> tracker + memory; (3) compress the memory index if it grew; (4) write the brief and/or the prompt
> (recommend which fits); (5) a proper pack commit; (6) notify the owner it is ready. The brief is a
> PARTIAL warm-start, NOT the scope - the next chat re-derives the full work list from the
> source-of-truth as if the brief did not exist, then cross-checks. Every brief prominently carries the
> anti-scope framing and the two completeness gates (second pass + backward retro sweep) as separate
> reported steps. At EVERY stage/domain boundary, prepare the independent-review variant proactively
> (unasked) and tell the owner "ready for a fresh-chat review of <stage/domain>" - see the
> Per-Stage Independent-Review gate.

## Generates in target

- A "Handoff" section in the root `CLAUDE.md`.

## Files it scaffolds

- `templates/HANDOFF.template.md` - the continuation / independent-review brief (one skeleton, two modes).
- `templates/handoff-prompt.template.md` - the copy-paste opening-message variant.

## Why

Chat-swaps are constant, and they are where work silently disappears. Two failure modes recur and must
be designed OUT of the handoff itself, not re-caught by the owner each time: (a) the next chat treats
the brief's seeds / file-map / prior conclusions as the scope and under-investigates; (b) the next chat
skips the second-pass / backward-sweep completeness gates. The anti-scope framing kills (a); the two
visible gates kill (b). The independent-review variant is the load-bearing seam of the whole OS: it lets
a reviewer who is **not** the author re-derive expected behavior from the source-of-truth and catch the
gate-passing failures a green check never will (the external oracle - see `gates` Gate 7 and the OS thesis).
