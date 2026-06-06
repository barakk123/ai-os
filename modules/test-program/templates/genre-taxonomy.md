# Test Genre Taxonomy — the complete universe

> The universe of test GENRES (kinds of checks) to CONSIDER per domain — NOT a list to run wholesale.
> Per domain you DISPOSITION every genre in the genre-applicability ledger (covered / partial /
> scoped-out-with-a-written-reason — silence is a gap). Categories J (client) and K (store) ship
> pre-registered but DORMANT until a frontend / mobile / release build exists.
>
> Legend: **[*]** essential (default these to priority=essential unless a written reason downgrades) ·
> **[BE]** backend · **[CL]** client (web/mobile) · **[ST]** store/release · **[CI]** infra/meta/CI.
> The ~149-genre breadth was rated "beyond-world-class" by adversarial critic panels on the source
> project — it is intentionally broader than most SaaS shops run; the point is to make the FULL space
> visible so nothing is missed by silence. Generalize the illustrative examples to your domain.

The lite tier (`convention.md` §1) covers roughly genres 1–5, 10, 16, 25, 36–38, 75–76, 82 — the
functional/negative/contract/RLS/property/mutation core. The full tier dispositions ALL of the below.

---

## A. Functional / behavioral [BE]
1. [*] **Unit (pure logic)** — engine internals, time/identity/pagination/RBAC/error-builder helpers,
   standalone schema predicates. Catches: wrong arithmetic, off-by-one, branch logic.
2. [*] **Integration (service × real local DB)** — triggers, stored procedures, exclusion/partial-unique
   constraints, CHECK violations, FK cascade/restrict, atomic multi-write + rollback, security-definer fns.
   Catches: the gap between "the query I wrote" and "what the DB actually does".
3. [*] **e2e / API end-to-end (in-process inject, full path)** — lifecycle, multi-step + cross-domain
   journeys, public unauth paths. Catches: routing / preHandler / serialization bugs the service layer hides.
4. [*] **Contract / schema (one-source-many-consumers)** — accept/reject, strict unknown-key rejection,
   enum exactness, wire-type fidelity. Catches: client/server drift, silent key-stripping.
5. [*] **Negative / error-path + status-code + machine-error-code coverage** — every code with a specific
   `{status, code}` assert. Catches: wrong-reason failures, missing rejections (the lite §1.3 rule).
6. [*] **API envelope conformance** — the standard response/error envelope shape.
7. **Pagination / sort / range taxonomy** — every pagination bucket, bounds, ceilings.
8. **Idempotency-key + idempotent-retry / double-submit dedup.**
9. **Transactional integrity / partial-failure / saga-compensation.**
10. [*] **State-transition / lifecycle exhaustiveness** — every LEGAL and ILLEGAL transition asserted.
11. **Boundary-value + equivalence-partitioning + decision-table** (multi-condition rules).
12. **Table-driven / data-driven (parameterized).**
13. **Golden** — hand-computed expected reference output. (Hand-compute from the spec; do NOT inline-snapshot
    off current output — that is failure-mode-B in genre form.)
14. **Smoke / sanity + accumulated CI-enforced regression.**
15. **Error-guessing + exploratory-as-chartered-sessions.**
16. [*] **Cross-domain interaction-matrix** — every runtime edge between domains (see interaction-matrix).
17. **Content-negotiation / HTTP-semantics.**
18. **Versioning / backward-compatibility / contract-evolution.**
19. [*] **Feature-flag / plan-gate × settings-matrix combinatorial** — the config-toggle × plan-gate space,
    downgrade-preserve, gated-feature-only.

## B. Data & DB layer [BE]
20. [*] **Migration forward-apply** (fresh, in order).
21. **Migration idempotency / re-run safety.**
22. **Migration down / rollback / reversibility** (or an explicit no-rollback policy).
23. [*] **Schema-conformance introspection** — columns/types/defaults/nullability/grants/exposure.
24. [*] **Constraint & index introspection** — CHECK/UNIQUE/EXCLUDE/FK + partial-predicate correctness.
25. [*] **RLS / row-level authz behavior** (live credentials, not a bypass) + recursion/cost tripwire +
    a dormant 2nd-layer independent verification.
26. [*] **Trigger behavior.**
27. [*] **Stored-function / security-definer logic** (+ search-path hardening).
28. **Transaction isolation & serialization-anomaly.**
29. **Referential-integrity / cascade / on-delete + orphan-cleanup-after-delete.**
30. **Data-integrity INVARIANTS over arbitrary/random states** (property-based; see 75).
31. [*] **Time/timezone storage** — timezone-aware columns + DST (incl. the transition DAY).
32. **Data-migration / backfill correctness.**
33. **NULL / empty / encoding / collation edge cases.**
34. [*] **Production-shaped migration DRY-RUN** — rehearse the bulk-apply on prod-shaped data.
35. **Query & index performance / N+1 / plan + big-tenant data-skew.**

