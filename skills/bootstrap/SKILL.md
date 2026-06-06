---
name: ai-os-bootstrap
description: Install the ai-os collaboration layer into a target project, end to end. Picks a source mode + profile, evaluates the source of truth (delegating the rubric to source-of-truth and the interview to spec-developer when weak), installs the profile's modules (templates + CLAUDE.md install blocks, honoring dependencies), GENERATES the agent fleet at the chosen tier, assembles CLAUDE.md + path-scoped files + settings.json, seeds the knowledge docs (GLOSSARY / MEMORY / state-docs / TRACKER), and commits. Use when starting ai-os in a project with /ai-os-bootstrap [path-to-source].
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# ai-os Bootstrap

Installs ai-os into a target project, wired and cross-referenced from message one - so the project
starts coherent instead of being bolted together incrementally.

## What this skill is (and is NOT)

This skill is an **orchestrator and a generator**, not a convention dictionary.

- The CONVENTIONS live in `modules/<name>/` (README + `convention.md` + `templates/`).
  This skill does **not** re-explain them - it **composes** them. When a phase needs a rule, it
  reads the owning module and references it. Re-inlining a module's text here is the cardinal sin:
  it forks the truth and goes stale (the failure that retired the old monolithic bootstrap).
- Its richness is the **orchestration logic** (source-mode + profile selection, dependency-honored
  install order, CLAUDE.md assembly order, knowledge-doc wiring) and the **generation logic**
  (turning the source analysis into a per-project agent fleet, path-scoped rules, and a tailored
  settings allowlist).

Read this skill top to bottom once; then execute the phases in order. Each phase names the module(s)
it delegates to and the artifacts it produces. Checkpoints marked **[confirm]** pause for the owner.
Upgrades the old naive bootstrap lacked are marked **[UPGRADE]** - folding these in is the whole point
of the rebuild, not the trim.

## Usage
```
/ai-os-bootstrap                 # interactive: pick source mode + profile
/ai-os-bootstrap <path-to-spec>  # authored-spec hint: start from this master spec
/ai-os-bootstrap <profile>       # product-full | client-embedded | solo-lite (skip the recommend step)
```

---

## The orchestration spine (read before executing)

ai-os modules are **pipeline stages of one loop**, not standalone helpers. The bootstrap is
**message-one of that loop**: it is the single point that seeds every other module already wired to
its siblings. Two binding artifacts thread the whole install and must stay consistent across phases:

1. **The precedence ladder** (owned by `source-of-truth`) - the one ordering every generated role,
   gate, and doc points into. Establish it ONCE in Phase 1; every later phase references it, never
   re-spells it.
2. **The shared glossary** (owned by `shared-language`) - the connective vocabulary
   (`external oracle`, `derivable`, `no-finding-dissolves`, `T-XXXX`, `anti-scope framing`,
   `standing gate`, `Output Contract`, ...). Seed it on day one so the shorthand exists from the
   first task. Every phase **references** these tokens; no phase **redefines** them.

The reliability spine the whole install hangs from: **the source-of-truth doc is the oracle; the
code is never the oracle.** A green check proves "the artifact does what the check asserts," not
"what intent requires." The only reliable catch is an **external oracle** - a reviewer who
re-derives expected behavior from the source-of-truth without reading the implementation. Every
module either feeds that oracle or is verified by it. Keep this quotable; do not re-argue it per
phase - cite it.

The dependency the phases honor end to end:

```
Phase 0  mode + profile        -> picks the bundle (which modules, which tiers)
Phase 1  source eval           -> establishes the precedence ladder + the Analysis block
   (delegates the rubric to source-of-truth; the interview to spec-developer if weak)
Phase 2  install modules       -> templates + install blocks, in dependency order
Phase 3  GENERATE the fleet    -> agent-fleet tier; per-domain/-layer/-platform roles from Phase 1
Phase 4  assemble governance   -> root CLAUDE.md + path-scoped CLAUDE.md + settings.json
Phase 5  seed knowledge docs   -> GLOSSARY + MEMORY index + state-docs + TRACKER (wired, not bare)
Phase 6  commit                -> one bootstrap commit (git-dev-push convention if installed)
Final    report                -> what installed, what was skipped (a-la-carte addable), next step
```

