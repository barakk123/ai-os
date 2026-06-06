# memory

> Durable cross-session knowledge on Claude's NATIVE auto-memory, with a discipline layer on top:
> a one-line index, a strict per-file format, a hard size budget, and explicit write/recall hygiene.

**Status:** since v0.1
**Dependencies:** `standalone`.
Soft-deps: `state-docs`, `tracker` (the Project-Knowledge group - memory points to them, never copies them);
`shared-language` (memory holds ONE pointer to the glossary memory, not a second copy of it);
`doc-hygiene` (memory obeys its semantic-line-break rule, never co-claims it).

<!-- ai-os:manifest
tier: core            # core | heavy
deps: { hard: [], soft: [state-docs, tracker, shared-language, doc-hygiene] }
tiers: [single]       # [single]  OR  [lite, full]  OR named e.g. [authored-spec, external, hybrid]
-->

> **Storage: Claude Code's native auto-memory at its standard paths - NOT rerouted, NOT renamed.**
> This module adds a discipline layer; it does not move the location or change the native file types.
> `feedback` stays a native type (owner-ruled: do not reroute it into `project`).
> (Non-Claude environments: a portable fallback location is an open question - do not force one.)

---

## What it gives you

- A `MEMORY.md` **one-line index** - the only memory file that is auto-loaded every session.
- A strict **per-memory file format** (one file = one fact) with native frontmatter and a worked body pattern.
- A concrete **size budget** for the index, with a compaction trigger (bloat = silent partial load = silent loss).
- **Write-time and recall-time hygiene** - what to save, what NOT to save, how to recall without trusting stale facts.

## Install

Paste into the project's `CLAUDE.md` (under a `## Memory` heading):

> ### Memory
> Durable cross-session knowledge lives in Claude's native auto-memory. `MEMORY.md` is the always-loaded
> **index**: one line per memory - a `[Title](file.md)` link plus a one-sentence recall hook (when is this useful?).
> Keep the index lean (it has a size budget; bloat means it loads only partially = silent memory loss).
> Each memory is **one file, one fact**, with native frontmatter (`name`, `description`,
> `metadata.type: user | feedback | project | reference`). For `feedback`/`project`, the body carries a
> **rule -> Why: (with the triggering incident + absolute date) -> How to apply: (numbered steps) -> `[[wikilinks]]`**.
> Before saving: (1) check for an existing memory that covers it - UPDATE, do not duplicate; (2) save only what is
> **non-obvious and durable** - never what the repo / git / specs already record; (3) convert relative dates to
> absolute; (4) add the one-line pointer to `MEMORY.md`. On recall: treat a fact as **true-when-written** - verify
> any named file / function / flag still exists before acting; delete memories that turn out wrong.

## Generates in target

- A `MEMORY.md` index stub at the native auto-memory path (if absent), seeded with the header + format reminder.
- The seeded bootstrap memories (the glossary pointer, the gate-rationale memories) appear here as one-line entries -
  written by the bootstrap module, indexed here.

## Files it scaffolds

- `templates/MEMORY.template.md` - the index header + the one-line entry format, with examples.
- `templates/memory-file.template.md` - a single-memory file skeleton (frontmatter + the worked body pattern),
  with a fully worked example of each type.

## Why

Memory is the cross-session backbone, and it fails two silent ways:

1. **Bloat.** The always-loaded index is read in full every session up to a token cap. In the proven project it grew
   past the cap (one entry alone ballooned to multi-KB) and then loaded only *partially* - so memories silently
   stopped being recalled, with no error. The one-line-index rule + a size budget is the fix.
2. **Staleness.** A fact that was true when written is trusted blindly later. The native store already stamps each
   recalled memory "point-in-time, verify before asserting" - the recall-hygiene rule turns that stamp into a habit.

The discipline layer - one-line index + size budget + write/recall hygiene - is what keeps memory an asset instead of
a liability. It feeds the rest of the OS: `state-docs`/`tracker` get pointed to (never duplicated), the glossary is
seeded as a memory so the shared language exists from message one, and `no-finding-dissolves` relies on nothing
living only in chat - durable knowledge lives here.
