# state-docs

> The standing "where are we now" layer: an owner-facing status + an AI-facing current-state,
> each with a role-boundary header that names its companions, kept lean with a rolling dated archive.

**Status:** since v0.1
**Dependencies:** `standalone`.
Soft-deps (the Project-Knowledge family, all referenced - never owned - by state-docs):
`memory` (durable facts + the always-loaded index), `tracker` (`docs/TRACKER.md`, the deferrals/findings
ledger), `handoff` (`HANDOFF.md`, the episodic warm-start brief), `doc-hygiene` (semantic line breaks).

<!-- ai-os:manifest
tier: core            # core | heavy
deps: { hard: [], soft: [memory, tracker, handoff, doc-hygiene] }
tiers: [single]       # [single]  OR  [lite, full]  OR named e.g. [authored-spec, external, hybrid]
-->

---

## What it gives you

- `PROJECT_STATUS.md` (root, owner-facing, concise; the primary new-chat bootstrap; written in the
  project's chosen language).
- `docs/ai/current-state.md` (AI-facing, detailed, English; what is built / pending / decided / blocked,
  in depth, with a mandatory `Last updated:` stamp).
- A **role-boundary header** on each doc that names the other and names every sibling - so the two never
  re-converge and a reader always knows which doc owns what.
- The **lean + rolling-archive** lifecycle: overwrite (not append), snapshot to a dated archive at a size
  budget, then rewrite the live doc lean.

## Install

Paste into the project's `CLAUDE.md`:

> ### State Docs
> `PROJECT_STATUS.md` (owner-facing, concise) and `docs/ai/current-state.md` (AI-facing, detailed) are the
> standing "where are we now" docs - read first at session start, updated last after meaningful work.
> Each opens with a role-boundary header naming its companions, and each stays lean: when one approaches
> its size budget, snapshot the full file to `docs/archive/<name>-<YYYY-MM-DD>.md` and rewrite the live
> doc lean (now + next + blockers + pointers). They describe the **present only** - durable facts go to
> memory, deferrals/findings to the tracker, and the chat-swap brief to `HANDOFF.md`. They sit at the
> **bottom of the source-of-truth precedence ladder**: they describe reality, they never override product
> truth; if a state doc disagrees with the spec, the spec wins and the state doc is the thing that is wrong.

## Generates in target

- `PROJECT_STATUS.md` and `docs/ai/current-state.md` (from the templates), each with its role-boundary
  header pre-filled to point at the project's companion docs.

## Files it scaffolds

- `templates/PROJECT_STATUS.template.md` (owner-facing, the `Now / Next / Blockers` shape + the header).
- `templates/current-state.template.md` (AI-facing, the built/pending/decided/blocked sections + the
  `Last updated:` stamp + the header).

## Why

A new chat (and the owner) need one honest, concise "where are we" without re-reading the whole history.
Two docs with **distinct audiences** (owner vs AI) keep each one fit for its reader: the owner wants
"current + next + blockers" in their language; the AI wants depth (decisions, constraints, pending edges).
The role-boundary headers are what stop them from drifting back into one bloated catch-all.

The lean-and-archive rule is the other half: in the proven project these docs silently grew to **94KB and
127KB single files** - so large they only partially loaded into a session, which is worse than missing,
because the reader does not know what they did not see. Snapshotting to a dated archive and rewriting lean
keeps the live signal high while losing nothing (the full narrative is preserved verbatim in `docs/archive/`).

**Anti-pattern this module deliberately corrects:** the naive shape is *two converging status files* (an
owner status and a generic English "project status" that slowly becomes a second copy of the same content).
The proven project shipped that, found the redundancy, and **removed the duplicate**. Ship the role-bounded
set with non-overlapping jobs instead; never seed two generic status files.