---

## Phase 0 - Source mode + profile selection (the front door)  [UPGRADE]

Before reading any spec, establish two toggles that drive the entire install. Ask inline, in prose,
with full background per option and an explicit recommendation (not a popup). Two decisions:

**Decision A - Source of truth.** Where does this project's truth come from?
- **authored-spec** - you author a master spec; this is the project's top-of-ladder truth.
- **external-monorepo** - the truth is an upstream framework/monorepo (a client/embedded repo over a
  shared codebase). Do NOT author a parallel spec; distill thin house-rules + point at the upstream.
- **greenfield** - no spec yet. Start lean; grow the spec as the project does.

If the user passed a `<path-to-spec>`, default Decision A to authored-spec and confirm.

**Decision B - Profile.** Recommend from Decision A, then confirm. Read `profiles/<profile>.md` for
the exact module bundle + tiers before recommending:
- authored-spec + owned product, reliability is the point -> **`product-full`**
  (all Core + `source-of-truth` authored / `test-program` full / `agent-fleet` full).
- external-monorepo / client repo -> **`client-embedded`**
  (all Core + `source-of-truth` external / `agent-fleet` lite; no `test-program`).
- small / personal / experiment / greenfield -> **`solo-lite`** (Core only).

High-stakes or AI-authored codebases bias UP a profile (toward full) - that is exactly where the
external oracle pays for itself.

**Print** the chosen mode + profile + the consequences (the exact module list that will be installed,
the agent-fleet tier, whether the test charter ships, which gates will be dormant) and **[confirm]**.
Note explicitly that anything the profile excludes is **addable a-la-carte later** via `os-install`.

---

## Phase 1 - Source evaluation + the precedence ladder

Delegate the rubric and the interview to the modules; this phase wires their outputs into the install.

### 1a - Evaluate the source (delegate to `source-of-truth` Phase-0)

Read `modules/source-of-truth/convention.md` and run its **source-agnostic Phase-0** for the chosen
mode. Do NOT re-implement the rubric here - it is the module's, imported:

- **authored-spec**: read the entire spec; score the **8 dimensions** on the shared **1-5 scale,
  per-dimension floor >= 4** (Product Vision / Entities / Workflows / Business Rules / Tech Stack /
  Roles & Permissions / Edge Cases / Integrations). Decide on the **floor**, not just the total:
  - all dims >= 4 -> the spec is **derivable** -> proceed to 1b.
  - any dim == 3 -> light development: targeted questions for the weak dimensions, patch the spec,
    re-score the patched dimensions, proceed.
  - any dim <= 2 / many low -> the spec is rough -> **route to `spec-developer`** (the `/spec-developer`
    interview that develops a rough spec to "derivable" across stateful rounds), then return here.
    Do not bootstrap a weak spec; a weak spec produces a weak AI layer.
- **external-monorepo**: score **codebase legibility** instead (is there a conventions doc? are the
  patterns discoverable? is the upstream documented?). Fill the gaps into a thin `docs/house-rules.md`
  before building. No parallel spec.
- **greenfield**: note there is no spec yet; start lean and record that in the state docs.

Ask all user-facing prompts in the **project's chosen language, parameterized** - never hardcode a
language. `[UPGRADE]` for the `full` profile, offer the optional **adversarial audit round** after
"derivable": a deliberate second pass that produces a numbered gap register (it routinely surfaces
whole missing features the first pass scored as "complete").

### 1b - Extract the Analysis block  [confirm]

Read the now-sufficient source and extract the structured block below. These booleans/lists drive
every downstream phase - **[confirm]** before continuing:

