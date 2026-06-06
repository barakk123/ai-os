# ai-os

A modular operating system of AI-collaboration conventions -
the durable "how we work together" layer that any project can adopt
**whole, by profile, or one piece at a time** - and confirm it actually works.

## Start here - paste this into Claude

**New here? This is the whole onboarding - don't skip it.** Two steps, both copy-paste, no docs to read first.

**1. In a chat in your project, paste this:**

```text
Set up the ai-os helper in THIS project - do only this, nothing else:
1. If an ai-os checkout isn't already reachable, clone https://github.com/barakk123/ai-os.git OUTSIDE this project (a sibling folder or ~/tools/ai-os), read-only, and note its ABSOLUTE path.
2. Install ONLY the helper skill: copy `skills/ai-os-helper/` from the checkout into this project's `.claude/skills/ai-os-helper/`. Install nothing else; do NOT modify my CLAUDE.md - no convention is activated by this step.
3. Confirm `.claude/skills/ai-os-helper/SKILL.md` exists.
4. Then tell me to OPEN A FRESH CHAT (so the /ai-os-helper command loads), and print EXACTLY the following line for me to paste there, with <AI-OS-PATH> replaced by the checkout's absolute path from step 1:

   /ai-os-helper Set up ai-os for THIS project. The ai-os checkout is at <AI-OS-PATH>; use it as your source of truth. Run your full audit-first setup and install only what I approve.
```

It clones ai-os, installs just the helper, and hands you the line to paste in a fresh chat. There the helper - now a real skill in a clean chat - audits your project against the current modules and installs only what you approve. **Nothing in your project is touched until you say yes.**

