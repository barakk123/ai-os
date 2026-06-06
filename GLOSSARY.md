# Glossary - the shared language

The durable shorthand where one token carries a whole process.
This file is two things at once:
the working vocabulary for collaborating **inside** `ai-os`,
and the **seed** that the `shared-language` module ships to a new project.

> Rule: a token enters here only once it is agreed.
> The `mutual-push` reflex proposes candidates; the owner seals them.
> Never silently delete a token - annotate "superseded".

---

## Operational triggers (sealed v0.1)

| Token | Fires | Meaning |
|---|---|---|
| **תארוז** / "בוא נתכונן לצ'אט חדש" / "בוא נעשה handoff" / "החלון מלא" | user, or proactively at a closed+verified+committed point (auto when the window is nearly full) | Run the full **Handoff Protocol**: pre-commit alignment audit -> update state-docs + HANDOFF + tracker + memory -> compress/index memory if grown -> write a warm-start brief and/or a copy-paste prompt (recommend which) -> proper commit -> notify it's ready. |
| **סנכרן** | user, or when state has drifted from reality | The `sync` / update mode: refresh state-docs + memory + tracker **without** packing or switching chats (a handoff minus the chat-swap). |
| **core** / "הליבה" | user | "Install only the Universal Core modules in this project" (no heavy machinery). |

---

## Mechanisms

| Term | Meaning |
|---|---|
| **coin-it reflex** | When a pattern/instruction looks likely to recur, the assistant pauses and proposes `token = expansion`; once agreed it enters this glossary. Part of the `mutual-push` module. |
| **mutual-push** | The proactive two-way relationship: the assistant pushes the owner (surfaces improvements, coins shorthand, flags drift) and explicitly invites the owner to push back. "דחוף אותי לדחוף אותך לדחוף אותי." |
| **profile** | A named bundle of modules (`product-full` / `client-embedded` / `solo-lite`). |
| **module** | The atomic, individually-extractable unit of a convention. |
| **a-la-carte** | Taking individual modules rather than a whole profile. |

---

## Seed terms (universal, distilled from prior projects)

| Term | Meaning (one sentence) |
|---|---|
| **the spec is the oracle, never the code** | Expected behavior is derived from the source-of-truth, never snapshotted off current output. |
| **external oracle / reviewer != author** | Independent re-derivation from the spec (a fresh chat or a spec-only agent); the core reliability mechanism. |
| **anti-scope framing** | Seeds/handoffs are a PARTIAL warm-start, NOT the scope - re-derive the full list from the source as if the seeds did not exist, then cross-check. |
| **T-XXXX** | A tracker entry - every deferral AND every finding gets a durable id; nothing lives only in chat. |
| **no-finding-dissolves** | Every finding -> tracker + a register + (for tests) a per-row ledger; "done" is never a count. |
| **the standing gates** | Pre-commit alignment audit · implementation/plan-completeness audit · consult-before-edit · don't-declare-working-before-confirmed · negative-testing. |
| **lean + rolling archive** | State docs stay lean; the full history is snapshotted to a dated archive when a doc exceeds its size budget. |
| **one-line index** | The always-loaded memory index is one line per memory; detail lives in topic files loaded on demand. |