```yaml
Project Name:        <name>
Source Mode:         authored-spec | external-monorepo | greenfield
Source Path:         <master-spec path | upstream pointer | "none yet">
Source Version:      <version row, if any>
Profile:             product-full | client-embedded | solo-lite
UI Language(s):      <for parameterizing prompts + state-docs; e.g. project's owner-facing language>
Primary Language:    <TypeScript / Python / Go / ...>
Frontend:            <framework | none>
Backend:             <framework | none>
Database:            <postgres / mysql / mongo / none>
Managed Platform:    <Supabase / AWS / Firebase / Vercel / none + which>
OS / Shell:          <ask if unclear from context>
Directories:         [ <dir> -> <purpose> -> <lifecycle: verified | planned(phase N) | legacy> ]
Domains Found:       [ <domain> -> <one-line desc> + <significance: high | medium | low> ]
Has:                 Auth? · Scheduled Jobs? · Import/Export? · Mobile? · Web? ·
                     External APIs (+list)? · Managed Platform (+which)?
```

### 1c - Establish the precedence ladder (the spine)

From the Analysis + the chosen mode, write the **one** precedence ordering (highest first) that every
later phase references. Use the `source-of-truth` install block shape (fill the list + mode); do not
invent a private ordering anywhere else:
- authored-spec: `master spec > derived domain docs > AI-architecture doc > skills + CLAUDE.md > TRACKER > state files`.
- external-monorepo: `upstream + house-rules > local conventions > AI docs > state files`.

This ladder is the binding artifact threaded through Phases 2-5. Lower layers explain HOW to work,
never WHAT is true; on conflict, STOP and surface it - the highest-precedence source wins.

---

## Phase 2 - Install the bundled modules (templates + install blocks)

Install each module in the profile's bundle. The mechanism is fixed and uniform - **copy the
template, paste the install block** - never paraphrase the module's content:

For each module in `profiles/<profile>.md`, in **dependency order** (deps before dependents):

1. Read `modules/<name>/README.md`.
2. Copy its `templates/*` into the target at the path each template implies
   (e.g. `MEMORY.template.md` -> the native memory index path; `TRACKER.template.md` -> `docs/TRACKER.md`;
   `current-state.template.md` -> `docs/ai/current-state.md`; `GLOSSARY.template.md` -> `GLOSSARY.md`).
   Skip a template only if the file already exists - then merge, do not clobber.
3. Append its **Install** block (the blockquote in the README) to the target `CLAUDE.md` staging set
   (Phase 4 assembles them in canonical order; stage them now keyed by module).
4. **Honor dependencies.** If a module declares `soft-deps`/`hard-deps` that are in the bundle, ensure
   they install first. If a hard-dep is NOT in the bundle, STOP and surface it (the profile is
   inconsistent). A soft-dep absent from the bundle is fine (the module degrades) - note it.

**Install order (dependency-correct, for any profile):**
```
doc-hygiene -> shared-language -> memory -> tracker -> state-docs -> platform-notes ->
mutual-push -> sync -> handoff -> gates ->
[heavy, if in profile] source-of-truth -> test-program -> agent-fleet ->
[git-dev-push if/when it exists]
```
Rationale: the doc/knowledge substrate (hygiene, glossary, memory, tracker, state-docs) goes first
because every later module's install block + templates reference it; the behavioral protocols next;
the heavy machinery last because `agent-fleet` and `test-program` derive from `source-of-truth`.

**Never skip silently.** For each module the profile EXCLUDES, record one line in the final report:
"`<module>` not installed (profile `<x>`); addable a-la-carte via `/os-install <module>`."

---

## Phase 3 - GENERATE the agent fleet (tier from the profile)

Delegate the role grammar and the generation pattern to `agent-fleet`; this phase supplies the
**project-specific** inputs from the Phase-1 Analysis. Read `modules/agent-fleet/README.md`,
`modules/agent-fleet/templates/roles.md`, and the role FORMAT templates
(`implementer.format.md` / `specialist.format.md` / `platform-specialist.format.md`) first.
The tier comes from the profile.

