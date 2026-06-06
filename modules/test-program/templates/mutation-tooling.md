# Mutation Tooling — DB-free + DB-backed, merged

> Mutation testing is the empirical "did we actually test it" bar (genre 82). Coverage says a line RAN;
> mutation says a line is PROTECTED. A mutation tool injects bugs (mutants) into your code and re-runs the
> suite: a mutant the suite KILLS = your tests catch that bug; a SURVIVING mutant = an assertion gap (or a
> documented equivalent). It is the single most direct instrument for "are my tests real?" — it directly
> attacks failure-mode-B. Full-tier (`convention.md` §2.4).
>
> **The bar:** covered-mutation ≥ 90% (aspire 95%+), with **ZERO un-triaged survivors** — every survivor
> either killed or written down as provably-equivalent. Mutation, not coverage, is the real bar.

Illustrative vendor: Stryker (TypeScript). Substitute the equivalent mutation tool for your stack
(PIT for the JVM, mutmut/cosmic-ray for Python, go-mutesting for Go, …). The two-scope pattern below is
vendor-agnostic.

---

## Why two scopes (the crux)

A single mutation run forces a choice: run only the fast DB-free unit suite (misses every mutant that lives
in a DB-interaction line), or boot the full local stack for every mutant (slow, and still attributes a
"survived" to pure-logic mutants the unit suite already kills). Neither alone reaches the bar honestly. So
run TWO scopes and MERGE:

- **DB-free scope** — mutates the PURE logic (the algorithm, the helpers), runs ONLY the no-DB unit suite.
  Fast, no stack needed, kills the bulk of mutants.
- **DB-backed scope** — mutates the QUERY-ARG / DB-interaction line RANGES that only the integration/e2e
  suites can reach, run against the LIVE local stack.
- **Merge rule:** a mutant is killed OVERALL if killed by EITHER config. Report the combined
  covered-mutation score (e.g. `545/576 = 94.6%`). The 31 combined survivors are then each triaged
  (tolerant query projections, message strings, redundant sorts, provably-equivalent transforms) — and
  EVERY survivor is dispositioned. None left un-triaged.

This split is also why the bar is on **covered** mutation: "no-coverage" mutants (integration-only paths
the DB-free scope cannot reach) are excluded from the DB-free score and picked up by the DB-backed scope —
counting them as survivors in the DB-free run would understate the truth.

---

The configs below are shown in the most common mutation-tool config dialect (a JSON object with
`mutate` globs, a `testRunner`, per-test coverage, and a `break` threshold). Every concrete name is a
`<placeholder>` you substitute for your tool's equivalent — the config FILE name, the TEST-CONFIG file
name, the runner-plugin package, the temp-dir key. Only the SHAPE (two scopes, merged; pure-logic vs
DB-line-range mutate sets; report-only `break` until the bar) is mandated. A fully concrete worked example
(Stryker + Vitest) follows the parameterized pair.

### Config A — DB-free (the pure-logic scope)

```jsonc
// <mutation-config-file>          — DB-FREE: mutate the pure logic, run only the no-DB unit suite
{
  "packageManager": "<pnpm|npm|yarn>",
  "plugins": ["<runner-plugin-for-your-test-runner>"],
  "testRunner": "<your-runner>",
  "<runner-config-key>": { "configFile": "<test-config-file-no-db>" }, // include ONLY the no-DB projects
  "mutate": [
    "<path/to/core-logic.ts>"                                          // the pure engine / helpers
  ],
  "coverageAnalysis": "perTest",
  "incremental": true,
  "reporters": ["html", "clear-text", "progress"],
  "thresholds": { "high": 90, "low": 80, "break": null }     // REPORT-ONLY until the domain hits the bar
}
```

`<test-config-file-no-db>` includes only the DB-free projects (unit + contract) so the run needs no stack.

### Config B — DB-backed (the query-arg scope)

