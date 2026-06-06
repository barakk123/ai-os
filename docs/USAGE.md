# ai-os - Usage Guide

How to actually use `ai-os`: get it into a project, take everything or just a piece, run the tools,
and confirm it works. If the [README](../README.md) is the 30-second front door, this is the manual.

---

## 0. Start here - the guided paste (recommended)

The fastest way in: let ai-os onboard you. Two copy-paste steps, no manual file-wrangling.

**1. In a chat in your project, paste this:**

```text
Set up the ai-os helper in THIS project - do only this, nothing else:
1. If an ai-os checkout isn't already reachable, clone https://github.com/barakk123/ai-os.git OUTSIDE this project (a sibling folder or ~/tools/ai-os), read-only, and note its ABSOLUTE path.
2. Install ONLY the helper skill: copy `skills/ai-os-helper/` from the checkout into this project's `.claude/skills/ai-os-helper/`. Install nothing else; do NOT modify my CLAUDE.md - no convention is activated by this step.
3. Confirm `.claude/skills/ai-os-helper/SKILL.md` exists.
4. Then tell me to OPEN A FRESH CHAT (so the /ai-os-helper command loads), and print EXACTLY the following line for me to paste there, with <AI-OS-PATH> replaced by the checkout's absolute path from step 1:

   /ai-os-helper Set up ai-os for THIS project. The ai-os checkout is at <AI-OS-PATH>; use it as your source of truth. Run your full audit-first setup and install only what I approve.
```

It clones ai-os, installs only the helper skill, and prints a one-line prompt for you to paste in a **fresh chat**. That second prompt runs the helper (now a real skill), which audits your project by capability against the current modules and installs only what you approve - never overwriting your own content.

The sections below are the deeper reference and the manual paths (by hand, a-la-carte, whole-profile) for when you want them.

---

## 1. The mental model

Four kinds of thing, that is all:

- **Module** - one atomic convention (e.g. `memory`, `handoff`, `gates`). A folder under `modules/`
  with a `README.md` (a copy-paste **Install** block + a machine-readable `ai-os:manifest`),
  a `convention.md` (the full rule), and any `templates/` (the files it scaffolds).
- **Profile** - a named bundle of modules for a project shape (`product-full`, `client-embedded`,
  `solo-lite`). A profile is just a manifest listing modules + tiers.
- **Skill** - a runnable tool: `ai-os-bootstrap` (install a profile), `os-install` (install modules
  a-la-carte), `os-doctor` (verify an install), `spec-developer` (mature a spec).
- **Manifest** - the machine-readable block the installers read: `<!-- ai-os:manifest -->` in each
  module README (its tier + deps), `<!-- ai-os:profile -->` in each profile (its module list).

You compose these. Take one module by hand, a few via `os-install`, or a whole profile via the bootstrap.
Whatever you pick, `os-doctor` tells you it is correct.

---

## 2. Make ai-os available (the deployment model)

`ai-os` is a **toolbox repo you keep checked out**. Three ways to use it, from zero-setup to full-auto:

