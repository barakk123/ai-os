# Scenario List — <domain>

> Spec-derived. Every rule / edge / limit / toggle / gate / state-transition / error-code → a ROW,
> BEFORE writing tests. Get the LIST reviewed for breadth FIRST; only then write tests. This is the
> lite-tier `scenario-list-first` discipline (`convention.md` §1.2) and the seed of the full-tier
> master matrix.
>
> **Oracle rule:** every expected value here is derived from the spec FIRST (cite the section). Read the
> code ONLY to set `status` or flag a divergence — NEVER to derive what the answer should be. A snapshot
> off current output is failure-mode-B.

## How to build this list (the per-domain run process)

1. **Read the spec AND the code** for the domain — the spec to derive the oracle, the code to set status
   and spot divergences.
2. **First pass:** walk the spec section by section; every rule / edge / limit / toggle / gate / state
   transition / error code → one row, with its spec citation and its derived oracle.
3. **SECOND completeness pass (mandatory, reported separately):** re-derive from scratch, deliberately
   hunting what the first pass missed (boundary cases, no-op short-circuits, empty/null inputs, illegal
   transitions, role × tenant positions, cross-domain edges). Record what it added ("2nd pass added
   S-NN..S-MM"). It routinely finds real gaps.
4. **Apply the negative-path rule:** every must-fail path is its own row, asserting the SPECIFIC failure
   (status + machine code / SQLSTATE) — never "an error occurred".
5. **Classify each cross-domain edge** for the interaction matrix (a/b/c scope taxonomy).
6. After review, write the tests. Close cheap gaps in-run; log the rest as tracked findings
   (no-finding-dissolves). NEVER guess the test count — run the suite for the real number.

## Scenario rows

| id | scenario (from the spec) | spec ref | genre(s) | oracle (expected — spec-derived) | status | finding |
|---|---|---|---|---|---|---|
| S-01 | <the rule / edge / limit> | §X.Y | functional / golden | <explicit expected value(s)> | absent | |
| S-02 | <a must-fail path> | §X.Z | negative [*] | reject: <status> + code `<machine_code>` | absent | |
| S-03 | <illegal state transition> | §… | state-transition [*] | reject: `<code>` (transition forbidden) | absent | |
| S-04 | <boundary / no-op / empty input> | §… | boundary | <expected> | absent | |
| S-05 | <cross-domain edge> | §… | interaction | <expected> + scope-class (a/b/c) | absent | |

**status:** `absent` · `exists-correct` · `exists-weak` (asserted, not to bar) · `exists-wrong-expectation`
(failure-mode-B: green but asserts the wrong thing) · `spec-ambiguity` (needs an owner decision — do NOT
invent an oracle, mark it and surface it).

**finding:** the tracker id for any divergence / gap / weakness this row surfaces (no-finding-dissolves —
the `T-XXXX` id scheme and the no-finding-dissolves rule are owned by the `tracker` module). When the code
diverges from the spec, keep ONE row asserting the SPEC value (it will fail against current code — that
failure IS the surfaced finding) AND ONE clearly-labelled "per-code / known divergence" pin.

## Second-pass log
> What the mandatory second completeness pass added (kept visible as evidence the pass ran):
> - 2nd pass added: …

## Carve-outs ledger (deferred-with-tracking)
> Rows deliberately deferred (not "done" — tracked). A domain is NEVER "100% / done" while a carve-out is open.
> - <item> → tracked as `T-XXXX`, reason: <why deferred>, unblocks-when: <condition>.
