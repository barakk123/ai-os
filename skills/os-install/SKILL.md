---
name: os-install
description: Install one or more ai-os modules a-la-carte (not a whole profile), dependency-aware and self-verifying. Resolves each module's manifest dependencies, copies templates, appends CLAUDE.md install blocks idempotently, records the install in ai-os.lock, and runs os-doctor. Use with /os-install <module>[,<module>...].
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# os-install

Install one or more ai-os modules a-la-carte - vs `ai-os-bootstrap`, which installs a whole profile.
**Dependency-aware, idempotent, and self-verifying:** whatever you ask for, it pulls in what that needs
and proves the result is healthy, so you are never left guessing whether the install is correct.

## Usage
```
/os-install shared-language
/os-install shared-language,memory,handoff
```

## Prerequisite
The ai-os `modules/` source must be reachable (the skill reads each module's README + manifest +
templates). Use the ai-os checkout you ran this from, or the `source:` path in an existing `ai-os.lock`;
if you cannot find it, ask the owner for the ai-os checkout path.

## Steps

1. **Resolve the dependency closure.** For each requested module, read its `<!-- ai-os:manifest -->`
   block in `modules/<name>/README.md`. Add every `hard` dep to the install set (transitively) - a hard
   dep is non-optional, so pull it in automatically and tell the owner you did. For each `soft` dep not
   already installed and not in the set, note it as an optional add (the module works degraded without
   it) - do not force it.
2. **Order the set** so dependencies install before dependents (topological on the manifest deps).
3. **Per module, install** (idempotent - skip with a one-line "already present" if it is):
   - Copy its `templates/*` into the project at the path each implies (per `Generates in target`);
     do not overwrite an existing file without confirming.
   - Append its **Install** block (the README blockquote) to the project `CLAUDE.md`
     (skip if its heading already exists).
   - Seed any knowledge doc it needs (e.g. `GLOSSARY.md` for `shared-language`) from its template.
4. **Record it.** Create or update `ai-os.lock` at the project root - the ai-os version, the `source:`
   checkout path, and the installed-module list (+ tier). Append the newly-installed modules.
5. **Self-verify.** Run `/os-doctor` and carry its GREEN/ISSUES result into the report; fix any miss now.

## Report
```
ai-os install: <GREEN | ISSUES>

Requested:          <list>
Auto-added (deps):  <hard deps pulled in>
Soft-deps absent:   <list> -> optional: /os-install <dep>
Files created:      <list>
CLAUDE.md blocks:   <added | already present, per module>
Health (os-doctor): <GREEN - N modules wired | the issues + fixes>
```

## Note
Idempotent: re-running a module that is already installed makes no change (and says so). Safe to re-run
to repair a partial install - it fills only what is missing, then re-verifies.