### 2a. By hand - no setup at all
Works today, needs nothing installed. Open the ai-os checkout, find the module you want, and copy it in
(see [Path C](#5-path-c--take-one-convention-by-hand)). This is the foundation; the skills just automate it.

### 2b. With the skills, per project
Copy `skills/*` from the ai-os checkout into your **project's** `.claude/skills/`. Now `/ai-os-bootstrap`,
`/os-install`, `/os-doctor`, `/spec-developer` work inside that project. The skills read `modules/` from
the ai-os checkout - keep it reachable (they record its path in `ai-os.lock`).

### 2c. With the skills, everywhere
Copy `skills/*` into `~/.claude/skills/` (user-level). The tools are then available in **every** project.
Same `modules/` reachability rule.

> **The one prerequisite for the skills:** the ai-os `modules/` source must be readable when a skill runs
> (it reads each module's README + manifest + templates). If a skill cannot find it, it asks you for the
> ai-os checkout path. The by-hand path (2a) has no such dependency.

---

## 3. Path A - bootstrap a whole project

For a new (or newly-AI-fied) project where you want a coherent layer from message one.

1. Run **`/ai-os-bootstrap`** (optionally `/ai-os-bootstrap <path-to-spec>` or `/ai-os-bootstrap <profile>`).
2. **Phase 0** - it asks two things, with a recommendation: the **source mode** (authored-spec /
   external-monorepo / greenfield) and the **profile**. Confirm.
3. **Phase 1** - it evaluates your source of truth (scores a spec on 8 dimensions, or scores codebase
   legibility for a monorepo) and, if the spec is weak, routes to `/spec-developer` first.
4. **Phases 2-5** - installs the profile's modules (dependency-ordered), generates the agent fleet at the
   profile's tier, assembles `CLAUDE.md` + path-scoped files + `.claude/settings.json`, seeds the knowledge
   docs (GLOSSARY / MEMORY index / state docs / TRACKER).
5. **Phase 5b** - writes `ai-os.lock` and runs `os-doctor`; fixes any miss before committing.
6. **Phase 6 + report** - commits, prints a tally **including the `os-doctor` health line**.

Result: a wired, self-verified AI layer. Anything the profile excluded is addable later (Path B).

### Which profile?
| Profile | For | Gets |
|---|---|---|
| [`product-full`](../profiles/product-full.md) | an owned product, reliability is the point | all core + source-of-truth (authored) + test-program (full) + agent-fleet (full) |
| [`client-embedded`](../profiles/client-embedded.md) | a client/embedded repo over an upstream monorepo | all core + source-of-truth (external) + agent-fleet (lite); no test-program |
| [`solo-lite`](../profiles/solo-lite.md) | a small/personal project | core only |

You can always start lighter and add heavy modules a-la-carte later.

---

## 4. Path B - install modules a-la-carte

For an existing project where you want specific capabilities, not a whole profile.

1. Run **`/os-install <module>[,<module>...]`** - e.g. `/os-install shared-language,memory,handoff`.
2. It **auto-resolves dependencies** from each module's manifest: `hard` deps are pulled in automatically
   (it tells you); `soft` deps that are absent are noted as optional adds (the module still works).
3. It installs each (idempotent), updates `ai-os.lock`, and runs `os-doctor`.
4. You get a report: requested, auto-added deps, files created, CLAUDE.md blocks, and the health line.

Re-running is safe - it fills only what is missing and re-verifies (use it to repair a partial install).

**Example - "I just want the shared language + memory discipline":**
```
/os-install shared-language,memory
```
-> installs both, seeds `GLOSSARY.md` + the `MEMORY.md` index, adds their CLAUDE.md blocks, verifies GREEN.

---

## 5. Path C - take one convention by hand

No skills, no setup. For when you want exactly one practice and nothing else.

1. Open `modules/<name>/` in the ai-os checkout.
2. Read its `README.md`. Paste the **Install** blockquote into your project's `CLAUDE.md`.
3. Copy anything under its `templates/` to the path its **Generates in target** section names
   (e.g. `templates/GLOSSARY.template.md` -> your project root as `GLOSSARY.md`).
4. Check its manifest's `soft`/`hard` deps - if you want the full effect, grab those too (by hand or `os-install`).

That is the whole mechanism the skills automate. A module is always self-describing.

---

## 6. Anatomy of a module (so any module is readable)

Every `modules/<name>/` has the same shape:

- **`README.md`** - `What it gives you` · **`Install`** (the copy-paste CLAUDE.md block) ·
  `Generates in target` (what appears in your project) · `Files it scaffolds` (the templates) · `Why` ·
  and the `<!-- ai-os:manifest -->` block (tier + `deps: {hard, soft}` + tiers).
- **`convention.md`** - the full operational rule (procedures, worked examples) - the depth behind the
  install block.
- **`templates/`** - the actual files the module drops into a project.

Read the README to install it; read the convention to understand it.

---

## 7. The runnable skills

| Skill | When you run it | What you get |
|---|---|---|
| `/ai-os-bootstrap` | starting ai-os in a project | a whole profile installed, fleet generated, docs seeded, self-verified, committed |
| `/os-install <m,...>` | adding capabilities to an existing install | those modules + their deps, recorded + verified |
| `/os-doctor` | anytime - after install, or to re-check | a GREEN/ISSUES health report with concrete fixes |
| `/spec-developer` | your spec is rough and you want it `derivable` | a matured spec via stateful interview rounds + an adversarial audit |

All four are Claude Code skills - put them in `.claude/skills/` (per project or user-level) and invoke with `/`.

---

## 8. How you know it works (confidence)

This is deliberate, not an afterthought:

- **`ai-os.lock`** (written by bootstrap / os-install) records the ai-os version, the source path, and
  exactly which modules (+ tiers) are installed. It is your manifest of "what I pulled".
- **`/os-doctor`** reads that lock and checks, per module: the Install block is in `CLAUDE.md`, the
  templates are at their paths, and the manifest's dependencies are satisfied - then prints
  **`ai-os health: GREEN`** (explicitly) or the exact issues + one-line fixes.
- The installers run `os-doctor` themselves before finishing, so a fresh install reports its own health.

So at no point are you left with "a pile of files I think is set up." You get a definite answer, and a
command to fix anything that is off.

---

## 9. A worked example - `client-embedded` on an external-monorepo frontend

You work on a client frontend over a shared upstream monorepo. You want the
collaboration OS, no parallel spec, no heavy test program.

1. Put the skills in `~/.claude/skills/` (you reuse them across clients).
2. In the repo, run `/ai-os-bootstrap` -> Decision A: **external-monorepo**; Decision B (recommended):
   **client-embedded**. Confirm.
3. Phase 1 scores the repo's **legibility** (is there a conventions doc? are patterns discoverable?) and
   distills a thin `docs/house-rules.md` pointing at the upstream - it does **not** author a parallel spec.
4. It installs all core modules + `source-of-truth` (external) + `agent-fleet` (lite = spec-guardian +
   code-reviewer as a review checklist), assembles `CLAUDE.md`, seeds GLOSSARY/MEMORY/state-docs/TRACKER,
   writes `ai-os.lock`, runs `os-doctor` -> **GREEN**, commits.
5. Later you decide you do want a test charter: `/os-install test-program` -> it pulls its soft-deps'
   awareness, installs the lite tier, re-verifies. Done.

You did two commands and got a verified, coherent layer - tailored to a client repo, no spec ceremony.

---

## 10. Quick reference - "I just want X"

| I want... | Do this |
|---|---|
| the whole thing, best practice | `/ai-os-bootstrap` -> `product-full` |
| the OS in a client/embedded repo | `/ai-os-bootstrap` -> `client-embedded` |
| just the core conventions, nothing heavy | `/ai-os-bootstrap` -> `solo-lite` |
| only the shared language + shortcuts | `/os-install shared-language` (add `mutual-push` for the coin-it reflex) |
| only the memory + feedback discipline | `/os-install memory` |
| only the project-state docs | `/os-install state-docs` (pairs with `tracker`, `memory`) |
| only the handoff / pack-for-new-chat flow | `/os-install handoff` (pulls its companions) |
| only the standing review gates | `/os-install gates` |
| one convention, zero setup | copy `modules/<name>/` by hand (Path C) |
| to know my install is healthy | `/os-doctor` |
| to mature a rough spec | `/spec-developer path/to/spec.md` |

---

## 11. The manifests (for the curious / for tooling)

- **`<!-- ai-os:manifest -->`** (each module README) - `tier` (core/heavy), `deps: { hard, soft }`,
  `tiers` (e.g. `[lite, full]`). The installers read this to resolve dependencies and order installs.
- **`<!-- ai-os:profile -->`** (each profile) - the `core` list, the `heavy` modules + their chosen tier,
  and `deferred` / `excluded`. The bootstrap reads this to know the bundle.

These let `os-install`, `ai-os-bootstrap`, and `os-doctor` work from one machine-readable source instead of
parsing prose - which is what makes any selection install correctly and verify itself.
