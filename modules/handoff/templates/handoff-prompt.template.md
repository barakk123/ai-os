<!--
  handoff-prompt.template.md - the COPY-PASTE OPENING MESSAGE for the next chat.

  This is the compressed, paste-as-message-#1 variant of the handoff (see HANDOFF.template.md
  for the full document form). Fill the <...> placeholders, then the owner pastes the result as
  the FIRST message of the fresh chat. Two modes below: CONTINUATION and INDEPENDENT-REVIEW -
  keep one, delete the other.

  Recommended pattern: write the full HANDOFF.md brief AND a short prompt (Mode C) that POINTS at it,
  so the owner has a one-paste hand-off while the depth stays in the file.

  This prompt is throwaway: paste, work, discard. Do NOT commit it.
  Obey doc-hygiene if you save it: semantic line breaks, no block-trigger as a wrapped line's first char.
-->

================================================================================
MODE A - CONTINUATION (paste this to resume a task in a fresh chat)
================================================================================

You are continuing `<task>` on `<project>`. Start from a clean slate - everything you need is below,
but treat it as a PARTIAL warm-start, NOT the scope.

ANTI-SCOPE (read first): the seeds / file-map / "what's done" / next-steps below are what the previous
chat happened to notice - NOT the full work list. Re-derive the remaining work FROM SCRATCH from the
source-of-truth (listed below) as if this message did not exist; THEN cross-check against the seeds.
The oracle is the spec, never the code. You WILL find things not listed here, and you should. Tripwire:
if your work-list ends up looking like a copy of the seeds below, you under-investigated.

DO BOTH COMPLETENESS GATES, visibly, and report each result:
- Gate I (second completeness pass, before building): re-derive from a different angle - walk every
  sentence of `<spec section>` + every setting - and confirm each maps to a work item.
- Gate II (backward retro sweep, after): re-examine the already-done areas for edges this work creates.

WHERE WE ARE: <one or two sentences: the exact point reached, and what is NOT done yet>.

CHECKPOINT, NOT DONE: the per-batch commits here are CHECKPOINTS - the review gate has NOT fired. Do NOT
declare `<domain>` done or scale to other domains until the full-closure path + a fresh-chat independent
review are complete. "Done" is never a green run or a built-count.

RECONCILE AT START (stale living artifacts - regenerate, do NOT rebuild done rows): <ledger/matrix> is stale
(<why>) - reconcile against `<source>` + flip the boxes for what exists; <test index> is at `<stale count>` -
regenerate from `<generator>`. Cross-ref before building so you don't redo already-done rows.

