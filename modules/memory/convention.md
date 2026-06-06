# memory - convention

The operational layer on top of Claude's native auto-memory: the index format, the per-file format,
the size budget, the topic-file pattern, and write/recall hygiene - each with a worked example.

---

## 0. Storage (do NOT change)

Memories live in **Claude Code's native auto-memory at its standard paths**. This module adds discipline ON TOP;
it does NOT reroute, rename, or relocate the store, and it does NOT change the native frontmatter shape. In
particular `feedback` stays a native `type` (owner-ruled: do not fold it into `project`). Everything below describes
*how to use* that native store well - never *where* to put it.

> Note on the native store's behavior you must design around:
> - `MEMORY.md` (the index) is **auto-loaded in full every session, up to a token cap**. Past the cap it loads
>   *partially and silently* - the back half of the index is simply not seen. This is the single most important
>   constraint and the reason for the one-line rule + size budget below.
> - Each individual memory file is loaded **on demand** (when its topic becomes relevant), and the native store
>   prepends a system-reminder like *"this memory is N days old ... verify against current code before asserting."*
>   The recall-hygiene section turns that reminder into a rule.
>
> The native store auto-manages `MEMORY.md` (it can append entries and re-order them on its own). The one-line-index
> discipline does not fight that - it constrains the *shape* of whatever ends up there (one hook line per memory,
> detail in the files), so auto-added entries stay within budget; periodically re-fit any auto-grown entry back to one
> line in a compaction pass (Section 6).

---

## 1. The index (`MEMORY.md`)

The ONLY always-loaded memory file. It is a flat list of one-liners - a router, not a knowledge base.

**Format - exactly one line per memory:**

```
- [Title](file.md) - one-sentence recall hook (WHEN is this useful, not just WHAT it is)
```

