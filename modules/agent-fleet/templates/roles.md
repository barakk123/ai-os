# Roles - <project>

One-screen index of the fleet. Drop into the AI-architecture doc's "Role Set" section. The reliability
rule under everything: **the reviewer is never the author.**

## Generic (shipped as-is from this module's templates)
- **orchestrator** - route + classify meaningful work, select roles, plan the gates, approve / return.
- **spec-guardian** - the intent oracle: spec alignment, cross-doc consistency, invented-logic risk.
- **code-reviewer** - the correctness oracle: bugs, regressions, edge cases, weak/missing verification.
- **current-state-specialist** - the reality recorder: update the state docs after approved work.
- **git-dev-push** - terminal summarize-first commit/push (its own `git-dev-push` module, deferred;
  until it ships, the commit step is a plain commit).

## Project (full tier; generated from the `*.format.md` templates)
- **`<layer>-implementer`** (one per tech layer) - write `<layer>` code; no invented logic; consume the
  specialist's interpretation. Paired peers (web/mobile) carry the parity block.
- **`<domain>-specialist`** (one per significant domain) - interpret `<domain>` rules; no implementation;
  carry the "Surface effects on:" radar + the ambiguities-to-escalate field.
- **`<platform>-platform-specialist`** (one per managed platform) - guide manual platform actions via the
  six-field Guidance Format; verify; no code.

## The loop (full tier)
orchestrator -> specialist -> implementer -> spec-guardian -> code-reviewer -> current-state-specialist
-> orchestrator approves. Standing gates (the `gates` module) wrap it at commit / milestone / stage
boundaries. Do not bypass the gates on meaningful work.

## Generation
Generics: copy as-is. Implementers/specialists/platform-specialists: fill `implementer.format.md`,
`specialist.format.md`, `platform-specialist.format.md`. Generate exactly what the analysis warrants -
a domain earns a specialist only with its OWN entities + workflows + edges; no managed platform -> no
platform specialist.