PARTIALLY DONE (don't assume a clean tree): committed + pushed + CI-green at HEAD `<hashX>`: <first half>.
Built-but-UNCOMMITTED in the working tree (owner commits when ready): M `<file a>`, `<file b>`; NEW
`<file c>` - `<tests-only / no engine change>`, verified green, not yet committed. Do NOT rebuild it.

SOURCE OF TRUTH (re-derive from these, in precedence order): <master spec + the governing sections> ·
<derived domain doc> · <schema / api / error-code sections> · <the code, ONLY to check status>.

NEXT STEP(S): <the immediate next action(s), ordered; what is blocked vs buildable now>.

SEALED OWNER DECISIONS (do NOT re-ask): <OD-n: the ruling> · <...>.
OWNER-APPROVED DEFERRALS (reason-bound, settled-later, do NOT re-ask or re-flag as missing - distinct from
open findings): <item -> target step (reason)> · <...>.
OPEN FINDINGS (none may dissolve, still need work): <T-xxxx: one line> · <...>.

READ FIRST (in order): <memory index + relevant memories> · <CLAUDE.md + path-scoped CLAUDE.md> ·
`HANDOFF.md` (the full brief) · <the spec sections above>.

VERIFY THE BASELINE YOURSELF before building (don't trust this message):
`<test command>` -> expect `<result>`; `<typecheck>` -> clean; `<CI check>` -> green on `<HEAD>`;
re-derive any count yourself.

STANDING GATES: pre-commit alignment audit · implementation/plan-completeness audit · regression-coverage
(where the harness exists) · don't-declare-working-before-confirmed · negative-testing (assert the
specific failure code) · consult-before-edit · No-Invented-Business-Logic. Commit/push only when asked;
any gated change needs its named approver(s).

Confirm the baseline, present your re-derived delta for approval, then proceed.


================================================================================
MODE B - INDEPENDENT REVIEW (paste this to run a fresh-chat adversarial review)
================================================================================

Your role in this chat: an honest, adversarial, INDEPENDENT review of `<artifact>` on `<project>`.
Do NOT rubber-stamp. This is READ-ONLY - report findings to a new doc `<review-doc-path>`; do NOT edit
the artifact or write code (corrections happen back in the origin chat). Reviewer != author by design:
you are the external oracle that catches what a green check cannot.

CARDINAL PRINCIPLE - do NOT trust the artifact OR this message. Every count / "what's done" / oracle
value is a claim by the chat that produced it. Re-derive expected behavior FROM THE SOURCE-OF-TRUTH,
never from the code or from this message. Read the code only to check status, never to decide what
SHOULD happen. (Green proves "the code does X", not "X is the intended Y".)

ANTI-SCOPE: the artifact and any list below are a PARTIAL warm-start, NOT your scope. Derive the COMPLETE
expected behavior for `<artifact>` FROM SCRATCH from the spec, as if the artifact did not exist; THEN
cross-check the two derivations to catch what the artifact MISSED. Tripwire: if your findings look like a
restatement of the artifact, you under-investigated.

DO BOTH COMPLETENESS GATES, visibly, and report each result:
- Gate I (second completeness pass): re-derive from a different angle - walk every sentence of
  `<spec section>` + every setting - and confirm each maps to >=1 item in the artifact.
- Gate II (backward retro sweep): re-examine the cross-domain edges for interactions the artifact omits.

REVIEW + CATEGORIZE findings as: missing-item / wrong-oracle / wrong-status / missing-genre / structural /
invented / spec-ambiguity. Each finding: severity (blocker/high/med/low) + evidence (the ref +
what-the-spec-says) + a recommended correction. Consider parallel independent sub-reviews from different
angles (completeness vs oracle-correctness vs genre-coverage).

SOURCE OF TRUTH (re-derive from these, in precedence order): <master spec + the governing sections> ·
<derived domain doc> · <schema / api / error-code sections>.

CONTEXT (the WHY): <the program-context doc / the ratified bars / the taxonomy doc>.

VERIFY THE BASELINE YOURSELF: `<test command>` -> expect `<result>`; `<typecheck>` -> clean;
`<CI check>` -> green on `<HEAD>` (no code changed since, only docs).

YOUR DELIVERABLE: `<review-doc-path>` with the categorized findings, then a VERDICT - is `<artifact>` a
sound, complete, oracle-correct foundation, or what must be fixed first? Findings return to the origin
chat for remediation (you do not fix them here).


================================================================================
MODE C - POINTER PROMPT (recommended: paste this; it sends them to the full HANDOFF.md)
================================================================================

You are <continuing `<task>` | independently reviewing `<artifact>`> on `<project>`. Open `HANDOFF.md` at
the repo root and follow it - but treat it as a PARTIAL warm-start, NOT the scope: re-derive the work
from the source-of-truth it points to, as if the brief did not exist, then cross-check; the oracle is the
spec, never the code. Run BOTH completeness gates visibly (second pass + backward retro sweep) and report
each. Verify the baseline yourself (re-run the suite / tsc / CI per the brief) before acting. Present your
re-derived delta for approval, then proceed. [REVIEW only: this is READ-ONLY - report findings to a new
doc + a verdict; do not edit or fix; reviewer != author.]