### Tier: lite (`client-embedded`, or `solo-lite` + a-la-carte)
Install the two **review-gate roles only**, as a checklist (no 7-role ceremony):
- `spec-guardian` - the external oracle for **intent** (contradiction / invented business logic /
  derived-vs-stored truth / cross-domain / unresolved ambiguity).
- `code-reviewer` - the external oracle for **correctness** (regressions, edge cases, missing tests).
The reliability invariant even at lite: **the reviewer is never the author.**

### Tier: full (`product-full`)
Generate the whole fleet. Two parts:

**3a - The generic roles (ship as templates, stack-independent).** From `templates/roles.md`:
`orchestrator` (router / gate-manager / hub), `spec-guardian`, `code-reviewer`,
`current-state-specialist` (reality recorder), and `git-dev-push` (terminal summary-first push) IF
that module exists in the install (it is deferred today - if absent, the commit in Phase 6 is a plain
commit). These reference the project's `CLAUDE.md` to discover the source-of-truth **dynamically** -
**never hardcode doc paths or names** into a role (the stale-cross-ref trap).

**3b - The generated, project-specific roles.** Apply the role FORMAT templates
(`implementer.format.md`, `specialist.format.md`, `platform-specialist.format.md`) to the Phase-1 Analysis.
Every generated role uses the **universal skill-body shape** owned by `agent-fleet`:
`Use When` (trigger list) · `Required Sources` (ordered by the Phase-1c precedence ladder) · body ·
`Do Not` (>= 1 lane-discipline bullet) · `Output Contract`. Generate:

- **Implementers - one per tech layer** found in Phase 1 (e.g. an API implementer, a DB/migrations
  implementer, a web implementer, a mobile implementer). Title each with the exact stack. For paired
  peer implementers (web + mobile), add the **parity-gate** block symmetrically so neither drifts.
- **Domain specialists - one per high/medium-significance domain.** Interpret, do not implement.
  Each carries the always-present **"Surface effects on:" radar** (cross-domain implications) and an
  always-present **"ambiguities to escalate"** Output-Contract field, plus custody of any normative
  user-facing copy for that domain. (Skip low-significance domains - they fold into a sibling.)
- **Platform specialists - one per managed platform** found (if any). Use the six-field **Guidance
  Format** (Goal / Where / What-to-click / What-to-enter / Verify / Risks) and stub a
  `Project Context` block ("record current environment facts here").

Write each generated role to the project's skills dir (`.claude/skills/<name>/SKILL.md`).

**The seam to make match (the specialist -> implementer Output-Contract chain):** a specialist's **Output Contract IS the implementer's
input**; the implementer's Output Contract (incl. its **Required User Actions** field) **IS the
reviewer's + human's input**. Generate the contracts so they chain - this is what makes the roles a
fleet, not isolated prompts. Print the fleet roster (generics + generated, by type).

---

## Phase 4 - Assemble the governance layer (CLAUDE.md + path-scoped + settings)

### 4a - Root CLAUDE.md (canonical section order)
Compose the target `CLAUDE.md` from the staged install blocks (Phase 2) + the Phase-1 outputs, in
this fixed order. **Reference each shared protocol once** - do NOT triplicate the precedence list,
runtime model, or ambiguity protocol (the original bootstrap's worst noise):

1. **Authority line + Bootstrap read-order** - "read these first on meaningful work": the precedence
   ladder's top docs, the AI-architecture doc, current-state, owner-status, the TRACKER (scan the
   relevant open rows before proposing changes).
2. **Repo Shape** - the Phase-1 directories with their **lifecycle tags** (verified / planned / legacy).
3. **Source Of Truth** - the Phase-1c precedence ladder + the stop-on-conflict rule (the
   `source-of-truth` install block, filled).