> Prefer to do it by hand? See [Quick Start](#quick-start) and the full [docs/USAGE.md](docs/USAGE.md).

---

> Status: **v0.1 - sealed + distributable** (13 of 14 modules; `git-dev-push` deferred). Versioned over time.

---

## What this is

Running real projects with an AI partner taught a set of lessons the hard way:
how to keep a shared language, how to keep memory and state docs lean and synced,
when and how to hand off to a fresh chat, which standing gates catch the failures a green test
suite misses, how to verify tests against intent instead of against the code.

`ai-os` captures each lesson as a **self-contained module** you can compose,
so the next project starts where the last one ended - not from scratch.
It is deliberately **not** one monolithic script. It is a registry of modules + installers.

---

## The mental model

- **Modules** (`modules/`) - the atomic units. One folder each: a `README` (with a copy-paste **Install**
  block + a machine-readable `ai-os:manifest`), a `convention.md`, and any `templates/`. Take one, several, or all.
- **Profiles** (`profiles/`) - named bundles for a project shape: `product-full`, `client-embedded`, `solo-lite`.
- **Skills** (`skills/`) - the runnable tools: `ai-os-helper` (the concierge - start here),
  `ai-os-bootstrap`, `os-install`, `os-doctor`, `spec-developer`.

---

## Quick Start

> **Already installed the skills? Run `/ai-os-helper` and ask** - "what should I add?", "explain X", "set me up".
> The concierge knows the whole catalog, recommends with reasoning, and installs + verifies for you.
> The steps below are the manual equivalent of what the paste-prompt at the top automates.

1. **Make the tools available.** Copy `skills/*` into your project's `.claude/skills/`
   (or `~/.claude/skills/` to use them in every project). Keep the ai-os checkout reachable -
   the installers read `modules/` from it.
2. **Pick your path:**
   - **Everything / a profile** -> run `/ai-os-bootstrap` and pick `product-full` | `client-embedded` | `solo-lite`.
   - **Just some modules** -> `/os-install shared-language,memory` - it auto-pulls dependencies and self-verifies.
   - **One convention, by hand (no skills)** -> copy `modules/<name>/`, paste its **Install** block into your
     `CLAUDE.md`, copy its `templates/`.
3. **Confirm it works:** `/os-doctor` -> a GREEN/ISSUES health report (every installed module present, wired,
   deps satisfied). You never have to guess whether the install is correct.

**Full walkthrough, the deployment model, and a worked end-to-end example: [docs/USAGE.md](docs/USAGE.md).**

---

## Module registry

> All built in v0.1 except `git-dev-push` (deferred). Each links to its README (with its Install block + manifest).

### Core (every project)
| Module | What it gives you | Tier |
|---|---|---|
| [shared-language](modules/shared-language/README.md) | A sealed glossary + the coin-it mechanism (one token = a whole process) | core |
| [mutual-push](modules/mutual-push/README.md) | The proactive two-way relationship + the coin-it reflex | core |
| [handoff](modules/handoff/README.md) | The `תארוז` pack-for-a-fresh-chat protocol + HANDOFF templates | core |
| [sync](modules/sync/README.md) | The `סנכרן` "refresh the living docs" mode (a handoff minus the chat-swap) | core |
| [memory](modules/memory/README.md) | Native auto-memory discipline: one-line index, format, budget, write/recall hygiene | core |
| [state-docs](modules/state-docs/README.md) | The owner-facing + AI-facing "where are we now" docs, lean + rolling-archive | core |
| [tracker](modules/tracker/README.md) | The `T-XXXX` deferrals + findings ledger; no-finding-dissolves | core |
| [gates](modules/gates/README.md) | The standing review gates (each with its procedure + rationale) | core |
| [doc-hygiene](modules/doc-hygiene/README.md) | Semantic line breaks + no-em-dash + lean docs | core |
| [platform-notes](modules/platform-notes/README.md) | Environment/tooling gotchas + verified fixes (TLS, shell, CI) | core |
| git-dev-push *(deferred)* | Terminal summarize-first commit/push convention | core |

### Heavy (opt-in)
| Module | What it gives you | Tier |
|---|---|---|
| [source-of-truth](modules/source-of-truth/README.md) | Precedence ladder + 3 source modes + the Phase-0 score rubric (+ the `spec-developer` skill) | floor + heavy |
| [test-program](modules/test-program/README.md) | The test charter: external-oracle, genre taxonomy, mutation tooling, prod-safety | lite + full |
| [agent-fleet](modules/agent-fleet/README.md) | The role set + gated workflow + per-project specialist generation | lite + full |

---

## The skills (runnable)

Invoke each as `/<name>` once it is in your `.claude/skills/`.

| Skill | What it does |
|---|---|
| [`/ai-os-helper`](skills/ai-os-helper/SKILL.md) | **The concierge - start here.** Knows the whole catalog; explains at any level, recommends with reasoning, installs + verifies on request, routes to health. |
| [`/ai-os-bootstrap`](skills/bootstrap/SKILL.md) | Install a whole profile end-to-end: pick mode + profile, evaluate the source, install modules, generate the agent fleet, assemble CLAUDE.md + settings, seed the knowledge docs, record + self-verify, commit. |
| [`/os-install`](skills/os-install/SKILL.md) | Install individual modules a-la-carte; auto-resolves dependencies, records `ai-os.lock`, self-verifies. |
| [`/os-doctor`](skills/os-doctor/SKILL.md) | Read-only health check: confirms every installed module is present, wired, and its deps satisfied. GREEN/ISSUES + fixes. |
| [`/spec-developer`](skills/spec-developer/SKILL.md) | Mature a rough spec to "derivable" across stateful rounds (used by source-of-truth / bootstrap when a spec is weak). |

---

## Read more

- **[CATALOG.md](CATALOG.md)** - the single readable map of every module, profile, and skill (what `/ai-os-helper` reads).
- **[docs/USAGE.md](docs/USAGE.md)** - the full how-to: deployment model, the three paths, module anatomy, a worked example, the verify story.

## Versioning

One version for the repo (`v0.1`, `v0.2`, ...). Each module notes "since vX" and carries a `version`
in its manifest that bumps when its content changes.
A consuming project records what it pulled - modules + their versions - in `ai-os.lock`.

## Provenance

Distilled from lessons captured while building real projects, then deepened from those projects'
proven artifacts and independently reviewed. The goal: institutionalize on day one what was learned
the hard way over many runs.
