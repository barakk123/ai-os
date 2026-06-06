# Glossary - <project> shared language

The durable shorthand where one token carries a whole process.
Canonical home of every sealed token: proposed via coin-it, sealed by the owner,
never silently deleted (annotate "superseded").

---

## Operational triggers

| Token | Fires | Meaning |
|---|---|---|
| _(add project triggers - e.g. a pack/handoff trigger, a sync trigger)_ | | |

## Mechanisms

| Term | Meaning |
|---|---|
| **coin-it reflex** | When a pattern looks likely to recur, propose `token = expansion`; sealed by the owner. |
| **mutual-push** | The proactive two-way relationship: the assistant pushes the owner + invites pushback. |

## Domain terms

| Term | Meaning |
|---|---|
| _(project-specific vocabulary)_ | |

---

## Universal seed terms (carry into any project)

| Term | Meaning |
|---|---|
| **the spec is the oracle, never the code** | Expected behavior is derived from the source-of-truth, never snapshotted off current output. |
| **external oracle / reviewer != author** | Independent re-derivation from the source (a fresh chat or a spec-only agent). |
| **anti-scope framing** | Seeds/handoffs are a PARTIAL warm-start, NOT the scope - re-derive the full list as if they didn't exist, then cross-check. |
| **T-XXXX** | A tracker entry - every deferral AND finding gets a durable id; nothing lives only in chat. |
| **no-finding-dissolves** | Every finding -> tracker + a register + a ledger; "done" is never a count. |
| **derivable** | The bar a spec must reach: ~90%+ of downstream decisions derivable from it alone (owned by source-of-truth / spec-developer). |
| **Output Contract** | The structured hand-off a role returns; one role's Output Contract is the next role's input (owned by agent-fleet). |
| **standing gate** | An always-on review discipline that fires on meaningful work (owned by gates). |
| **regression-coverage gate** | When work touches an already-covered domain: re-run its suites + back-fill the new interaction edges (seeded when test-program installs). |
| **per-stage independent-review gate** | At every stage/domain boundary: ready the infra + hand off to a fresh-chat external-oracle review (seeded when test-program installs). |
| **the two completeness gates** | (1) a 2nd completeness pass before continuing + (2) a backward retro sweep (owned by handoff). |
| **Required User Action** | A step the work cannot complete without the owner doing it (deploy / config / secret / verify); stated explicitly. |