- `[Title](file.md)` - the link to the memory's own file (native slug name, e.g. `feedback_pre_commit_audit.md`).
- The **hook** is written for *relevance during recall* ("before every commit ...", "when work touches the slot
  engine ...") - phrase it as the trigger condition, not as a bare label. A future session skims the index and must
  be able to decide, from this one line, whether to open the file.
- **Detail NEVER lives in the index.** No multi-clause paragraphs, no sub-bullets, no embedded decisions. The moment
  an entry needs more than ~one sentence of hook, that content belongs in the memory's file.

**Worked example (a healthy index):**

```markdown
# Memory Index

> The always-loaded index. One line per memory: a `[Title](file.md)` link + a one-sentence recall hook
> (when is it relevant?). Keep lean - detail lives in the memory files, never here.

- [Shared Language Glossary](feedback_shared_language_glossary.md) - the project shorthand (external oracle,
  derivable, standing gate, ...); read once so the terms mean the same thing every session.
- [Pre-Commit Audit](feedback_pre_commit_audit.md) - before every commit/push, reconcile all prompts since the last
  commit against what was actually written to files; fix gaps first.
- [No Finding Dissolves](feedback_no_finding_dissolves.md) - every review finding gets a durable tracked record;
  "done" is a dispositioned ledger + zero un-triaged findings, never a test count.
- [Master Spec](project_master_spec.md) - the single source of truth is `MASTER_SPEC.md`; update only it going forward.
- [Test Program](project_test_program.md) - status anchor for the test-program work; read before resuming any test run.
- [Tracker Location](project_tracker_location.md) - the deferrals + findings registry is `docs/TRACKER.md`; read it
  BEFORE proposing any new deferral.
```

**Anti-pattern (do NOT do this) - an index entry that swallowed a whole topic file:**

```markdown
- [Test Program](project_test_program.md) - Phases 1+2 COMPLETE: all 14 service domains + 14 schemas + the
  interaction matrix; suite 1604/95/0; now superseded by the reference-domain re-drive; T0/T1/T2 done; T3 step (a)
  goldens + (b) DST + (c) metamorphic +19 tests combined mutation 94.6% NEW findings logged; NEXT = step (d) ...
  [... 1,200 more words ...]
```

That entry alone can exceed the whole budget and push later entries past the cap, where they silently stop loading.
**The fix is mechanical: collapse the entry to one hook line; move every detail into the topic file.**

---

## 2. The size budget

The index has a **hard budget because it is loaded in full every session**. Treat it as a token budget, not a
line count - one bloated entry does more damage than fifty lean ones.

- **Budget:** keep `MEMORY.md` comfortably under the native auto-load cap - as a rule of thumb, **one screen of
  one-liners (~40-60 entries)**, and **no single entry longer than its one hook sentence**.
- **Why a hard budget:** in the proven project the index drifted past the cap because entries accreted detail
  (one entry alone reached multi-KB; the file reached ~27KB). Past the cap the index loads *partially* with no
  error - the back half of the list simply isn't recalled, so memories silently stop working. Collapsing every
  entry back to one line brought it to ~12KB and restored full load.
- **Compaction trigger:** when the index nears the budget, do a compaction pass (Section 6) - **before** adding the
  next entry, not after a crisis. Adding an entry to an already-over-budget index just pushes an older one off the
  cliff.

The discipline is simply: **index = pointers; files = knowledge.** As long as that holds, the budget holds.

---

## 3. A memory file (one file = one fact)

Each memory is its own file. The native frontmatter is fixed; the body follows a worked pattern for `feedback` and
`project` types. Keep one *fact* per file - if you're tempted to add a second unrelated rule, make a second file.

### 3.1 Frontmatter (native shape - keep exactly)

```yaml
---
name: kebab-case-slug
description: "The recall hook - one or two sentences, the same text (or a tighter version) as the index line."
metadata:
  node_type: memory
  type: feedback        # one of: user | feedback | project | reference
  originSessionId: <id> # native; leave as the store sets it - do not invent
---
```

- `name` - kebab-case slug; matches the file name's slug part.
- `description` - the recall hook. The native store surfaces this when deciding to load the file, so write it as a
  trigger ("before every commit ...", "when work touches X ...").
- `metadata.node_type: memory` - constant.
- `metadata.type` - the four native types:
  - **`user`** - who the person is / standing preferences about *them* (not about the work). Loaded broadly.
  - **`feedback`** - how to *work*: a durable workflow rule, with the why. The most common authored type.
  - **`project`** - ongoing work state / constraints / decisions **not derivable from code or git** (a roadmap
    position, a sealed owner decision, a "we apply nothing to prod yet" policy).
  - **`reference`** - a pointer to an external resource (a doc, a register, a URL) rather than a rule.
- `metadata.originSessionId` - native provenance; leave it to the store.

### 3.2 The body pattern (for `feedback` and `project`)

The worked shape that makes a memory durable and actionable:

1. **The rule** - one tight paragraph stating the durable fact, in the imperative for `feedback`.
2. **`Why:`** - the rationale **plus the triggering incident and its absolute date**. Rationale is what makes a rule
   stick; the incident is what stops a future session from "optimizing it away" as arbitrary.
3. **`How to apply:`** - a **numbered** procedure. Recall means walking this list *literally*, step by step - not
   acting from a vague memory of the gist (see Section 5).
4. **`[[wikilinks]]`** - link related memories liberally; a link to a not-yet-written memory is fine.
5. **A dated lapse/reinforcement note** (only if the rule is later reinforced or slipped) - appended, never a
   silent overwrite, so the history of *why it kept mattering* survives.

`user` and `reference` memories are usually shorter - a `reference` is often just frontmatter + a one-line pointer.

### 3.3 Worked example - a `feedback` memory

File `feedback_pre_commit_audit.md`:

```markdown
---
name: pre-commit-audit
description: "Before every commit/push, run a since-last-commit alignment audit: reconcile all prompts since the
  last commit against what was actually written to the files; fix/add gaps first. A standing part of the flow."
metadata:
  node_type: memory
  type: feedback
  originSessionId: <native>
---

Before committing or pushing ANY run / sub-task / item, perform a "pre-commit alignment audit": re-read every prompt
(user + assistant) since the last commit; extract everything discussed / concluded / decided / deferred; reconcile it
against what was actually written to the files. Fix or add anything missed BEFORE the push. Report findings (the fixes
applied, plus an explicit "nothing else outstanding" when clean).

**Why:** Made a standing gate after the audit caught real gaps twice in one week: an agreed design decision had not
been written to the spec, and a summary claimed "all N categories covered" when one was absent. Decisions reached in
conversation routinely fail to land in files, or land inaccurately; a disciplined re-read against the actual diff
catches the drift before it ships. [absolute date]

**How to apply:**
1. Identify the last commit; gather every user+assistant turn since then.
2. Extract each decision / agreement / deferral.
3. Cross-check each against the ACTUAL file content - READ the files; do not trust recollection of what you wrote.
4. Fix/add gaps; correct inaccuracies (honor [[consult-before-edit]] for any new design).
5. Report: fixes applied + "nothing else outstanding".
6. Only THEN commit/push.

Mirrored as a normative gate in `CLAUDE.md`. Related: [[consult-before-edit]], [[multi-file-updates]].
```

### 3.4 Worked example - a `project` memory

File `project_production_migration_strategy.md`:

```markdown
---
name: production-migration-strategy
description: "Apply NOTHING to production yet (no data, not live, keep it clean). At a later milestone: reset prod +
  bulk-apply ALL migrations in order. Local/test is the schema source of truth until then."
metadata:
  node_type: memory
  type: project
  originSessionId: <native>
---

Do not apply any migration to the production database right now. There is no production data and the system is not
live, so the cleanest path is: at a later, explicit milestone, RESET production and bulk-apply ALL migrations in
order in one pass. Until then, the local/test database is the authoritative schema source of truth.

**Why:** Owner ruled this on [absolute date] to avoid accumulating partial, hand-applied prod state that would later
be impossible to reconcile against the migration history. This SUPERSEDES the per-migration "apply this to prod now"
user-actions logged earlier (those are now no-ops until the reset milestone).

**How to apply:**
1. When a new migration lands, apply it to local/test only; do NOT add a "run in prod" user-action.
2. If a prod-only action seems required, STOP and surface it against this policy before acting.
3. Re-surface this memory when the team approaches the go-live milestone - that is when the reset + bulk-apply runs.

Related: [[tracker-location]] (the superseded user-actions are tracked there).
```

### 3.5 Worked example - a `reference` memory

A *pure* pointer often does not need its own file at all - the single index hook line IS the whole memory
(e.g. "Tracker location: the deferrals/findings registry is `docs/TRACKER.md` (owned by `tracker`); read it before
deferring."). Promote it to a standalone `reference` file only when the pointer carries more than one line of context
(stable-id rules, what-to-read-first, related links). When it does, the file looks like this:

File `reference_tracker_location.md` (short by design):

```markdown
---
name: tracker-location
description: "The deferrals + findings registry is docs/TRACKER.md; read it BEFORE proposing any new deferral."
metadata:
  node_type: memory
  type: reference
  originSessionId: <native>
---

The project's deferrals + findings registry lives at `docs/TRACKER.md` (stable `T-XXXX` ids, never reused, never
deleted). Read it BEFORE proposing a new deferral or logging a new finding, to avoid duplicates and to reuse an
existing entry where one fits. Owned/defined by the `tracker` module - this memory is only the pointer.
```

### 3.6 Worked example - a `user` memory

File `user_decision_questions_inline.md`:

```markdown
---
name: decision-questions-inline
description: "Owner prefers decision questions written inline in chat (full background per option + an explicit
  recommendation), NOT a popup/picker tool."
metadata:
  node_type: memory
  type: user
  originSessionId: <native>
---

When a decision needs the owner's input, write the question INLINE in the chat: lay out each option with full
background and tradeoffs, then give an explicit recommendation. Do not use a popup/picker UI for this.

**Why:** Owner stated the preference on [absolute date] - inline questions preserve the reasoning trail in the
transcript and let them respond in their own words.
```

---

## 4. The topic-file pattern (when a `project` memory IS the anchor)

Some `project` memories are **anchors**: the authoritative resume point for a large, multi-session workstream
(a roadmap, a long test program, a feature). The same one-file-one-fact rule holds - the "fact" is just "the live
state of this workstream" - but the body grows a few extra sections. **All of it stays in the topic file; only one
hook line goes in the index.**

A topic-file anchor typically carries:

- The **rule/summary** paragraph (what this workstream is, current headline status).
- A **"How to resume"** numbered list (which files to read, in what order, and the exact current position) - so a
  fresh session can warm-start from the anchor alone.
- A **live status block / matrix** that is updated on every transition (and an explicit "update on every transition"
  convention line, so it doesn't go stale).
- `[[wikilinks]]` to the sibling memories it depends on.

Discipline for anchors:

- **One anchor per workstream**, not one per run. New runs *update* the anchor's status block; they do not spawn
  new memories.
- **Roll, don't grow forever.** When the anchor's history section gets long, snapshot the old detail to the
  workstream's own doc (e.g. a `docs/<workstream>.md` or an archive), and keep the anchor lean: current status +
  how-to-resume + pointers. This mirrors the `state-docs` lean/rolling-archive convention - do not duplicate that
  convention here; obey it.
- **The anchor's index line is still ONE hook.** "Status anchor for the X workstream; read before resuming any X
  run." All the detail is inside the file, loaded on demand.

---

## 5. Write-time hygiene

Run this checklist BEFORE saving any memory:

1. **Dedup first.** Search existing memories for one that already covers this. If found, **UPDATE it** (append a
   dated reinforcement note) - do not create a near-duplicate. Two memories that say almost-the-same thing is how
   the index bloats and how a later session follows the stale one.
2. **Save only what is non-obvious AND durable.** Do NOT save what the repo / git / specs / CLAUDE.md already
   record - code structure, a past fix, project history, anything re-derivable. *Litmus test:* "could a fresh
   session reconstruct this by reading the code + git log + specs?" If yes, it is not a memory. If asked to
   "remember" such a thing, ask *what was non-obvious about it* and save only that.
   - **Save:** workflow rules with a why; sealed owner decisions; roadmap positions; constraints not in the code;
     the glossary; "we don't do X yet and here's why."
   - **Don't save:** "function `foo` is in `bar.ts`"; "we fixed the null bug last week"; "the API uses framework X"
     (that's in the code/CLAUDE.md).
3. **Convert relative dates to absolute.** "yesterday", "last sprint", "3 days ago" rot the moment the session ends.
   Write the calendar date.
4. **Pick the right `type`** (Section 3.1) and write the body in the worked pattern (rule -> Why+incident+date ->
   numbered How-to-apply -> wikilinks).
5. **Add the one-line pointer to `MEMORY.md`** - the hook phrased as a trigger condition. (A memory not in the index
   is effectively invisible - it is only loaded when the store independently decides it's relevant.)
6. **Budget check.** If the index is near budget, run a compaction pass (Section 6) before adding.

A note on multi-file consistency: a meaningful change usually touches the spec/derived doc + the human-facing status
+ the AI-facing state + the relevant memory + (if deferred) the TRACKER (owned by `tracker`) - in the SAME session,
so nothing goes stale. That cross-doc rule is owned by `state-docs`; this module's part of it is "the memory and its
index line are updated too."

---

## 6. Recall-time hygiene

A recalled memory reflects what was true **when it was written**. The native store stamps each loaded memory
"point-in-time ... verify before asserting" - treat that as binding:

- **Verify before acting on anything concrete.** Before relying on a recalled fact that names a file, function,
  flag, table, or count, **check it still exists / still holds** in the current code. Especially: never quote a
  remembered count, file:line, or "X is done" as current truth without re-checking.
- **Apply procedures literally.** When a memory has a numbered "How to apply", walk the steps in order, actually
  doing each - do not act from a fuzzy recollection of the rule's gist. The whole value of the numbered list is that
  it survives paraphrase decay.
- **Delete what's wrong.** If a recalled memory turns out false or superseded, fix or **delete** it in the same
  session - do not leave a stale fact to be re-recalled and re-trusted next time. (For a superseded *decision*,
  prefer updating the memory with a dated "superseded by ..." note over a silent delete, so the history survives.)
- **Trust the index as a router, not as truth.** The one-liner tells you a memory *might* be relevant; open the file
  for the actual content, then verify against code per above.

---

## 7. Maintenance

- **Compaction pass** (when the index nears budget): re-read every index line; collapse any that grew past one hook
  sentence (move the detail into the file); merge near-duplicate memories; drop memories that are now obviously stale
  or fully captured by code/docs. Goal: index back comfortably under budget, every entry one line.
- **Split overgrown files:** if a single memory file has accreted two unrelated facts, split it into two files (two
  index lines) - one file = one fact.
- **Archive superseded facts:** a memory whose content is wholly historical (a closed run, a superseded policy) can
  be condensed to a one-line "historical anchor" or removed if fully captured in an archive doc. Keep the live set
  about *what is true / what to do now*.
- **Re-seed on a fresh project via bootstrap:** the glossary memory + the gate-rationale memories are generated by
  the bootstrap module, indexed here, so the shared language and the standing-gate rationales exist from message one.

---

## 8. Cross-references (reference, do not redefine)

- **`tracker`** is the single owner of the `T-XXXX` deferrals + findings registry (and of `no-finding-dissolves`);
  **`state-docs`** owns the doc set + the lean/rolling-archive convention. Memory **points to** them (a `reference`
  memory for the tracker location, anchors that say "read TRACKER before deferring") - it does not restate their
  conventions and does not copy their content. Where an older note still credited the TRACKER to `state-docs`, the
  dedicated owner is `tracker`; cite `tracker` for anything `T-XXXX` / no-finding-dissolves.
- **`shared-language`** owns the glossary. Memory holds ONE `feedback` memory that *is* the glossary (or points to
  it), so the shorthand is auto-loaded - not a second copy.
- **`gates` / `no-finding-dissolves`** rely on "nothing lives only in chat" - durable workflow lessons and findings
  are recorded here (and in the TRACKER) rather than left in the transcript.
- **Semantic line breaks** apply to every memory file and to the index. This rule is **owned by `doc-hygiene`**
  (the dedicated owner) - memory only obeys it, never co-claims it: wrap prose at ~100-char clause boundaries,
  never one giant physical line, never start a wrapped line with a block trigger (`-` `*` `+` `>` `|` `#` `N.`).
  Identical render, readable/diffable source.