## C. Security, privacy & compliance [BE + CL/ST where noted]
36. [*] **Auth preHandler / authn-bypass / token integrity** (tamper/expiry/replay).
37. [*] **Authorization / RBAC + authorization-matrix exhaustiveness** (every role × every endpoint ×
    tenant-position).
38. [*] **Cross-tenant isolation / IDOR** (the multi-tenant master genre).
39. [*] **Privilege escalation** (vertical: role & plan boundary).
40. [*] **Injection** (SQL/operator/header/log/template) + SSRF / outbound-webhook safety.
41. **Mass-assignment / over-posting / parameter tampering.**
42. [*] **Secret / PII / internal-detail leakage** in responses + logs.
43. [*] **Enumeration / brute-force / rate-limit** (+ assert a currently-missing limiter as a pinned gap).
44. **Security headers & transport hardening** [CL/ST].
45. [*] **Dependency / supply-chain SCA + SAST / secret-scanning + 3rd-party-SDK compliance** [CI].
46. **Anti-fraud / business-logic abuse.**
47. [*] **Data-export / portability correctness** + format-fidelity + re-import round-trip.
48. [*] **Anonymization-not-deletion + retention-window + hard-delete lifecycle** (jurisdiction-specific).
49. [*] **Audit-log completeness + immutability / tamper-evidence / non-repudiation** (append-only).
50. **PII masking & audited reveal** (support-console tooling).
51. **Consent / lawful-basis / communications opt-in.**
52. **Accessibility-as-LEGAL** (jurisdictional accessibility regs) [CL].
53. **Data residency / sub-processor / cross-border transfer.**
54. [*] **Jurisdictional privacy-law consistency** across all surfaces + privacy-policy-URL reachability.

## D. Concurrency, reliability & operational [BE + CI]
55. [*] **Race conditions** + row-lock/advisory-lock + optimistic/pessimistic locking (NAMED: each quota
    trigger, ownership transfer, reactivation, N-parallel same-resource booking).
56. **Deadlock & lock-ordering.**
57. **Eventual-consistency / read-after-write / stale-read.**
58. **Linearizability / concurrency-invariant post-storm assertions.**
59. **Retry / backoff / timeout behavior.**
60. **Fault-injection / chaos** (dependency down/slow) + graceful degradation + fail-safe defaults.
61. **Backpressure / resource-exhaustion / connection-pool.**
62. [*] **Scheduled-job / cron correctness** + reminder-window end-to-end dispatch.
63. **Backup / restore / disaster-recovery + migration-drift.**
64. **Observability correctness** — structured logging / metrics / tracing / health-readiness-liveness
    probes emit truthfully.
65. **Synthetic monitoring / active probing + alerting/alarm-logic verification.**
66. **Deployment / post-deploy smoke + rollback-in-CI.**
67. **Breach-detection / anomaly / incident-readiness.**

## E. Performance & scalability [BE + CI]
68. **Load / stress / spike / soak-endurance.**
69. **Scalability / capacity** (scales with the knob).
70. [*] **Latency-percentile SLO assertions** (p50/p95/p99 pass/fail) + throughput/saturation.
71. **Payload-size & serialization performance.**
72. **Memory / resource-leak.**
73. **Cold-start / warm-up.**
74. **Performance-regression / benchmark gating in CI.**

## F. Generative / property / formal [BE]
75. [*] **Property-based** — output invariants over generated inputs (the engine-specific instance of 30).
76. [*] **Differential** — the real implementation vs an INDEPENDENT brute-force reference oracle (the
    cleanest external oracle: two independent derivations of the same answer).
77. **Metamorphic** — relations the system MUST preserve under input transforms.
78. **Stateful / model-based** — a lifecycle as a generated command sequence vs a model.
79. **Fuzzing** — structure-aware (valid-ish requests) + dumb/byte-level at the HTTP/token boundary.
80. **Formal-ish spec checks** — state-machine reachability + constraint/SMT encoding of the rules.
81. **Shrinking / counterexample-quality discipline.**

## G. Meta / test-quality measurement [CI] — measures the strength of all the above
82. [*] **Mutation testing** — code + schema; the empirical "did we test what matters" (see mutation-tooling).
83. [*] **Coverage adequacy & gating** (line/branch/condition/path) — necessary, not sufficient.
84. [*] **Spec-anchored oracle-correctness as an EXECUTABLE genre** (automate failure-mode-B detection, not
    only review) + oracle-independence audit (writer ≠ reviewer, spec-not-code expected values).
85. [*] **Flakiness / determinism detection & quarantine.**
86. **Test isolation & order-independence verification.**
87. **Assertion-density / anti-tautology linting + test-smell detection.**
88. **Build / CI reproducibility & hermeticity.**
89. **Test-data-builder / factory correctness + seed realism / production-like-volume fixtures.**
90. [*] **Spec-to-test traceability & catalog completeness** (the master matrix is the artifact).
91. [*] **Anti-regression bug-reproduction** — every fixed bug gets a permanent failing-first test.
92. **Documentation-as-tests / runnable examples.**
93. **AI-build provenance / inter-agent-drift / hallucinated-surface tests** (essential when the codebase
    is AI-authored — failure-mode-B compounds across agents).