```jsonc
// <mutation-integration-config-file>  — DB-BACKED: mutate the DB-call line ranges only,
// run the integration + e2e suites against the LIVE local stack.
{
  "packageManager": "<pnpm|npm|yarn>",
  "plugins": ["<runner-plugin-for-your-test-runner>"],
  "testRunner": "<your-runner>",
  "<runner-config-key>": { "configFile": "<test-config-file-integration>" }, // include integration/e2e projects
  "mutate": [
    "<path/to/core-logic.ts>:41-145",     // ONLY the DB-interaction line ranges
    "<path/to/core-logic.ts>:487-545",    // (the pure engine is already covered by Config A)
    "<path/to/core-logic.ts>:554-606"
  ],
  "coverageAnalysis": "perTest",
  "incremental": false,
  "reporters": ["clear-text", "progress"],
  "thresholds": { "high": 90, "low": 80, "break": null },
  "<temp-dir-key>": "<isolated-temp-dir-for-this-scope>",   // keep the two scopes' temp dirs separate
  "timeoutMS": 60000                       // DB-backed runs are slower per mutant
}
```

The DB-backed scope REQUIRES the local stack up, and it inherits the prod-safety guard (`convention.md`
§1.4): it must refuse to run against anything but the local/test target.

### Concrete worked example (illustrative — Stryker + Vitest; NOT mandated)

Resolving the placeholders for one real stack, so the shape is unambiguous. Substitute freely:

```jsonc
// stryker.conf.json  — DB-free (mutation-config-file = stryker.conf.json)
{
  "packageManager": "pnpm",
  "plugins": ["@stryker-mutator/vitest-runner"],            // runner-plugin-for-your-test-runner
  "testRunner": "vitest",                                   // your-runner
  "vitest": { "configFile": "vitest.mutation.config.ts" },  // runner-config-key = "vitest"
  "mutate": ["apps/api/src/services/slots.service.ts"],
  "coverageAnalysis": "perTest", "incremental": true,
  "reporters": ["html", "clear-text", "progress"],
  "thresholds": { "high": 90, "low": 80, "break": null }
}
// stryker.integration.conf.json adds: "tempDirName": ".stryker-tmp-integration"  (temp-dir-key = "tempDirName")
```

---

## Staged enforcement (do not break a green suite with a bar it does not yet meet)

- **Install + measure (now):** instruments wired, `break = null` (report-only). Record the first baseline
  per domain. CI mutation job is MANUAL (`workflow_dispatch`) + report-only + DB-free — it does NOT block
  push/PR (mutation is slow and not yet at the bar).
- **Enforce per domain (as each reaches the bar):** set that domain's `break = 90`, add a branch-coverage
  gate, and optionally make the CI mutation job blocking for that domain's files. This way the gate turns
  on domain-by-domain as Phase 2 closes each — a green suite is never broken by a gate it has not met.

## Triage discipline (zero un-triaged survivors)

For EVERY surviving mutant: either ADD a test that kills it, or write down WHY it is provably equivalent
(no observable behavior change) — in the findings register, with a tracker id. A bare "it survived, oh
well" is not a disposition. Common legitimate equivalents: log/message string mutations, redundant sort
stabilizers, tolerant response-projection changes the wire format already ignores. The DONE bar is
"covered-mutation ≥ 90% AND zero un-triaged survivors", checked as part of the per-domain completeness
gate (`convention.md` §2.6).

## Install note (corporate TLS-MITM networks)

On a network that does TLS inspection, adding new dev-deps may fail cert verification. The safe fix is to
install via the SYSTEM certificate store (e.g. Node 22+: `NODE_OPTIONS=--use-system-ca`), which KEEPS TLS
verification on and trusts the OS trust store — NEVER disable verification (no `strict-ssl=false`, no
`NODE_TLS_REJECT_UNAUTHORIZED=0`). After installing, commit the lockfile (CI uses a frozen install).
This is the generalized lesson; see the platform-notes stub in your project's CLAUDE.md.
