---
name: os-doctor
description: Health-check an ai-os install in the current project - verify every installed module's CLAUDE.md install block, templates, and dependencies are present and wired, then report a GREEN/ISSUES summary with concrete fixes. Read-only. Use after /ai-os-bootstrap or /os-install, or anytime, with /os-doctor.
allowed-tools: Read, Glob, Grep, Bash
---

# os-doctor

Health-check an ai-os install in THIS project, and tell the owner plainly whether it is intact.
**Read-only** - it diagnoses, it never changes files. Run it after `/ai-os-bootstrap` or `/os-install`,
or anytime you want positive confirmation that what you pulled actually works.

This is the confidence mechanism: the owner should never be left guessing whether an install is
correct. A green report is an explicit signal, not silence.

---

## What it checks

1. **The install record.** Read `ai-os.lock` at the project root (written by bootstrap / os-install):
   the ai-os version, the source checkout path, and the list of installed modules (+ tier). If it is
   absent, fall back to detecting installed modules from the `### <Module>` install-block headings in
   `CLAUDE.md`.
2. **Per installed module, three checks** (read `modules/<name>/README.md` from the source checkout):
   - **Install block present** - the module's section exists in the project `CLAUDE.md`.
   - **Templates present** - every file under the module's `Files it scaffolds` exists at its target
     path (per `Generates in target`).
   - **Dependencies satisfied** - every `hard` dep in the module's `<!-- ai-os:manifest -->` is also
     installed; every `soft` dep is installed OR recorded as intentionally absent.
3. **Cross-reference integrity** - no installed module's text points at a module that is not installed
   (e.g. a role that references `test-program` when the test charter was not pulled).

---

## How to run it

1. **Locate the ai-os source** (`modules/`): use `source:` from `ai-os.lock`; if missing, ask the owner
   for the ai-os checkout path. (You need the module READMEs to know each module's manifest + files.)
2. Read `ai-os.lock` (or detect from `CLAUDE.md`). Build the installed-module list (+ tiers).
3. For each module, run the three checks. Use `Read`/`Glob` for file presence; read the manifest block
   for deps; read `CLAUDE.md` for the install-block headings.
4. Aggregate into the report.

---

## Report (always print this, GREEN or not)

```
ai-os health: <GREEN | ISSUES>    (ai-os <version> · profile <x> · <N> modules)

  <module>          install ✓   templates ✓   deps ✓
  <module>          install ✓   templates ✗   deps ✓     -> missing file: <path>
  <module>          install ✓   templates ✓   deps ✗     -> missing dep: <dep>

Summary: <N> healthy, <M> with issues.
Fixes:
  - <concrete command per issue, e.g. "/os-install <dep>" or "copy templates/X to <path>">
  (or: "Nothing to fix - the install is healthy.")
```

- If **GREEN**, state it plainly: "All <N> installed modules are present, wired, and their dependencies
  satisfied." Positive confirmation is the point.
- If **ISSUES**, every issue gets a concrete one-line fix (a command or a file to copy) - never a vague
  "something is off".

---

## Why

An install the owner cannot trust is an install the owner will not use. `os-doctor` turns "I pulled
some files, I think it's set up?" into a definite answer, and makes every a-la-carte or profile install
self-checking. It is also the regression guard: re-run it any time to confirm a manual edit did not
break a cross-reference or drop a dependency.