94. [*] **Local-stack vs hosted-prod platform-fidelity** (env/config drift).
95. **Web ↔ mobile parity** [CL].

## H. Acceptance & human-oracle [BE/CL] — closes oracle-2 (truth-docs ↔ owner intent)
96. [*] **Owner-legible acceptance / BDD** — scenarios in the OWNER's language the owner can validate.
    This is the layer only the human can sign off; it is how truth-docs↔intent gets closed.
97. **Example-mapping / specification workshops** (rules → examples → questions, pre-code).
98. **End-to-end user-journey / persona-driven scenarios.**
99. [*] **Negative-space / forbidden-behavior assertions** — prove the things that must NEVER happen, can't.

## I. Domain-specific correctness [BE]
> Generalize: replace the source project's scheduling examples with YOUR domain's core invariants.
100. [*] **Core-algorithm correctness** — every step/branch/policy of the system's hardest logic (the
     reference-domain candidate for an AI-authored codebase).
101. [*] **Timezone / DST / temporal correctness** — incl. the transition DAY + real-world calendar fixtures.
102. [*] **Core resource invariants** (no double-allocation per resource, capacity) + constraint backstop stress.
103. [*] **Eligibility / window / gate matrices** — the booking/eligibility-gate combinatorial.
104. [*] **Settings conflict-matrix combinatorial coverage.**
105. [*] **Money / currency / minor-unit / rounding / tax** + customer-facing suppression rules.
106. [*] **Notification delivery / templating / dedup / idempotent emission + deep-link/share-link contract.**
107. [*] **Allocation / offer-lifecycle deep correctness** (queues, holds, offers).
108. [*] **Multi-actor variable-duration combinatorial + actor-lifecycle ordering.**
109. **Admin/platform-console + support-session behavioral.**

## J. Client / frontend [CL — DORMANT until web/mobile exists]
110. Component render + hook/state-logic. 111. Consumer-driven contract (client vs API).
112. e2e UI. 113. Visual-regression / snapshot. 114. Accessibility (screen-reader flows).
115. i18n + text-direction (UI render AND data-layer). 116. Responsive / cross-device / cross-browser.
117. Navigation / routing. 118. Form-validation parity (client mirrors backend rules).
119. Permission / role-state UI. 120. Offline / poor-network / resilience.
121. Client performance (bundle / render / runtime). 122. Client AppSec (XSS / DOM / token storage) + CORS/CSRF.
123. Push + deep-link (client side).

## K. Store release & compliance — TESTING facet [ST — DORMANT until a release build]
> The full set (124–149) covers: external-payment / IAP anti-steering conformance (often the top rejection
> risk) · app-review rejection-risk sweep · privacy-label / data-safety accuracy vs actual flows · privacy
> manifest + required-reason APIs · in-app account-deletion compliance · push in a release build · code-signing
> / entitlements / binary provenance · min-OS + device/fragmentation matrix · review-build completeness +
> demo-account liveness · test-track / staged-rollout validation · OTA-update correctness + rollback + policy
> conformance · target-API-level + permissions minimization · pre-launch device-farm crawl + app-signing ·
> deep-link / universal-link verification + scheme-hijack security · content/age rating vs actual behavior ·
> export-compliance declaration · in-app review-prompt policy · listing-as-test (metadata vs binary, forbidden-
> term scan) · regional availability + tax · store-artifact perf budgets · guideline-version-drift re-validation ·
> managed-workflow injection audit (auto-added permissions/SDKs you did not author).
> Register the specific genres when the release build is on the roadmap; until then this category is a single
> dormant placeholder so it is never forgotten.

---

## The ratified completeness BARS (the definition of "done" per domain)
- **Spec-completeness = 100%:** every spec rule / edge / error-code / state-transition (legal + illegal) /
  interaction-edge / setting downstream-effect → a matrix row with a test OR a justified N/A.
- **Mutation score ≥ 90%** (aspire 95%+), **ZERO un-triaged survivors** — each survivor killed or documented
  provably-equivalent. (Mutation, not coverage, is the real "did we test it" bar.)
- **Branch coverage ≥ 95%** floor (line ~100% where practical) — necessary, not sufficient.
- **100%** of error codes have a specific-assert negative test; **100%** of state transitions tested.
- **Zero flaky, zero skips, order-independent** — determinism is a hard gate.
- **Oracle-independence** — expected values derived from the spec independently of the code; audited by a
  different agent than the author (writer ≠ reviewer).
- **Non-functional gates** — perf SLOs (p95/p99) pass/fail; SCA/SAST/secret-scan clean; (client) a11y;
  (store) the store gates green before any submission.
