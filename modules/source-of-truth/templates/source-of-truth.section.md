## Source Of Truth

Truth comes from: `<authored-spec | external-monorepo | hybrid-greenfield>`.

Precedence (highest first):

1. `<master spec  |  the upstream monorepo + its docs + docs/house-rules.md>`
2. `<derived per-domain docs in docs/  |  local conventions>`
3. `<architecture / AI-architecture doc>`
4. `<skills + CLAUDE.md files>`
5. `<TRACKER (docs/TRACKER.md)>`
6. `<state files (owner-facing status + AI-facing current-state)>`

Rules:
- If two sources disagree, **STOP** and surface the contradiction; the **highest-precedence source wins**.
  Do not guess and do not pick the more convenient one - a lower rung contradicting a higher rung is a bug
  in the lower doc (or a real spec gap), surfaced, never silently reconciled.
- **Lower layers explain HOW to work, not WHAT is true.** Never treat a summary or a state file as
  stronger truth than the source it derives from.
- Derived docs add technical precision but **never new business truth**. If a needed rule is missing from
  the master, fix the master (and log a `T-XXXX`); never invent it in a derived doc.

### Hot spots (the no-invented-logic footgun list)

<Fill with this project's counter-intuitive semantics: deprecated/removed fields, fields whose meaning is
not what the name suggests, "X is cancel+create-new, there is no rescheduled status" style traps. These
are the specific places a downstream role is most likely to invent the wrong rule.>

- `<field/concept>` - `<the counter-intuitive truth>`