4. **Engineering Baseline** - primary language + stack from Phase 1; "secrets backend-only".
5. **No Invented Business Logic + Hot Spots** - the rule, plus a **hot-spots list** (the project's
   deprecated/removed fields and counter-intuitive semantics surfaced during Phase 1 - the
   no-invented-logic footgun list; seed it even if short).
6. **Required User Actions** - the human-in-the-loop class (say-so / why / steps / what to verify).
7. **[UPGRADE] Standing Gates** - the `gates` install block (all gates, each with trigger + procedure
   + dormancy clause for infra-dependent ones + inline rationale).
8. **[UPGRADE] Shared Language** - the `shared-language` install block (the glossary pointer, live
   from message one).
9. **The remaining installed-module blocks** (memory, state-docs, tracker, handoff, sync, mutual-push,
   doc-hygiene, platform-notes, test-charter if full) - each appended once.
10. **Platform Notes** - the `platform-notes` block + the Phase-1 OS/shell facts + the
    semantic-line-break reminder (`doc-hygiene`).
11. **Path-Scoped Rules index** - links to the per-directory CLAUDE.md files (4b).

### 4b - Path-scoped CLAUDE.md (one per major layer)
For each significant directory (backend, web, mobile, db, ...), emit `<dir>/CLAUDE.md` on the 5-block
skeleton: **Domain** (which spec is primary for this layer) / **Conventions** / **Required-per-new-
`<unit>` checklist** (e.g. "every new table needs RLS + PostgREST grant verification") / **Do Not** /
**Environment**. Keep these thin - they complement, never duplicate, the root.

### 4c - settings.json  [UPGRADE]
Emit a committed `.claude/settings.json` on the **allow-the-recoverable / deny-the-irreversible-and-
secret-touching** principle:
- **Deny floor (always):** read+write of `.env*`, `rm -rf`, the force-push family (`--force` / `-f`),
  `git reset --hard`, `git clean -fd`. Generate the secret-file denylist consistently with the
  Engineering-Baseline "secrets backend-only" rule.
- **Allow:** Read/Glob/Grep/Edit/Write + the safe git verbs (status/diff/log/branch/fetch/add/commit/
  push/pull) + **stack-specific** run/test/build verbs keyed to the Phase-1 stack (node/pnpm, python/
  pytest, go, the DB/migration tool, docker, the test runner, the mutation tool if `test-program` full).
- **[UPGRADE] Native-shell mirror:** on a non-POSIX OS (Windows/PowerShell per Phase 1), mirror the
  allowlist in the native shell form so the same commands are pre-approved there too.
- Note the **two-tier model** in a comment: this committed baseline + a gitignored `settings.local.json`
  that accretes one-off noise to be periodically harvested into the baseline. Do NOT pre-seed the local
  tier with literals.

---

## Phase 5 - Seed the knowledge docs (wired, not bare)  [UPGRADE]

The templates were copied in Phase 2; this phase **wires** them so the substrate is live from message
one. Seed the **role-bounded doc set** - never two converging status files (the naive shape the source
project itself removed):

- **GLOSSARY.md** (`shared-language`) - keep the universal seed terms (external oracle, derivable,
  no-finding-dissolves, T-XXXX, anti-scope framing, standing gate, Output Contract, ...) AND add the
  project's sealed tokens surfaced in Phase 1. This is the connective tissue every role/gate/doc
  references rather than redefining.
