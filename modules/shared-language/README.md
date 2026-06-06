# shared-language

> A durable shared vocabulary where one token carries a whole process -
> plus the format for tokens and the rule for how they get sealed.

**Status:** since v0.1
**Dependencies:** `standalone`.
Soft-deps: `mutual-push` (its coin-it reflex proposes new tokens);
`tracker` (optional, if you log proposals before sealing).

<!-- ai-os:manifest
version: 0.1.0        # bumps (semver) when this module's content changes; absence is fine (older installs have none)
tier: core            # core | heavy
deps: { hard: [], soft: [mutual-push, tracker] }
tiers: [single]       # [single]  OR  [lite, full]  OR named e.g. [authored-spec, external, hybrid]
-->

---

## What it gives you

- A `GLOSSARY.md` in the project: the canonical home of every sealed token.
- A token format and three categories (operational triggers / mechanisms / domain terms).
- A sealing rule: tokens are proposed (coin-it) and sealed by the owner; never silently deleted.
- A one-line `CLAUDE.md` pointer so the vocabulary is live from message one.

## Install

Paste into the project's `CLAUDE.md`:

> ### Shared Language
> The project keeps a sealed shorthand in `GLOSSARY.md` - one token carries a whole process.
> Use sealed tokens exactly as written. When something looks likely to recur, propose
> `token = expansion` (the coin-it reflex); it enters `GLOSSARY.md` only once the owner seals it.
> Never silently delete a token - annotate "superseded".

Then copy `templates/GLOSSARY.template.md` to the project root as `GLOSSARY.md` and seed it.

## Generates in target

- `GLOSSARY.md` at the project root (from the template).
- A "Shared Language" section in `CLAUDE.md` (the install block above).

## Files it scaffolds

- `templates/GLOSSARY.template.md` - a starter glossary
  (universal seed terms + empty project-token sections).

## Why

A shared, durable shorthand is a force-multiplier across long projects and chat-swaps:
one agreed token replaces a paragraph of context and survives a fresh chat intact.
Coining it deliberately - and sealing it by the owner - keeps the vocabulary precise
instead of letting it drift.
