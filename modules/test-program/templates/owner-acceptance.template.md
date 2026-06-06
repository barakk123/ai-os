# Owner-Acceptance Checklist — <domain / feature>

> The owner-legible acceptance layer. This is the artifact that closes **oracle 2 — truth-docs ↔ owner
> intent** (`convention.md` §0). Every other test artifact closes oracle 1 (tests ↔ truth-docs): a test
> can be perfectly faithful to a spec that quietly DRIFTED from what the owner actually wanted, and inherit
> the drift. Only the human owner can close oracle 2, and only if the assertions are written in language
> they can read and SIGN — not in code or spec-section shorthand. Full-tier (genre H: acceptance / human).
>
> **One row per owner-visible decision.** Plain language ("as an owner, when X happens, then Y"), the spec
> citation it traces to, and a sign-off the owner physically marks. A row the owner reads and says "no, I
> wanted Z" is a spec-drift finding (log it → `tracker`), NOT a test bug. This layer is where that surfaces.

## How to use this checklist

1. Derive one row per OWNER-VISIBLE behavior from the same spec sections the master matrix covers — but
   phrase each as the OWNER experiences it, not as the engine computes it. (Internal-only invariants that an
   owner can never observe do not belong here; they live in the master matrix.)
2. Hand the rendered checklist to the owner. The owner reads each `Then` and marks **accept** / **reject** /
   **unsure**. Capture their words verbatim in the notes column — that phrasing is itself oracle-2 signal.
3. A **reject** or a corrected expectation = a truth-doc drift finding: the spec (or the owner's intent
   capture) is what is wrong, not necessarily the code. Log it as a `T-XXXX` (no-finding-dissolves, owned by
   the `tracker` module) and route the spec correction before re-deriving the affected matrix rows.
4. An **unsure** = a `spec-ambiguity`: do NOT invent the answer — surface it for an explicit owner decision.
5. Re-confirm the affected rows whenever the underlying spec section changes (a signed row goes stale when
   its source drifts). The signature is dated so staleness is visible.

## Acceptance rows (structured checklist)

> Owner-legible. Each row is a single observable behavior, traced to its spec source, with the owner's mark.

| id | as an owner, when… (Given/When) | …then I expect (Then — plain language) | spec ref | owner mark (accept / reject / unsure) | notes (owner's words) |
|---|---|---|---|---|---|
| OA-01 | <the trigger an owner can see> | <the outcome in the owner's terms> | §X.Y | ☐ accept ☐ reject ☐ unsure | |
| OA-02 | <a setting the owner toggles> | <how behavior visibly changes> | §X.Z | ☐ accept ☐ reject ☐ unsure | |
| OA-03 | <a must-fail action the owner might attempt> | <the refusal the owner sees, in plain words> | §… | ☐ accept ☐ reject ☐ unsure | |

## Gherkin form (optional alternate rendering)

Some owners read scenarios more naturally than tables. The SAME rows, rendered as Given/When/Then. Pick the
table OR this form per audience — do not maintain both as separate sources of truth (one is the source, the
other a rendering of it).

```gherkin
# OA-01  (spec: §X.Y)
Scenario: <plain-language title of the owner-visible behavior>
  Given <the precondition the owner can observe / set>
  When  <the action the owner takes or the event that occurs>
  Then  <the outcome the owner expects to see>
  # owner mark: ____________   date: __________   notes: ____________
```

## Sign-off

> The owner signs the SET once every row is marked. A signed checklist is the evidence oracle 2 was closed
> for this domain at this spec version. It goes stale when the cited sections change — re-confirm then.

- Domain / feature: <name>
- Spec version reviewed: <vX.Y / commit>
- Rows accepted: __ / __  ·  rejected (→ findings): __  ·  unsure (→ ambiguities): __
- Owner sign-off: ____________________   date: __________
- Findings raised this pass (T-XXXX): <ids — every reject/unsure has one>