- **MEMORY index** (`memory`, at the native auto-memory path) - the one-line-per-entry index, under
  its size budget. `[UPGRADE]` seed it with: a one-line pointer to the **glossary memory** (so the
  shared language is recallable cross-session) and the **gate-rationale memories** (each standing
  gate's "established because X cost Y"). One line each - detail lives in topic files.
- **State docs** (`state-docs`), each with its **role-boundary header naming its companions**:
  - owner-facing `PROJECT_STATUS.md` (root, concise, "read first," the project's language).
  - AI-facing `docs/ai/current-state.md` (detailed, English): **What's Done** = the bootstrapped
    artifacts list (sub-specs, fleet roster, CLAUDE.md set, settings, seeded docs) / **Pending** =
    first implementation not started / **Blockers** = none / **Required User Actions** = anything
    surfaced in Phase 1 (platform setup, secrets, migrations) / **Next Step**.
- **TRACKER** (`tracker`, `docs/TRACKER.md`) - the `T-XXXX` ledger with the category taxonomy +
  `USER_ACTION_REQUIRED`, severity, status, source, milestone anchor, `last_updated`, and the
  "when to read this" block. **Pre-seed** any `USER_ACTION_REQUIRED` items surfaced in Phase 1
  (e.g. apply migration X, configure secrets, verify grants).
- **Test charter** (`test-program`, full profile only) - the scenario-list + genre-taxonomy reference
  docs; and mark the **Regression-Coverage + per-stage independent-review gates dormant until the test
  harness lands** (they are conditional on `test-program` infra existing).
- **HANDOFF.md is NOT created now** - it is produced on the first `תארוז` (handoff trigger). The
  `handoff` template exists in-repo; the live file appears on first pack.

All long-form docs obey **semantic line breaks** (`doc-hygiene`) and the **lean + rolling-archive**
convention written into their headers (snapshot to `docs/archive/<doc>-<date>.md` at budget, rewrite
the live doc lean). Headline counts in any doc are **computed roll-ups** (a sum of per-group rows),
never hand-edited.

---

## Phase 5b - Record the install + self-verify  [UPGRADE]

Prove the layer is intact and give the owner a definite GREEN, not a guess, BEFORE committing.

- **Write `ai-os.lock`** at the project root: the ai-os version, the `source:` checkout path used, the
  chosen profile + source-mode, and the installed-module list (+ tier + version each, read from each module's manifest). This is the record that
  `os-doctor` and future `os-install` runs read to know what is present.
- **Run `/os-doctor`** (or its checks inline): for every installed module confirm its install block is
  in `CLAUDE.md`, its templates are at their target paths, its `ai-os:manifest` hard-deps are installed,
  its soft-deps are installed-or-noted, and nothing references an un-installed module. **Fix any miss
  now** (a dropped template, a missing dep) - do not commit a red install.
- Carry the GREEN/ISSUES result into the Final report.

---

## Phase 6 - Commit

Run the standing **pre-commit alignment audit** (re-read the install decisions vs what actually
landed; READ the files, do not trust recollection; fix gaps) before committing.

- If `git-dev-push` is installed, use its convention (summarize since last push -> stage -> commit).
- Otherwise, a plain commit: stage the installed layer and commit
  `chore: install ai-os (<profile>)`.

Commit only when the owner asks, or per the project's push norm. If on the default branch, branch
first per the project's git rules.

---

## Final report

Print a concise tally - this is the skill's return value, so keep it parseable:

```
ai-os bootstrap complete: <Project Name>  (mode <source-mode>, profile <profile>)

Installed modules:     <list, in install order>
Excluded (a-la-carte):  <list> -> add later via /os-install <module>
Source eval:           <derivable | developed via spec-developer | external house-rules | greenfield>
Sub-specs / house-rules: <N files in docs/>
Agent fleet (<tier>):  generics <list> | implementers <list> | specialists <list> | platform <list>
Governance:            CLAUDE.md (root) + <N> path-scoped + .claude/settings.json
Knowledge docs:        GLOSSARY.md · MEMORY index · PROJECT_STATUS.md · docs/ai/current-state.md · docs/TRACKER.md
Dormant until built:   <regression-coverage + independent-review gates, if test-program full>
Required user actions: <list from Phase 1 / Phase 5 - migrations, secrets, platform setup, verifications>
Health (os-doctor):    <GREEN - all N installed modules present, wired, deps satisfied | ISSUES + the fixes>

Recommended next step: <commit if not done> ; then /orchestrator <first task>   (message-one of the work loop)
```

If any spec gaps remain, or platform-setup actions are required, list them **explicitly** - do not
declare the layer "ready" while a required user action is outstanding.
