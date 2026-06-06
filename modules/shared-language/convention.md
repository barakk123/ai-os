# shared-language - convention

## What a token is
A short trigger (one or two words, any language) that maps to a whole process or concept,
so it can be invoked or referenced without re-explaining it each time.

## Format
`GLOSSARY.md` holds tokens in tables, grouped by category:

- **Operational triggers** - a token the owner *fires* to run a process.
  Columns: `Token | Fires (who / when) | Meaning (the process)`.
- **Mechanisms** - named behaviors or relationships.
  Columns: `Term | Meaning`.
- **Domain terms** - project-specific vocabulary.
  Columns: `Term | Meaning`.

## Sealing (how a token enters or leaves)
- A token is **proposed** by the coin-it reflex (the `mutual-push` module),
  when a pattern looks likely to recur.
- It is **sealed** by the owner. Only sealed tokens go in `GLOSSARY.md`.
- A token is **never silently deleted**. When it stops applying, annotate it "superseded"
  (keep the row; note what replaced it).

## Placement (DRY)
Canonical glossary = `GLOSSARY.md` (always-loaded, project-scoped).
Memory holds a single pointer to it, not a copy (so there are never two drifting glossaries).

## Relationship to other modules
- `mutual-push` produces token proposals (coin-it) and feeds them here.
- `handoff` / `sync` rely on sealed tokens as their own triggers (e.g. `תארוז`, `סנכרן`).
- `tracker` (optional) can log a proposal until it is sealed.
