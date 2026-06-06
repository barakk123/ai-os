# sync - convention

## Trigger
Owner: the project's sealed **sync trigger** - whatever short control token the owner has sealed for
"realign the docs to reality" (sealed control tokens are the owner's to set per project; phrase the
trigger in the project's chosen language, and keep this English template - never hardcode one locale's
word as the only form). ai-os dogfoods its own sealed instance, `סנכרן`, and uses it throughout this
module's examples; substitute your project's sealed token.

Proactive: when the assistant notices the docs no longer match reality.

## What it does
1. Re-read the prompts / work since the last sync or commit.
2. Refresh the living docs to match what is actually true now:
   - state-docs (where we are), memory (durable facts), tracker (deferrals / findings).
3. Report what changed.

## What it does NOT do (the boundary vs handoff)

The two modes are fired by two distinct sealed tokens - the project's **sync trigger** and its
**pack / handoff trigger**. ai-os's own sealed instances are `סנכרן` (sync) and `תארוז` (pack);
the columns below name those examples, but the boundary is by role, not by any one locale's word.

| | sync (the sync trigger; ai-os: `סנכרן`) | handoff (the pack trigger; ai-os: `תארוז`) |
|---|---|---|
| refresh state-docs / memory / tracker | yes | yes |
| pre-commit alignment audit | light | full |
| warm-start brief / copy-paste prompt | no | yes |
| pack commit + notify | no | yes |
| intended for a chat-swap | no | yes |

Fire the sync trigger often (it is cheap); fire the pack trigger at a real stopping point or a
chat-swap.
