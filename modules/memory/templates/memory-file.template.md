<!--
A single memory = one file = one fact, on Claude's NATIVE auto-memory path (do not reroute).
Copy ONE of the blocks below (per `type`), fill it in, save as <type>_<slug>.md, and add a ONE-LINE
pointer to MEMORY.md. Keep the body in the worked pattern: rule -> Why (incident + absolute date) ->
How to apply (numbered) -> [[wikilinks]]. Convert all relative dates to absolute. Semantic line breaks
(~100 chars, clause boundaries; never start a wrapped line with - * + > | # or N.).
-->

================================================================================
TEMPLATE A - `feedback` (how to WORK: a durable workflow rule, with the why)
================================================================================

---
name: kebab-case-slug
description: "The recall hook - phrase it as a trigger ('before every X ...', 'when work touches Y ...').
  Same text (or tighter) as the index line."
metadata:
  node_type: memory
  type: feedback
  originSessionId: <native - leave to the store>
---

<One tight paragraph stating the durable rule, in the imperative.>

**Why:** <the rationale PLUS the triggering incident and its ABSOLUTE date - rationale makes it stick, the
incident stops a future session from optimizing it away as arbitrary.>

**How to apply:**
1. <step - concrete enough to walk literally>
2. <step>
3. <step>

<Where this is mirrored, e.g. a CLAUDE.md gate.> Related: [[other-memory]], [[another-memory]].

================================================================================
TEMPLATE B - `project` (ongoing work state / constraint / decision NOT derivable from code or git)
================================================================================

---
name: kebab-case-slug
description: "The recall hook - the current headline state or the constraint, phrased so a future session
  knows when to open this."
metadata:
  node_type: memory
  type: project
  originSessionId: <native>
---

<The durable fact: the constraint / the sealed decision / the current workstream status.>

**Why:** <who decided it + the ABSOLUTE date + what it supersedes, if anything.>

**How to apply:**
1. <what to do given this fact>
2. <what to STOP and surface if the fact would be violated>
3. <when to re-surface this memory (e.g. at a specific milestone)>

<!-- If this is an ANCHOR for a multi-session workstream, also add: -->
## How to resume
1. Read MEMORY.md -> this file -> <the human-facing status doc> -> <the AI-facing state doc> -> <TRACKER>.
2. Current position: <the exact resume point>.

## Live status  (update on EVERY transition)
<a compact status block or matrix; roll old detail to a doc/archive when it grows>

Related: [[sibling-anchor]], [[tracker-location]].

================================================================================
TEMPLATE C - `reference` (a pointer to an external resource - short by design)
================================================================================

---
name: kebab-case-slug
description: "What this points to + when to consult it."
metadata:
  node_type: memory
  type: reference
  originSessionId: <native>
---

<One or two sentences: the resource, its location, and when to read it. No rule body needed - it is a
pointer, owned/defined by the module that owns the resource; this memory only routes to it.>

================================================================================
TEMPLATE D - `user` (who the person is / a standing preference about THEM)
================================================================================

---
name: kebab-case-slug
description: "The standing preference / fact about the person, in one line."
metadata:
  node_type: memory
  type: user
  originSessionId: <native>
---

<The durable preference or fact about the person (not about the work).>

**Why:** <when they stated it - ABSOLUTE date - and the reasoning, if given.>
